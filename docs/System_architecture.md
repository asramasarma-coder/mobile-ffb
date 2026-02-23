# System Architecture Document
**Version:** 1.0  
**Status:** Draft  
**Date:** February 2026  

---

## Overview

AgriField runs entirely on GCP serverless infrastructure. The mobile app (React Native / Expo) communicates with a suite of Cloud Run microservices behind an API Gateway. All data is stored in Cloud SQL (PostgreSQL) and Cloud Storage. Async processing is handled via Cloud Tasks and Pub/Sub.

---

## 1. High-Level Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                     CLIENT LAYER                              │
│                                                              │
│  ┌─────────────────────┐    ┌──────────────────────────┐    │
│  │  React Native App   │    │   Web Admin Dashboard    │    │
│  │  (iOS + Android)    │    │   (React — Phase 2)      │    │
│  └──────────┬──────────┘    └────────────┬─────────────┘    │
│             │ HTTPS                       │ HTTPS            │
└─────────────┼───────────────────────────-┼──────────────────┘
              │                             │
              ▼                             ▼
┌─────────────────────────────────────────────────────────────┐
│               GCP API GATEWAY + CLOUD ARMOR                  │
│         (Auth validation, rate limiting, WAF, TLS)           │
└──────────┬────────────┬──────────────┬──────────────────────┘
           │            │              │
           ▼            ▼              ▼
┌──────────────┐ ┌─────────────┐ ┌───────────────┐
│  api-core    │ │  api-sync   │ │  admin-api    │
│  (Cloud Run) │ │  (Cloud Run)│ │  (Cloud Run)  │
└──────┬───────┘ └──────┬──────┘ └───────┬───────┘
       │                │                │
       ▼                ▼                ▼
┌─────────────────────────────────────────────────┐
│              Cloud SQL (PostgreSQL)              │
│              via Cloud SQL Auth Proxy            │
└─────────────────────────────────────────────────┘
       │
       ▼ (on visit submit)
┌──────────────────────┐
│    Cloud Tasks       │
│  (AI dispatch queue) │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐      ┌─────────────────────┐
│   ai-dispatcher      │─────►│   External AI API    │
│   (Cloud Run)        │      │   (HTTPS)            │
└──────────────────────┘      └─────────────────────┘
           │
           ▼ (results stored)
    Cloud SQL + Pub/Sub
           │
           ▼
┌──────────────────────┐
│  notification-service│
│  (Cloud Run)         │
└──────────┬───────────┘
           │
           ▼
    Firebase Cloud Messaging
    (Push to iOS + Android)

┌──────────────────────────────────────────────────┐
│              Cloud Storage                        │
│   photos-raw/  photos-processed/  help-content/  │
└──────────────────────────────────────────────────┘

┌──────────────────────┐
│   Cloud Scheduler    │ → scheduler service (Cloud Run)
│   (Cron jobs)        │   generates visit schedules daily
└──────────────────────┘

┌──────────────────────┐
│   Memorystore        │ → Redis cache for master data
│   (Redis)            │   and session tokens
└──────────────────────┘
```

---

## 2. Cloud Run Services

| Service | Description | Trigger | Scales To |
|---|---|---|---|
| `api-core` | Main REST API — farms, visits, users, master data | HTTP | 0–50 instances |
| `api-sync` | Bulk offline sync push/pull endpoint | HTTP | 0–20 instances |
| `admin-api` | Admin operations — approvals, schedule mgmt | HTTP | 0–10 instances |
| `ai-dispatcher` | Calls external AI API, stores results | Cloud Tasks | 0–20 instances |
| `scheduler` | Generates scheduled visits from schedule rules | Cloud Scheduler (cron) | 0–2 instances |
| `notification-service` | Sends FCM push notifications | Pub/Sub | 0–10 instances |
| `export-service` | Generates CSV/PDF reports | HTTP | 0–5 instances |

### Service Configuration

All Cloud Run services are configured with:

```yaml
# Common config applied to all services
minInstances: 0           # Scale to zero when idle
maxInstances: 50          # Hard cap per service (adjust per service)
timeoutSeconds: 60        # Request timeout
memory: 512Mi             # Memory per instance
cpu: 1                    # vCPU per instance
concurrency: 80           # Max concurrent requests per instance
```

**`ai-dispatcher` specific:**
```yaml
timeoutSeconds: 300       # AI API calls can be slow
memory: 1Gi
concurrency: 10           # Lower — external API bound
```

---

## 3. Data Flow — Visit Submission

```
1. Agent submits visit in app
       ↓
2. Stored locally (WatermelonDB) with sync_status: 'pending_upload'
       ↓
3. Sync engine triggers on connectivity
       ↓
4. POST /sync/push → api-sync service
       ↓
5. api-sync writes visit to Cloud SQL
   → Returns server_id to mobile
   → Mobile updates local record: sync_status = 'synced'
       ↓
6. api-sync enqueues task to Cloud Tasks: { visit_id }
       ↓
7. Cloud Tasks calls ai-dispatcher
       ↓
8. ai-dispatcher fetches photo signed URLs from Cloud Storage
   → Calls external AI API with photos + metadata
   → Stores analysis results in visit_analyses table
       ↓
9. ai-dispatcher publishes event to Pub/Sub: 'visit.analysed'
       ↓
10. notification-service subscribes to 'visit.analysed'
    → Sends FCM push to agent: "Your visit results are ready"
       ↓
11. Agent opens Visit Result screen
    → GET /visits/:id → api-core returns full visit + analysis
