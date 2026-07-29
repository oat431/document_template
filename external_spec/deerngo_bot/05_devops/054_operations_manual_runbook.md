---
document_type: Operations Manual / Runbook
version: "0.1"
status: Draft
author: "DevOps"
created: "2026-07-30"
last_updated: "2026-07-30"
project_name: "Deerngo Bot"
project_id: "DERNBOT-001"
classification: "Internal"
tags: [runbook, operations, docker, homelab, troubleshooting, swebok, sebok]
standard_ref:
  - SWEBOK v4 — Operations
  - SEBoK v2 — Operations
parent_project: "Deerngo Bot — VRM"
---

# Operations Manual / Runbook

> **Project:** Deerngo Bot — Viewer Relationship Management (VRM)
> **Version:** 0.1 | **Status:** Draft
> **Last Updated:** 2026-07-30

---

## Document Control

| Field | Value |
|-------|-------|
| Document Owner | DevOps |
| Server | Homelab (`remote.panomete.com`) |
| SSH Access | `flowero@remote.panomete.com` (Tailscale + key auth) |

### Revision History

| Version | Date | Author | Change Description |
|---------|------|--------|--------------------|
| 0.1 | 2026-07-30 | DevOps | Initial runbook — container ops, troubleshooting, routine checks |

---

## 1. Purpose

> Step-by-step operational procedures for managing Deerngo Bot in production — daily checks, restart procedures, troubleshooting, and incident response. This is the operations bible for the homelab deployment.

---

## 2. System Overview

| Component | Technology | Container | Port | Location |
|-----------|-----------|-----------|------|----------|
| Go Backend | Go 1.24+ / Fiber v3 | `deerngo-bot` | :8008 | Homelab (Docker, `db-network`) |
| Next.js Frontend | Next.js 15+ | `deerngo-web` | :3008 | Homelab (Docker, `db-network`) |
| PostgreSQL 18 | PostgreSQL | `local-postgres` | :5432 | Homelab (Docker, `db-network`) |
| Valkey 9 | Valkey | `local-valkey` | :6379 | Homelab (Docker, `db-network`) |
| Cloudflare Tunnel | cloudflared | systemd | — | Homelab |
| streamer.bot | Windows app | — | :7474, :8681 | Local Windows PC |

---

## 3. Access & Credentials

| System | Access Method | Location |
|--------|-------------|----------|
| Homelab Server | SSH (`flowero@remote.panomete.com`) | Tailscale network |
| Docker | `docker` CLI on homelab | SSH into server |
| PostgreSQL | `docker exec -it local-postgres psql -U postgres -d deerngo` | Via container |
| GHCR Images | `ghcr.io/deerngo/deerngo-bot`, `ghcr.io/deerngo/deerngo-web` | GitHub Packages |
| GitHub Actions | GitHub repo → Actions tab | Web UI |

> **Security:** Passwords, API keys, and SSH keys are managed by the server owner. Never stored in docs or Git.

---

## 4. Routine Operations

### 4.1 Daily Checks

| # | Check | Command | Expected | Action if Failed |
|---|-------|---------|---------|-----------------|
| 1 | Backend health | `curl -sf http://localhost:8008/api/v1/health` | `{"status":"ok"}` | Restart container (§5.1) |
| 2 | Frontend loads | `curl -sf http://localhost:3008` | HTML (200) | Restart container (§5.1) |
| 3 | Containers running | `docker ps --filter name=deerngo` | Both `Up` | Check logs (§6.1) |
| 4 | Public URL | `curl -I https://deerngo-viewer-score.panomete.com` | `200 OK` | Check Nginx + Tunnel (§6.3) |
| 5 | Disk usage | `df -h /` | < 80% | Cleanup Docker (§5.4) |

### 4.2 Weekly Maintenance

