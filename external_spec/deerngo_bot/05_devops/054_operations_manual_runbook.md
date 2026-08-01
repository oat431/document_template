---
document_type: Operations Manual / Runbook
version: "0.2"
status: Draft
author: "DevOps / PO"
created: "2026-07-30"
last_updated: "2026-08-02"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
classification: "Internal"
tags: [runbook, operations, docker, homelab, troubleshooting, members, points, easydonate, privacy]
standard_ref:
  - SWEBOK v4 — Operations
  - SEBoK v2 — Operations
parent_project: "Deerngo Bot — VRM"
---

# Operations Manual / Runbook

> **Project:** Deerngo Bot — Viewer Relationship Management (VRM)
> **Version:** 0.2 | **Status:** Draft
> **Last Updated:** 2026-08-02
>
> **Scope change:** YouTube subscriber polling/OAuth is removed from the active MVP. This runbook covers explicit members, EasyDonate, points, visibility, and the public scoreboard.

---

## 1. Purpose

Step-by-step operational procedures for the homelab deployment, health checks, member/points data safety, EasyDonate reconciliation, and incident response.

## 2. System Overview

| Component | Technology | Container | Port | Location |
|-----------|-----------|-----------|------|----------|
| Go Backend | Go / Fiber / sqlx | `deerngo-bot` | :8008 | Homelab Docker (`db-network`) |
| Next.js Frontend | Next.js | `deerngo-web` | :3008 | Homelab Docker (`db-network`) |
| PostgreSQL 18 | PostgreSQL | `local-postgres` | :5432 | Homelab Docker |
| Cloudflare Tunnel | cloudflared | systemd | — | Homelab |
| streamer.bot | Windows app | — | :7474, :8681 | Streamer's Windows PC |
| EasyDonate | External webhook/API | — | HTTPS | Provider |

## 3. Access and Credentials

| System | Access | Location |
|--------|--------|----------|
| Homelab | SSH over Tailscale | Server owner-managed |
| Docker | SSH server CLI | Homelab |
| PostgreSQL | Restricted `psql`/container access | Homelab |
| GitHub/GHCR | Owner-managed account/token | GitHub |
| EasyDonate | Owner account; backend API key in deployment secret | Never in docs/logs |

Never store passwords, API keys, webhook path tokens, provider payloads, or database credentials in this document or Git.

---

## 4. Routine Operations

### 4.1 Daily Checks

| # | Check | Command | Expected | Action if Failed |
|---|-------|---------|---------|-----------------|
| 1 | Backend health | `curl -sf http://localhost:8008/healthz` | 200/healthy | Restart/check logs |
| 2 | Frontend loads | `curl -sf http://localhost:3008` | HTML 200 | Restart/check logs |
| 3 | Containers | `docker ps --filter name=deerngo` | Both Up/healthy | Inspect logs |
| 4 | Public scoreboard | `curl -I https://deerngo-viewer-score.panomete.com` | 200 | Check tunnel/frontend |
| 5 | Disk | `df -h /` | <80% | Cleanup/escalate |
| 6 | Donation backlog | Query pending/unmatched counts | Within expected range | Reconcile provider/matching |

### 4.2 Weekly Checks

- Review backend errors with secrets/raw donor data redaction.
- Review `donations` statuses: pending, unmatched, not_eligible, matched.
- Verify EasyDonate fallback sync last-run time and 429 behavior.
- Verify public scoreboard excludes private/inactive/zero-point rows.
- Check Docker image/dependency updates.

### 4.3 Monthly Checks

- Restore a database backup into a test database.
- Run Go/Node dependency security audits.
- Review raw donation-data retention against the owner-approved policy.
- Review webhook path token exposure/rotation need.
- Review manual point-adjustment notes.

---

## 5. Container Procedures

### 5.1 Restart

```bash
ssh <operator>@<homelab-host>
cd ~/platform
docker compose -f docker-compose.deerngo.yml restart deerngo-bot
curl -sf http://localhost:8008/healthz

docker compose -f docker-compose.deerngo.yml restart deerngo-web
curl -sf http://localhost:3008
```

### 5.2 Logs

