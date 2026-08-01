---
document_type: Database Schema (DDL)
version: "0.2"
status: Draft
author: "PO / SA / Dev"
created: "2026-07-29"
last_updated: "2026-08-02"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
tech_lead: "Dev"
classification: "Internal"
tags: [database-schema, ddl, postgresql, members, donations, points, privacy]
standard_ref:
  - SWEBOK v4 — Design
parent_project: "Deerngo Bot — VRM"
---

# Database Schema (DDL)

> **Project:** Deerngo Bot — Viewer Relationship Management (VRM)
> **Version:** 0.2 | **Status:** Draft
> **Last Updated:** 2026-08-02
>
> **Scope change:** The subscriber-observation and YouTube OAuth tables are removed from the active Phase 1 MVP. Explicit `members` are the source of membership and points eligibility.

---

## 1. Purpose

This document defines the Phase 1 physical data model: members, donations, point totals, constraints, indexes, and idempotent point application.

## 2. Database Overview

| Field | Detail |
|-------|--------|
| RDBMS | PostgreSQL 18 |
| Database Name | `deerngo` |
| Schema | `public` |
| Naming Convention | `snake_case`, lowercase |
| Required Extensions | `uuid-ossp` only for the MVP schema |
| Connection | Shared homelab PostgreSQL instance |

> `pg_trgm` is no longer required for the Phase 1 member matching path because matching is normalized exact matching.

---

## 3. Extensions

```sql
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
```

---

## 4. DDL Scripts

### 4.1 Members Table

> Stores viewers who explicitly register through streamer.bot. The YouTube display name is intentionally not stored.

```sql
CREATE TABLE members (
    member_id          UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    youtube_user_id     VARCHAR(255) NOT NULL,
    youtube_handle     VARCHAR(100) NOT NULL,
    status              VARCHAR(20) NOT NULL DEFAULT 'active',
    public_visibility   BOOLEAN NOT NULL DEFAULT TRUE,
    registered_at       TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    total_points       DECIMAL(12,2) NOT NULL DEFAULT 0,
    donation_count     INTEGER NOT NULL DEFAULT 0,
    last_donation      TIMESTAMP WITH TIME ZONE,
    created_at         TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    updated_at         TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),

    CONSTRAINT chk_members_status CHECK (status IN ('active', 'inactive')),
    CONSTRAINT chk_members_handle CHECK (LENGTH(youtube_handle) > 0),
    CONSTRAINT chk_members_points CHECK (total_points >= 0),
    CONSTRAINT chk_members_donation_count CHECK (donation_count >= 0)
);

-- A user can have at most one active member record.
CREATE UNIQUE INDEX uk_members_active_user
    ON members(youtube_user_id)
    WHERE status = 'active';

-- A normalized handle can belong to at most one active member.
CREATE UNIQUE INDEX uk_members_active_handle
    ON members(youtube_handle)
    WHERE status = 'active';

CREATE INDEX idx_members_user_id ON members(youtube_user_id);
CREATE INDEX idx_members_handle ON members(youtube_handle);
CREATE INDEX idx_members_status ON members(status);
CREATE INDEX idx_members_public_points
    ON members(total_points DESC)
    WHERE status = 'active' AND public_visibility = TRUE AND total_points > 0;
```

### 4.2 Donations Table

> Stores EasyDonate events privately for reconciliation, exact matching, and idempotency. Raw donor names and messages must never be returned by public APIs.

```sql
CREATE TABLE donations (
    donation_id        UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    reference_no       VARCHAR(100) NOT NULL,
    donor_name         VARCHAR(255) NOT NULL,
    amount_thb         DECIMAL(12,2) NOT NULL,
    currency           VARCHAR(3) NOT NULL DEFAULT 'THB',
    donation_time      TIMESTAMP WITH TIME ZONE NOT NULL,
    message            TEXT,
    source             VARCHAR(20) NOT NULL,
    match_status       VARCHAR(30) NOT NULL DEFAULT 'pending',
    matched_member_id  UUID,
    points_applied_at  TIMESTAMP WITH TIME ZONE,
    created_at         TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    updated_at         TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),

    CONSTRAINT uk_donations_reference UNIQUE (reference_no),
    CONSTRAINT fk_donations_member FOREIGN KEY (matched_member_id)
        REFERENCES members(member_id) ON DELETE SET NULL,
    CONSTRAINT chk_donations_source CHECK (source IN ('webhook', 'api_poll')),
    CONSTRAINT chk_donations_match_status CHECK (match_status IN (
        'pending', 'matched', 'not_eligible', 'unmatched', 'duplicate', 'manual_review'
    )),
    CONSTRAINT chk_donations_amount CHECK (amount_thb > 0),
    CONSTRAINT chk_donations_donor CHECK (LENGTH(TRIM(donor_name)) > 0)
);

CREATE INDEX idx_donations_reference ON donations(reference_no);
CREATE INDEX idx_donations_time ON donations(donation_time DESC);
CREATE INDEX idx_donations_match_status ON donations(match_status);
CREATE INDEX idx_donations_member ON donations(matched_member_id);
CREATE INDEX idx_donations_donor_name ON donations(donor_name);
```