| # | Task | Command | Duration |
|---|------|---------|---------|
| 1 | Check Docker image updates | `docker images --filter dangling=true` | 2 min |
| 2 | Review backend logs for errors | `docker logs --since 7d deerngo-bot 2>&1 \| grep -i error` | 5 min |
| 3 | Review frontend logs | `docker logs --since 7d deerngo-web 2>&1 \| grep -i error` | 5 min |
| 4 | Check database size | `docker exec local-postgres psql -U postgres -d deerngo -c "SELECT pg_size_pretty(pg_database_size('deerngo'));"` | 1 min |
| 5 | Clean unused Docker resources | `docker system prune -f` | 2 min |

### 4.3 Monthly Maintenance

| # | Task | Command | Duration |
|---|------|---------|---------|
| 1 | Dependency audit (Go) | `cd deerngo-bot && govulncheck ./...` | 10 min |
| 2 | Dependency audit (Node) | `cd deerngo-web && npm audit` | 5 min |
| 3 | Database backup verification | Restore backup to test DB, verify data | 30 min |
| 4 | YouTube API quota check | Check Google Cloud Console → API quota | 5 min |
| 5 | EasyDonate API status | Manual check of API availability | 5 min |

---

## 5. Operational Procedures

### 5.1 Restart Containers

```bash
# SSH to homelab
ssh flowero@remote.panomete.com

# Restart backend only
cd ~/platform
docker compose -f docker-compose.deerngo.yml restart deerngo-bot

# Restart frontend only
docker compose -f docker-compose.deerngo.yml restart deerngo-web

# Restart both
docker compose -f docker-compose.deerngo.yml restart

# Verify
docker ps --filter name=deerngo
curl http://localhost:8008/api/v1/health
curl http://localhost:3008
```

### 5.2 View Logs

```bash
# Backend logs (last 100 lines)
docker logs --tail 100 deerngo-bot

# Frontend logs (last 100 lines)
docker logs --tail 100 deerngo-web

# Follow live logs
docker logs -f deerngo-bot
docker logs -f deerngo-web

# Logs from last hour
docker logs --since 1h deerngo-bot

# Search for errors
docker logs --since 24h deerngo-bot 2>&1 | grep -i "error\|fatal\|panic"
```

### 5.3 Update Environment Variables

```bash
# 1. SSH to homelab
ssh flowero@remote.panomete.com

# 2. Edit .env file
nano ~/platform/.env

# 3. Restart containers to pick up new values
cd ~/platform
docker compose -f docker-compose.deerngo.yml up -d

# 4. Verify
docker exec deerngo-bot env | grep DATABASE_URL
```

### 5.4 Docker Cleanup

```bash
# Remove stopped containers, unused networks, dangling images
docker system prune -f

# Remove unused images (keeps running ones)
docker image prune -a -f

# Check disk usage
docker system df
```

### 5.5 Database Operations

```bash
# Connect to PostgreSQL
docker exec -it local-postgres psql -U postgres -d deerngo

# Common queries
# Check table sizes
SELECT schemaname, tablename, pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename))
FROM pg_tables WHERE schemaname = 'public' ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;

# Check subscriber count
SELECT count(*) FROM subscribers;

# Check donation stats
SELECT match_status, count(*), sum(amount_thb) FROM donations GROUP BY match_status;

# Check top viewers
SELECT * FROM viewer_points ORDER BY total_points DESC LIMIT 10;

# Run migration
docker exec deerngo-bot ./deerngo-bot migrate up

# Rollback migration
docker exec deerngo-bot ./deerngo-bot migrate down
```

### 5.6 View Running Schedulers

The Go backend runs 3 background schedulers. To verify they're active:

```bash
# Check logs for scheduler activity
docker logs --since 1h deerngo-bot 2>&1 | grep -i "scheduler\|poll\|sync\|match"

# Expected log entries (every few minutes):
# "YouTube API poll completed — X new subscribers"
# "EasyDonate sync completed — X new donations"
# "Name matching completed — X matched, X unmatched"
```

---

## 6. Troubleshooting Guide

### 6.1 Container Won't Start

