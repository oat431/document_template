---
document_type: Database Schema (DDL)
version: "0.1"
status: Draft
author: "SA / Designer Persona"
created: "2026-07-29"
last_updated: "2026-07-29"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
tech_lead: "SA / Designer Persona"
classification: "Internal"
tags: [database-schema, ddl, postgresql, swebok, vrm, pg-trgm]
standard_ref:
  - SWEBOK v4 — Design
parent_project: "Deerngo Bot — VRM"
---

# Database Schema (DDL)

> **Project:** Deerngo Bot — Viewer Relationship Management (VRM)
> **Version:** 0.1 | **Status:** Draft
> **Last Updated:** 2026-07-29

---

## Document Control

| Field | Value |
|-------|-------|
| Document Owner | SA / Designer Persona |
| Database | PostgreSQL 18 (existing homelab) |
| Database Name | `deerngo` |

### Revision History

| Version | Date | Author | Change Description |
|---------|------|--------|--------------------|
| 0.1 | 2026-07-29 | SA | Initial schema — subscribers, donations, viewer_points, oauth_tokens |

---

## 1. Purpose

> This document provides the physical database schema for Deerngo Bot — DDL statements, indexes, constraints, triggers, and seed data. It implements the data model defined in [[024_ERD]].

---

## 2. Database Overview

| Field | Detail |
|-------|--------|
| RDBMS | PostgreSQL 18 |
| Database Name | `deerngo` |
| Character Set | UTF-8 |
| Collation | `en_US.UTF-8` |
| Schema | `public` |
| Naming Convention | `snake_case`, lowercase |
| Extensions Required | `pg_trgm` (fuzzy matching), `uuid-ossp` (UUID generation) |
| Connection | Shared homelab PostgreSQL instance |

---

## 3. Extensions

```sql
-- Required extensions
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";    -- UUID generation
CREATE EXTENSION IF NOT EXISTS "pg_trgm";      -- Fuzzy string matching (similarity(), % operator)
```

---

## 4. DDL Scripts

### 4.1 Subscribers Table

> Stores YouTube subscriber data captured via hybrid approach (YouTube API polling + streamer.bot real-time).

```sql
CREATE TABLE subscribers (
    id              UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    youtube_handle  VARCHAR(100) NOT NULL,
    display_name    VARCHAR(255) NOT NULL,
    subscribed_at   TIMESTAMP WITH TIME ZONE NOT NULL,
    source          VARCHAR(20) NOT NULL DEFAULT 'youtube_api',
    created_at      TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at      TIMESTAMP WITH TIME ZONE DEFAULT NOW(),

    -- Upsert key — unique per YouTube handle (normalized: lowercase, no @)
    CONSTRAINT uk_subscribers_handle UNIQUE (youtube_handle),

    -- Source must be one of the two capture methods
    CONSTRAINT chk_subscribers_source CHECK (source IN ('youtube_api', 'streamer_bot')),

    -- Handle must not be empty
    CONSTRAINT chk_subscribers_handle CHECK (LENGTH(youtube_handle) > 0)
);

-- Indexes
CREATE INDEX idx_subscribers_handle ON subscribers(youtube_handle);
CREATE INDEX idx_subscribers_subscribed ON subscribers(subscribed_at DESC);
CREATE INDEX idx_subscribers_source ON subscribers(source);

-- Trigram index for fuzzy matching (used by name matching engine)
CREATE INDEX idx_subscribers_handle_trgm ON subscribers USING gin (youtube_handle gin_trgm_ops);
```

### 4.2 Donations Table

> Stores donation records from EasyDonate (via webhook or REST API polling).