```bash
docker logs --tail 100 deerngo-bot
docker logs --since 1h deerngo-bot
docker logs --tail 100 deerngo-web
```

Do not use commands that dump environment variables or secrets into shared output. Search for safe error classes, not raw payloads.

### 5.3 Environment Changes

1. Update deployment secret store or restricted `.env` on the homelab.
2. Do not paste values into chat/GitHub.
3. Restart only the affected service.
4. Verify health and a safe synthetic request.
5. If changing webhook path token, update EasyDonate dashboard URL and test a safe event.

---

## 6. Database Procedures

### 6.1 Connect

```bash
docker exec -it local-postgres psql -U postgres -d deerngo
```

### 6.2 Safe Read Checks

```sql
-- Table inventory
SELECT tablename FROM pg_tables WHERE schemaname = 'public' ORDER BY tablename;

-- Member status counts
SELECT status, public_visibility, COUNT(*)
FROM members
GROUP BY status, public_visibility
ORDER BY status, public_visibility;

-- Donation status totals (do not print donor_name/message in routine output)
SELECT match_status, source, COUNT(*), COALESCE(SUM(amount_thb),0)
FROM donations
GROUP BY match_status, source
ORDER BY match_status, source;

-- Public scoreboard projection
SELECT youtube_handle, total_points
FROM members
WHERE status='active' AND public_visibility=TRUE AND total_points > 0
ORDER BY total_points DESC, youtube_handle ASC
LIMIT 20;
```

### 6.3 Apply Migrations

```bash
# Back up first
./scripts/backup-deerngo.sh

# Apply versioned migration via the project migration command
# (Use the actual command provided by the built backend.)
docker exec deerngo-bot ./deerngo-bot migrate up

# Verify table/index state and health
docker exec deerngo-bot ./deerngo-bot migrate status
curl -sf http://localhost:8008/healthz
```

Never manually drop/alter production tables to “make the feature work.”

### 6.4 Manual Point Correction — MVP Procedure

Used only by the owner/back-office worker when a viewer re-registers with a new handle and points must be manually transferred.

1. Identify old inactive `member_id` and new active `member_id`.
2. Take/verify a current backup.
3. Record both totals before changing anything.
4. Start a transaction.
5. Update only the approved member total.
6. Insert a `point_adjustment_notes` row with before/after totals, reason, operator, and time.
7. Commit.
8. Re-query the new member and check `:deer: point`/scoreboard behavior.
9. If anything is wrong, rollback before commit or restore using the approved recovery procedure.

Example shape — replace placeholders only in a controlled operator session:

```sql
BEGIN;

-- Inspect first; do not paste raw donor data into logs.
SELECT member_id, youtube_handle, status, total_points
FROM members
WHERE member_id IN (:old_member_id, :new_member_id)
FOR UPDATE;

-- After independent verification, record the approved correction.
INSERT INTO point_adjustment_notes
    (member_id, points_before, points_after, reason, changed_by)
VALUES
    (:new_member_id, :points_before, :points_after,
     'Manual transfer after viewer handle re-registration', :operator);

UPDATE members
SET total_points = :points_after
WHERE member_id = :new_member_id
  AND status = 'active';

COMMIT;
```

The actual operator must use parameterized SQL or a reviewed script. Do not edit the old inactive record to make it active, and do not fabricate donation matches.

---

## 7. EasyDonate Operations

### 7.1 Webhook Verification

- Confirm the URL configured in EasyDonate matches the current deployment path token.
- Confirm a safe provider test event reaches the backend.
- Verify `referenceNo` is recorded once.
- Verify invalid path/payload is rejected.
- Do not assume HMAC/signature support until dashboard/provider contract confirms it.

### 7.2 Fallback Reconciliation

- API polling runs every configured interval (default 5 minutes).
- Use backend-only API key with documented donation-read scope.
- Respect `429` and `Retry-After`.
- Repeated provider events must be skipped by `referenceNo`.
- Confirm webhook/API sources converge without double-counting.

### 7.3 Donation Debugging