| Symptom | Possible Cause | Investigation | Resolution |
|---------|---------------|-------------|-----------|
| `deerngo-bot` keeps restarting | Crash on startup | `docker logs deerngo-bot` | Check config, DATABASE_URL, fix code |
| `deerngo-web` keeps restarting | Build error | `docker logs deerngo-web` | Check NEXT_PUBLIC_API_URL |
| Port conflict | Port already in use | `ss -tlnp \| grep 8008` | Stop conflicting process or change port |
| Image pull fails | GHCR auth / network | `docker pull ghcr.io/deerngo/deerngo-bot:latest` | Check network, GHCR access |

### 6.2 API Not Responding

| Symptom | Possible Cause | Investigation | Resolution |
|---------|---------------|-------------|-----------|
| `curl /health` returns nothing | Container down | `docker ps \| grep deerngo-bot` | Restart container (§5.1) |
| 500 errors in API | Application error | `docker logs --tail 50 deerngo-bot` | Check error message, fix code |
| Slow responses (> 2s) | Database slow | Check PostgreSQL: `docker exec local-postgres psql -U postgres -d deerngo -c "SELECT * FROM pg_stat_activity;"` | Optimize queries, check indexes |
| Connection refused | Network issue | `docker network inspect db-network` | Ensure containers on `db-network` |

### 6.3 Public Scoreboard Down

| Symptom | Possible Cause | Investigation | Resolution |
|---------|---------------|-------------|-----------|
| `deerngo-viewer-score.panomete.com` not loading | Cloudflare Tunnel down | `systemctl status cloudflared` | Restart tunnel: `sudo systemctl restart cloudflared` |
| Page loads but no data | Backend API down | `curl http://localhost:8008/api/v1/scoreboard` | Restart backend (§5.1) |
| 502 Bad Gateway | Nginx misconfigured | `nginx -t && systemctl status nginx` | Fix Nginx config, reload |
| Page loads slowly | Frontend slow | Check `deerngo-web` logs | Check Next.js build, restart container |

### 6.4 Donation Points Not Updating

| Symptom | Possible Cause | Investigation | Resolution |
|---------|---------------|-------------|-----------|
| Viewer says "I donated but no points" | Name not matched | `SELECT * FROM donations WHERE match_status = 'pending'` | Check matching engine logs |
| Points wrong amount | Calculation error | `SELECT * FROM donations WHERE matched_handle = '@viewer'` | Verify donation amounts |
| Donation not captured | Webhook failed | `docker logs deerngo-bot 2>&1 \| grep webhook` | Check webhook secret, EasyDonate config |
| EasyDonate sync not running | Scheduler crashed | `docker logs --since 30m deerngo-bot 2>&1 \| grep -i easydonate` | Restart backend |
| YouTube subs not captured | API quota exhausted | `docker logs deerngo-bot 2>&1 \| grep -i "403\|quota"` | Wait for quota reset (midnight PT) |

### 6.5 Name Matching Issues

| Symptom | Possible Cause | Investigation | Resolution |
|---------|---------------|-------------|-----------|
| False matches | Threshold too low | `SELECT * FROM donations WHERE match_status = 'matched' AND match_score < 0.8` | Tune threshold in code (currently 0.7) |
| No matches for valid name | Threshold too high / name format | Check `donor_name` vs `youtube_handle` format | Adjust matching rules |
| `manual_review` backlog | Multiple similar names | `SELECT * FROM donations WHERE match_status = 'manual_review'` | Manual review and update |

---

## 7. Incident Response

### 7.1 Severity Levels

| Severity | Definition | Response Time | Example |
|----------|-----------|-------------|---------|
| 🔴 **Critical** | System completely down, data loss | Immediate | All containers crashed, DB corruption |
| 🟡 **High** | Major feature broken | < 1 hour | Bot commands not working, scoreboard down |
| 🟢 **Medium** | Degraded performance | < 4 hours | Slow responses, intermittent errors |
| ⚪ **Low** | Minor issue, workaround exists | Next session | Cosmetic issues, log noise |

### 7.2 Incident Response Steps