### 4.3 Optional Audit Note Table

> Lightweight manual-correction record. This is not an admin console; it preserves who/why for direct database corrections.

```sql
CREATE TABLE point_adjustment_notes (
    adjustment_id      UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    member_id          UUID NOT NULL REFERENCES members(member_id),
    points_before      DECIMAL(12,2) NOT NULL,
    points_after       DECIMAL(12,2) NOT NULL,
    reason             TEXT NOT NULL,
    changed_by         VARCHAR(255) NOT NULL,
    created_at         TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);
```

The owner/back-office worker may use this table when manually transferring points after a handle change. It does not expose correction details publicly.

---

## 5. Triggers and Point Application

### 5.1 Updated At Trigger

```sql
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_members_updated
    BEFORE UPDATE ON members
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER trg_donations_updated
    BEFORE UPDATE ON donations
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
```

### 5.2 Point Application Rule

Points must be applied by a transaction or equivalent guarded service operation, not by a naive donation-status trigger.

Pseudocode:

```sql
BEGIN;

-- Lock the donation and ensure it has not already applied points.
SELECT *
FROM donations
WHERE donation_id = :donation_id
FOR UPDATE;

-- Find only an active member with exact normalized handle match.
SELECT *
FROM members
WHERE status = 'active'
  AND youtube_handle = :normalized_donor_name
  AND registered_at <= :donation_time
FOR UPDATE;

-- If exactly one member exists and points_applied_at is null:
UPDATE members
SET total_points = total_points + :amount_thb,
    donation_count = donation_count + 1,
    last_donation = GREATEST(COALESCE(last_donation, :donation_time), :donation_time)
WHERE member_id = :member_id;

UPDATE donations
SET matched_member_id = :member_id,
    match_status = 'matched',
    points_applied_at = NOW()
WHERE donation_id = :donation_id
  AND points_applied_at IS NULL;

COMMIT;
```

Rules:

- Pre-registration donations become `not_eligible`.
- Donations matching inactive members become `not_eligible` or `unmatched`.
- No match remains `unmatched` and uncredited.
- A duplicate `reference_no` never creates points twice.
- An active handle conflict is prevented at registration.

---

## 6. Public Scoreboard Query

```sql
SELECT
    ROW_NUMBER() OVER (ORDER BY total_points DESC, youtube_handle ASC) AS rank,
    youtube_handle,
    total_points
FROM members
WHERE status = 'active'
  AND public_visibility = TRUE
  AND total_points > 0
ORDER BY total_points DESC, youtube_handle ASC
LIMIT :limit OFFSET :offset;
```

Never select `youtube_user_id`, `donor_name`, `message`, or inactive/private members for the public response.

---

## 7. Migration Plan

| Version | Description | Notes |
|---------|-------------|-------|
| v0.1 | Original subscriber/donation/viewer_points/oauth schema | Superseded; do not use for new MVP code |
| v0.2 | `members`, revised `donations`, optional `point_adjustment_notes` | New Phase 1 baseline |

Recommended migration approach:

1. Create new tables in an additive migration.
2. Do not migrate old YouTube subscriber rows into members automatically.
3. Do not migrate old points/donation matches into the new member totals automatically.
4. Start all new members at 0 points.
5. Keep the old schema only as a temporary migration artifact, then remove it after Dev verifies no active code depends on it.

---

## 8. Development Seed Data

```sql
INSERT INTO members (
    youtube_user_id, youtube_handle, status, public_visibility, registered_at
) VALUES
    ('UC-test-001', 'testviewer1', 'active', TRUE, NOW() - INTERVAL '2 days'),
    ('UC-test-002', 'privateviewer', 'active', FALSE, NOW() - INTERVAL '1 day'),
    ('UC-test-003', 'inactiveviewer', 'inactive', TRUE, NOW() - INTERVAL '3 days');
```

Use synthetic donor data in tests. Do not use real client donations or personal data in development fixtures.

---

## Related Documents

| Document | Relationship |
|----------|-------------|
| [[022_API_specification]] | API endpoints using this model |
| [[024_ERD]] | Logical data model |
| [[012_user_stories]] | Member and points requirements |
| [[013_acceptance_criteria]] | Data-integrity criteria |
| [[072_MM06_dev-to-po-qa-youtube-subscriber-limit_20260801]] | Approved scope change |

---

> **Template Standard:** Based on SWEBOK v4
> **Usage:** Use migrations to manage this schema. Never modify production tables ad hoc without a backup, transaction, and correction note.