```sql
CREATE TABLE donations (
    id              UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    easydonate_id   VARCHAR(100) NOT NULL,
    donor_name      VARCHAR(255) NOT NULL,
    amount_thb      DECIMAL(12,2) NOT NULL,
    currency        VARCHAR(3) NOT NULL DEFAULT 'THB',
    donation_time   TIMESTAMP WITH TIME ZONE NOT NULL,
    message         TEXT,
    matched_handle  VARCHAR(100),          -- YouTube handle matched (nullable)
    match_status    VARCHAR(20) NOT NULL DEFAULT 'pending',
    match_score     DECIMAL(5,4),          -- similarity score (0.0000 → 1.0000)
    source          VARCHAR(20) NOT NULL DEFAULT 'webhook',
    created_at      TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at      TIMESTAMP WITH TIME ZONE DEFAULT NOW(),

    -- Idempotency key — prevent duplicate donations
    CONSTRAINT uk_donations_easydonate UNIQUE (easydonate_id),

    -- Foreign key to subscribers (nullable — unmatched donations have no subscriber)
    CONSTRAINT fk_donations_subscriber FOREIGN KEY (matched_handle)
        REFERENCES subscribers(youtube_handle) ON DELETE SET NULL,

    -- Match status must be one of the defined states
    CONSTRAINT chk_donations_match_status CHECK (match_status IN (
        'pending',       -- not yet processed by matching engine
        'matched',       -- successfully matched to a subscriber
        'unmatched',     -- no match found (anonymous, no close match)
        'manual_review'  -- multiple possible matches, needs human review
    )),

    -- Source must be webhook or api_poll
    CONSTRAINT chk_donations_source CHECK (source IN ('webhook', 'api_poll')),

    -- Amount must be positive
    CONSTRAINT chk_donations_amount CHECK (amount_thb > 0),

    -- Donor name must not be empty
    CONSTRAINT chk_donations_donor CHECK (LENGTH(donor_name) > 0)
);

-- Indexes
CREATE INDEX idx_donations_easydonate ON donations(easydonate_id);
CREATE INDEX idx_donations_donor ON donations(donor_name);
CREATE INDEX idx_donations_time ON donations(donation_time DESC);
CREATE INDEX idx_donations_match_status ON donations(match_status);
CREATE INDEX idx_donations_matched_handle ON donations(matched_handle);
CREATE INDEX idx_donations_source ON donations(source);

-- Trigram index on donor_name for fuzzy matching against subscribers
CREATE INDEX idx_donations_donor_trgm ON donations USING gin (donor_name gin_trgm_ops);
```

### 4.3 Viewer Points Table

> Materialized summary of points per viewer. Updated when donations are matched. Denormalized for fast query by the `:deer: point` command.

```sql
CREATE TABLE viewer_points (
    id              UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    youtube_handle  VARCHAR(100) NOT NULL,
    display_name    VARCHAR(255) NOT NULL,
    total_points    DECIMAL(12,2) NOT NULL DEFAULT 0,
    donation_count  INTEGER NOT NULL DEFAULT 0,
    last_donation   TIMESTAMP WITH TIME ZONE,
    created_at      TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at      TIMESTAMP WITH TIME ZONE DEFAULT NOW(),

    -- One record per YouTube handle
    CONSTRAINT uk_viewer_points_handle UNIQUE (youtube_handle),

    -- Foreign key to subscribers
    CONSTRAINT fk_viewer_points_subscriber FOREIGN KEY (youtube_handle)
        REFERENCES subscribers(youtube_handle) ON DELETE CASCADE,

    -- Points cannot be negative
    CONSTRAINT chk_viewer_points_total CHECK (total_points >= 0),

    -- Donation count cannot be negative
    CONSTRAINT chk_viewer_points_count CHECK (donation_count >= 0)
);

-- Indexes
CREATE INDEX idx_viewer_points_handle ON viewer_points(youtube_handle);
CREATE INDEX idx_viewer_points_total ON viewer_points(total_points DESC);
CREATE INDEX idx_viewer_points_name ON viewer_points(display_name);
```

### 4.4 OAuth Tokens Table

> Stores OAuth 2.0 refresh tokens for YouTube Data API access. Only the refresh token is persisted — access tokens are short-lived and regenerated at runtime. Designed to support multiple YouTube channels in the future.

```sql
CREATE TABLE oauth_tokens (
    id                  UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    provider            VARCHAR(50) NOT NULL,
    youtube_channel_id  VARCHAR(100) NOT NULL,
    refresh_token       TEXT NOT NULL,
    created_at          TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at          TIMESTAMP WITH TIME ZONE DEFAULT NOW(),

    -- One token per provider + channel combination
    CONSTRAINT uk_oauth_provider_channel UNIQUE (provider, youtube_channel_id),

    -- Provider must be defined
    CONSTRAINT chk_oauth_provider CHECK (provider IN ('youtube'))
);

-- Index
CREATE INDEX idx_oauth_provider ON oauth_tokens(provider);
CREATE INDEX idx_oauth_channel ON oauth_tokens(youtube_channel_id);
```

---

## 5. Triggers

### 5.1 Updated At Trigger