| Symptom | Investigation | Safe Resolution |
|---------|---------------|-----------------|
| Donation not stored | Check provider delivery status and webhook route logs | Verify path/config/provider contract; wait for API fallback |
| Donation stored but unmatched | Query `match_status` and normalized handle privately | Confirm donor naming rule; no fuzzy auto-credit |
| Donation before registration | Compare `donation_time` and `registered_at` | Keep `not_eligible`; no automatic retro-credit |
| Points applied twice | Compare `reference_no`, `points_applied_at`, member total | Stop matcher, backup, investigate transaction guard |
| API 429 | Inspect safe provider error class | Back off and follow provider limit |

Do not print raw donor name/message in a public or shared log.

---

## 8. streamer.bot Operations

### 8.1 Commands

| Command | Backend action |
|---------|----------------|
| `:deer: register` | POST actual `youtube_user_id` + current handle |
| `:deer: public` | PUT visibility true |
| `:deer: private` | PUT visibility false |
| `:deer: point` | GET identity-based visibility-aware points |
| `:deer: donate` | Send configured EasyDonate URL |

### 8.2 Offline Behavior

- streamer.bot offline: no chat command can be processed.
- Backend offline: streamer.bot sends the configured friendly temporary-unavailable message.
- No partial registration is assumed; viewer retries after recovery.

### 8.3 Safe Action Logs

Log trigger/time/result class only. Never log API keys, path tokens, raw provider payloads, display names, or database connection strings.

---

## 9. Troubleshooting

### Backend Not Responding

```bash
docker ps --filter name=deerngo-bot
docker logs --tail 100 deerngo-bot
curl -v http://localhost:8008/healthz
docker network inspect db-network
```

### Public Scoreboard Down

```bash
systemctl status cloudflared
curl -I https://deerngo-viewer-score.panomete.com
curl -sf http://localhost:3008
curl -sf http://localhost:8008/api/v1/scoreboard
```

### Member Registration Fails

- Verify streamer.bot sends actual `userId` and handle fields.
- Verify LAN reachability to backend.
- Check 400/409 response code without exposing internal details.
- For handle conflict, inspect active-handle uniqueness and owner-resolution process.
- For changed handle, confirm old inactive/new active transaction.

### Points Not Updating

- Check donation ingestion source and `reference_no`.
- Check `match_status` and cutoff timestamps privately.
- Check normalized donor name vs active handle.
- Check exactly-once guard/transaction logs.
- Do not manually credit by editing donation history.

---

## 10. Backup & Recovery

### Backup Before Migration or Point Correction

```bash
docker exec local-postgres pg_dump -U postgres -d deerngo > ~/backups/deerngo_$(date +%Y%m%d_%H%M%S).sql
```

Keep backup paths restricted. Do not upload database dumps to GitHub or chat.

### Recovery

Use the homelab's approved restore procedure. Validate restoration in a test database first where possible. After restore:

1. Apply/verify migration state.
2. Check member status/points counts.
3. Reconcile donation references.
4. Run API and scoreboard smoke tests.
5. Document incident and update this runbook if needed.

## 11. Incident Response

```text
1. DETECT — health check, operator/viewer report, provider notification
2. ASSESS — containers, logs, DB integrity, public route
3. MITIGATE — stop duplicate worker/webhook if points integrity is at risk; restart/rollback app
4. PRESERVE — backup DB and redact evidence
5. RESOLVE — fix provider/config/code root cause
6. VERIFY — run active regression and a safe smoke event
7. REVIEW — update risk register, issue, and runbook
```

## Related Documents

| Document | Relationship |
|----------|-------------|
| [[052_deployment_plan]] | Deployment and rollback |
| [[051_CICD_pipeline_configuration]] | Pipeline configuration |
| [[022_API_specification]] | Current API contract |
| [[023_database_schema_DDL]] | Current DB model |
| [[061_security_test_report]] | Security controls |
| [[071_risk_register]] | Runtime risks |
| `https://github.com/oat431/deerngo-bot/issues/18` | EasyDonate contract gate |
| `https://github.com/oat431/deerngo-bot/issues/19` | Manual correction procedure |

---

> **Template Standard:** Based on SWEBOK v4 and SEBoK v2
> **Usage:** Operational source of truth for the revised member-based MVP. Update after every incident or provider-contract change.
---