```

---

## 4. Data Flow — Photo Upload

```
1. Agent takes photo in app
       ↓
2. Saved to local filesystem (compressed)
       ↓
3. Sync engine triggers
       ↓
4. GET /visits/:id/photos/:photo_id/signed-url → api-core
   → api-core generates Cloud Storage signed URL (60 min expiry)
       ↓
5. Mobile PUT photo bytes directly to Cloud Storage signed URL
   (bypasses API server — efficient, no API payload overhead)
       ↓
6. POST /visits/:id/photos/:photo_id/confirm → api-core
   → api-core updates visit_photos: upload_status = 'uploaded'
       ↓
7. When all required photos uploaded:
   → Cloud Storage trigger fires → Pub/Sub → ai-dispatcher
   → AI analysis begins
```

---

## 5. Data Flow — Farm Registration & Approval

```
1. Field assistant registers new farm
       ↓
2. POST /farms/register → api-core
   → Farm created with status: 'pending_approval'
   → Pub/Sub event: 'farm.registration.submitted'
       ↓
3. notification-service → FCM push to all admins in region:
   "New farm registration pending approval"
       ↓
4. Admin opens Approval Queue → Approval Detail
   → Reviews GPS, photos, details
       ↓
5. Admin taps Approve
   → POST /approvals/:id/approve → admin-api
   → Farm status updated to 'active'
   → Pub/Sub event: 'farm.registration.approved'
       ↓
6. notification-service → FCM push to field assistant:
   "Your farm registration was approved"
       ↓
7. Next sync pull: mobile receives farm in assigned farms list
```

---

## 6. Caching Strategy

**Memorystore (Redis)** caches frequently-read, rarely-changed data:

| Cache Key | Data | TTL | Invalidated By |
|---|---|---|---|
| `master:crop_types` | All active crop types | 1 hour | Admin edits crop type |
| `master:templates:{crop_id}` | Active template for crop | 1 hour | Admin publishes template |
| `user:{user_id}` | User record + role | 15 min | Role change, deactivation |
| `nearby:{lat_grid}:{lng_grid}` | Farms in GPS grid cell | 5 min | Farm registered/updated |
| `predictions:{commodity_id}` | Latest price predictions | 30 min | New prediction generated |

Cache-aside pattern — check cache first, fall through to DB on miss, populate cache on DB read.

---

## 7. GCP Project Structure

```
org: agrifield
  │
  ├── agrifield-dev
  │     ├── Cloud Run (all services, dev builds)
  │     ├── Cloud SQL (dev instance — shared)
  │     ├── Cloud Storage (dev buckets)
  │     ├── Cloud Tasks (dev queues)
  │     ├── Firebase (dev project)
  │     └── Cloud Build (triggers on feature branches)
  │
  ├── agrifield-staging
  │     ├── Cloud Run (all services, release candidates)
  │     ├── Cloud SQL (staging instance — prod-like size)
  │     ├── Cloud Storage (staging buckets)
  │     ├── Cloud Tasks (staging queues)
  │     ├── Firebase (staging project)
  │     └── Cloud Build (triggers on main branch)
  │
  └── agrifield-prod
        ├── Cloud Run (all services, production)
        ├── Cloud SQL (HA — primary + read replica)
        ├── Cloud Storage (production buckets, versioning enabled)
        ├── Cloud Tasks (production queues)
        ├── Firebase (production project)
        ├── Cloud Armor (WAF)
        ├── Memorystore (Redis)
        └── Cloud Build (triggers on release tags)
```

---

## 8. CI/CD Pipeline

```
Developer pushes code
       ↓
GitHub PR created
       ↓
Cloud Build trigger (PR)
  ├── Run unit tests
  ├── Run translation completeness check
  ├── Run linting
  └── Build Docker image (dry run)
       ↓
PR reviewed + merged to main
       ↓
Cloud Build trigger (main branch)
  ├── Run full test suite
  ├── Build Docker images for all changed services
  ├── Push images to Artifact Registry
  └── Deploy to agrifield-staging (Cloud Run)
       ↓
QA sign-off on staging
       ↓
Tag release: git tag v1.x.x
       ↓
Cloud Build trigger (release tag)
  ├── Deploy to agrifield-prod (Cloud Run)
  └── Run smoke tests
       ↓
Production deployment complete
```

---

## 9. Observability

| Tool | Purpose |
|---|---|
| Cloud Logging | Structured logs from all Cloud Run services |
| Cloud Monitoring | Metrics — latency, error rates, instance count |
| Cloud Trace | Distributed tracing across services |
| Error Reporting | Automatic error grouping and alerting |
| Uptime Checks | Synthetic monitoring on key endpoints |

### Key Alerts

| Alert | Threshold | Channel |
|---|---|---|
| API error rate | > 1% over 5 min | PagerDuty + Slack |
| Cloud SQL CPU | > 80% over 10 min | Slack |
| Sync queue depth | > 1000 tasks | Slack |
| AI dispatcher failures | > 10 failures in 1 hour | Slack |
| Cloud Run instance count | Hitting max instances | Slack |

---

## 10. Disaster Recovery

| Component | Backup | RTO | RPO |
|---|---|---|---|
| Cloud SQL (prod) | Automated daily backup + PITR | 1 hour | 1 hour |
| Cloud Storage | Object versioning enabled | Immediate | 0 (versioned) |
| Cloud Run services | Stateless — redeploy from Artifact Registry | 10 min | N/A |
| Redis cache | Cache-aside — DB is source of truth | Immediate | N/A |

---

*AgriField — Commodity Intelligence Platform | Confidential — Internal Use Only*