```sql
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ language 'plpgsql';

CREATE TRIGGER trg_subscribers_updated
    BEFORE UPDATE ON subscribers
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER trg_donations_updated
    BEFORE UPDATE ON donations
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER trg_viewer_points_updated
    BEFORE UPDATE ON viewer_points
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER trg_oauth_tokens_updated
    BEFORE UPDATE ON oauth_tokens
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
```

### 5.2 Sync Viewer Points Trigger

> Automatically updates `viewer_points` when a donation's `match_status` changes to `matched`. Keeps the materialized summary in sync with the donations table.

```sql
CREATE OR REPLACE FUNCTION sync_viewer_points()
RETURNS TRIGGER AS $$
BEGIN
    -- Only process when match_status changes to 'matched'
    IF NEW.match_status = 'matched' AND NEW.matched_handle IS NOT NULL THEN
        INSERT INTO viewer_points (youtube_handle, display_name, total_points, donation_count, last_donation)
        VALUES (
            NEW.matched_handle,
            COALESCE(
                (SELECT display_name FROM subscribers WHERE youtube_handle = NEW.matched_handle),
                NEW.matched_handle
            ),
            NEW.amount_thb,
            1,
            NEW.donation_time
        )
        ON CONFLICT (youtube_handle) DO UPDATE SET
            total_points = viewer_points.total_points + NEW.amount_thb,
            donation_count = viewer_points.donation_count + 1,
            last_donation = GREATEST(viewer_points.last_donation, NEW.donation_time),
            display_name = COALESCE(
                (SELECT display_name FROM subscribers WHERE youtube_handle = NEW.matched_handle),
                viewer_points.display_name
            );
    END IF;
    RETURN NEW;
END;
$$ language 'plpgsql';

CREATE TRIGGER trg_donations_sync_points
    AFTER UPDATE ON donations
    FOR EACH ROW
    WHEN (OLD.match_status IS DISTINCT FROM NEW.match_status)
    EXECUTE FUNCTION sync_viewer_points();
```

---

## 6. Database Migrations

| Version | Date | Description | Script |
|---------|------|-------------|--------|
| v0.1 | 2026-07-29 | Initial schema — extensions, tables, indexes, triggers | `001_initial_schema.sql` |

> **Migration Tool:** `golang-migrate/migrate` — file-based migrations, compatible with sqlx workflow.

---

## 7. Seed Data (Development)

```sql
-- Development-only: insert test subscribers
INSERT INTO subscribers (youtube_handle, display_name, subscribed_at, source) VALUES
    ('@testviewer1', 'TestViewer1', NOW() - INTERVAL '7 days', 'youtube_api'),
    ('@testviewer2', 'TestViewer2', NOW() - INTERVAL '3 days', 'streamer_bot'),
    ('@deer_fan', 'Deer Fan', NOW() - INTERVAL '1 day', 'youtube_api');

-- Development-only: insert test donations
INSERT INTO donations (easydonate_id, donor_name, amount_thb, donation_time, match_status, matched_handle, source) VALUES
    ('ed-test-001', 'testviewer1', 100.00, NOW() - INTERVAL '5 days', 'matched', '@testviewer1', 'webhook'),
    ('ed-test-002', 'testviewer2', 250.50, NOW() - INTERVAL '2 days', 'matched', '@testviewer2', 'webhook'),
    ('ed-test-003', 'anonymous', 50.00, NOW() - INTERVAL '1 day', 'unmatched', NULL, 'webhook'),
    ('ed-test-004', 'deer_fan', 500.00, NOW() - INTERVAL '6 hours', 'matched', '@deer_fan', 'api_poll');
```

---

## 8. Backup & Recovery

| Strategy | Frequency | Retention | Method |
|----------|-----------|-----------|--------|
| Full backup | Daily (homelab schedule) | 30 days | `pg_dump` (shared with other databases) |
| Point-in-time | On demand | 7 days | WAL archiving (if configured on homelab) |

> **Note:** Backup strategy follows the homelab's existing PostgreSQL backup schedule. No separate backup configuration needed for Phase 1.

---

## Related Documents

| Document | Relationship |
|----------|-------------|
| [[024_ERD]] | Logical model this schema implements |
| [[021_architecture_decision_records]] | ADR-002 (PostgreSQL), ADR-006 (pg_trgm), ADR-008 (sqlx) |
| [[022_API_specification]] | API endpoints that query these tables |
| [[013_acceptance_criteria]] | ACs that verify data integrity |

---

> **Template Standard:** Based on SWEBOK v4
> **Usage:** This is the *physical* database schema. Use `golang-migrate/migrate` to manage schema changes. Never modify production schema manually.