```
1. DETECT — How did we find out?
   - Daily health check failed
   - Viewer reported issue in chat
   - Error alert from logs

2. ASSESS — How bad is it?
   - Check all containers: docker ps
   - Check logs: docker logs --tail 100 deerngo-bot
   - Check public URL: curl -I https://deerngo-viewer-score.panomete.com

3. MITIGATE — Stop the bleeding
   - Restart containers (§5.1)
   - If code issue: rollback to previous image (see 052_deployment_plan §7)
   - If DB issue: stop backend, assess data integrity

4. RESOLVE — Fix the root cause
   - Identify root cause from logs
   - Fix code / config
   - Test locally
   - Deploy fix via CI/CD

5. REVIEW — Prevent recurrence
   - Document what happened
   - Update this runbook if new scenario
   - Add monitoring if gap identified
```

---

## 8. Backup & Recovery

### 8.1 Database Backup

```bash
# Manual backup
docker exec local-postgres pg_dump -U postgres -d deerngo > ~/backups/deerngo_$(date +%Y%m%d_%H%M%S).sql

# With compression
docker exec local-postgres pg_dump -U postgres -d deerngo | gzip > ~/backups/deerngo_$(date +%Y%m%d_%H%M%S).sql.gz

# List backups
ls -la ~/backups/deerngo_*.sql*
```

### 8.2 Database Restore

```bash
# Restore from backup
cat ~/backups/deerngo_20260730_120000.sql | docker exec -i local-postgres psql -U postgres -d deerngo

# Restore from compressed backup
gunzip -c ~/backups/deerngo_20260730_120000.sql.gz | docker exec -i local-postgres psql -U postgres -d deerngo
```

### 8.3 Backup Schedule (Recommended)

| Backup | Frequency | Retention | Method |
|--------|-----------|-----------|--------|
| Full DB dump | Daily | 7 days | Cron job on homelab |
| Before migration | Per migration | Until verified | Manual `pg_dump` |
| Before deploy | Per deploy | 3 days | Manual or CI step |

---

## 9. Emergency Contacts

| Role | Who | When |
|------|-----|------|
| Server Owner / DevOps | Deer_NGO | All issues — primary contact |
| Backup Contact | — | If server owner unavailable |

> **Note:** Single-developer project. All operational responsibility falls on the server owner / developer. No on-call rotation needed.

---

## 10. Quick Reference Card

```
┌─────────────────────────────────────────────────────────────────┐
│                    DEERNGO BOT — QUICK REFERENCE                │
├─────────────────────────────────────────────────────────────────┤
│ SSH:        ssh flowero@remote.panomete.com                     │
│ Compose:    cd ~/platform && docker compose -f docker-compose   │
│               .deerngo.yml <command>                            │
│                                                                 │
│ HEALTH:                                                         │
│   Backend:  curl http://localhost:8008/api/v1/health            │
│   Frontend: curl http://localhost:3008                          │
│   Public:   curl -I https://deerngo-viewer-score.panomete.com  │
│                                                                 │
│ RESTART:    docker compose -f docker-compose.deerngo.yml restart │
│ LOGS:       docker logs --tail 100 deerngo-bot                  │
│ DB:         docker exec -it local-postgres psql -U postgres     │
│               -d deerngo                                        │
│ MIGRATE:    docker exec deerngo-bot ./deerngo-bot migrate up    │
│ CLEANUP:    docker system prune -f                              │
│                                                                 │
│ IMAGES:     ghcr.io/deerngo/deerngo-bot:latest                 │
│             ghcr.io/deerngo/deerngo-web:latest                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Related Documents

| Document | Relationship |
|----------|-------------|
| [[052_deployment_plan]] | How to deploy (CI/CD + manual) |
| [[051_CICD_pipeline_configuration]] | Automated pipeline config |
| [[053_release_notes]] | What changed per release |
| [[022_API_specification]] | API endpoint reference |
| [[023_database_schema_DDL]] | Database schema reference |

---

> **Template Standard:** Based on SWEBOK v4, SEBoK v2
> **Usage:** The runbook is the *operations bible*. If it's not in the runbook, it doesn't exist as a procedure. Keep it updated after every incident.
