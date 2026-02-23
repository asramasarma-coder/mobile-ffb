# Offline & Sync Specification
**Version:** 1.0  
**Status:** Draft  
**Date:** February 2026  

---

## Overview

Field assistants frequently operate in areas with no mobile connectivity. The app must work fully offline and sync automatically when connectivity returns. This document defines the offline storage model, sync engine behaviour, conflict resolution rules, and error handling.

---

## 1. Principles

- **Offline-first** — every feature works without internet. Connectivity is treated as a bonus, not a requirement.
- **Local writes first** — all data is written to local SQLite (WatermelonDB) before any network call is made.
- **Server is authoritative for master data** — crop types, templates, farm profiles sync down from server.
- **Device is authoritative for new visits** — visits created on device are trusted and uploaded as-is.
- **Idempotent uploads** — every visit and registration has a `client_id` (UUID generated on device). Re-uploading the same `client_id` is safe — server deduplicates.
- **Never lose data** — if upload fails, it stays in queue and retries. Data is never discarded.

---

## 2. Local Data Categories

| Category | Direction | Storage | Notes |
|---|---|---|---|
| Master data (crop types, templates) | Server → Device | WatermelonDB | Read-only on device |
| Assigned farms & plants | Server → Device | WatermelonDB | Refreshed on sync pull |
| Scheduled visits | Server → Device | WatermelonDB | Refreshed on sync pull |
| Visits (new) | Device → Server | WatermelonDB | Created locally, uploaded |
| Photos (new) | Device → Cloud Storage | Local filesystem + queue | Compressed, uploaded separately |
| Registrations (new) | Device → Server | WatermelonDB | Uploaded for approval |
| Approval status updates | Server → Device | WatermelonDB | Pulled during sync |
| AI analysis results | Server → Device | WatermelonDB | Returned after visit upload |

---

## 3. Sync Engine Architecture

```
┌─────────────────────────────────────────────────────┐
│                   Sync Engine                        │
│                                                     │
│   Network Monitor                                   │
│       ↓ (on connectivity detected)                  │
│   Sync Coordinator                                  │
│       ↓                                             │
│   ┌─────────────────────────────────────────────┐  │
│   │  Phase 1: Push (Device → Server)            │  │
│   │  ├── Upload pending visits                  │  │
│   │  ├── Upload pending registrations           │  │
│   │  └── Upload pending photos                  │  │
│   └─────────────────────────────────────────────┘  │
│       ↓                                             │
│   ┌─────────────────────────────────────────────┐  │
│   │  Phase 2: Pull (Server → Device)            │  │
│   │  ├── Pull master data updates               │  │
│   │  ├── Pull farm / plant updates              │  │
│   │  ├── Pull scheduled visit updates           │  │
│   │  └── Pull approval status updates           │  │
│   └─────────────────────────────────────────────┘  │
│       ↓                                             │
│   Update local DB + notify UI                       │
└─────────────────────────────────────────────────────┘
```

---

## 4. Sync Triggers

| Trigger | Action |
|---|---|
| App foreground (from background) | Full sync if last sync > 15 minutes ago |
| Network connectivity restored | Full sync immediately |
| User taps "Sync Now" manually | Full sync immediately |
| Visit submitted | Attempt immediate push of that visit |
| App launch | Full sync if last sync > 30 minutes ago |
| Background app refresh (iOS/Android) | Sync if items pending |

---

## 5. Push Phase — Upload Queue

### Visit Upload

```
For each visit where sync_status = 'pending_upload':
    1. POST /sync/push with visit payload
    2. On 201 success:
        - Update local visit: server_id = response.server_id, sync_status = 'synced'
        - Trigger photo upload queue for this visit
    3. On 409 CONFLICT (client_id already exists):
        - Update local visit: sync_status = 'synced' (already uploaded)
    4. On 4xx (non-409):
        - Update local visit: sync_status = 'error', sync_error = error.message
        - Do NOT retry automatically — surface to user in Sync Status screen
    5. On 5xx or network error:
        - Increment retry_count
        - Schedule retry with exponential backoff: 1min → 5min → 15min → 60min
        - After 5 retries: mark as sync_status = 'error', notify user
```

### Photo Upload

Photos are uploaded separately from visit metadata to avoid large payloads and allow independent retry.

```
For each photo where upload_status = 'pending':
    1. GET /visits/:visit_id/photos/:photo_id/signed-url
    2. Compress photo to max 1280px on longest edge, JPEG quality 80
    3. PUT compressed photo to signed Cloud Storage URL
    4. POST /visits/:visit_id/photos/:photo_id/confirm
    5. On success:
        - Update local photo: upload_status = 'uploaded', storage_path = path
        - Delete local compressed copy (keep original for retry)
    6. On failure:
        - Increment retry_count
        - Exponential backoff retry
        - After 5 retries: mark upload_status = 'failed', surface in sync screen
```

### Registration Upload

Same pattern as visit upload — idempotent via `client_id`, queued and retried on failure.

---

## 6. Pull Phase — Download Updates

```
GET /sync/pull?updated_after={last_sync_timestamp}

Process response:
    1. Upsert farms (insert if new, update if changed)
    2. Upsert plants
    3. Upsert scheduled visits
    4. Process approval_updates:
        - Find matching pending_registration by client_id
        - Update status to 'approved' or 'rejected'
        - If approved: set server_id, trigger notification
        - If rejected: set rejection_reason, trigger notification
    5. Update last_sync_timestamp in local storage
```

### Master Data Sync

Master data is synced separately with a longer interval (once per session or manually):

```
GET /master-data/sync?updated_after={last_master_data_sync}

Process response:
    1. Upsert all crop types
    2. Upsert all active visit templates
    3. Upsert all growth stages
    4. Update last_master_data_sync timestamp
```

---

## 7. Conflict Resolution Rules

| Scenario | Resolution |
|---|---|
| Visit uploaded with same `client_id` twice | Server returns 409. Client marks as synced. No duplicate created. |
| Farm updated on server while client has cached copy | Server version wins on pull. Client cache overwritten. |
| Agent submits visit for a farm that was deactivated | Server accepts visit (historical record). Farm flagged as inactive on next pull. |
| Template version changed while visit in progress | Visit is submitted with the version it started with. Server stores the template version used. |
| Photo upload fails after visit metadata succeeds | Photo retried independently. Visit metadata already stored. AI analysis triggered only after all required photos uploaded. |
| Two agents submit visits for same farm simultaneously | Both accepted. Server records both with different `visited_at` timestamps. No merge. |

---

## 8. Sync Status UI

The Sync Status screen gives the field assistant full visibility:

```
┌─────────────────────────────────────┐
│  Sync Status                         │
│                                      │
│  Last synced: Today 10:32 AM    ✅   │
│                                      │
│  Pending uploads:                    │
│  ┌─────────────────────────────┐    │
│  │ 📋 2 visits                 │    │
│  │ 📷 7 photos (12.4 MB)       │    │
│  │ 🏠 1 registration           │    │
│  └─────────────────────────────┘    │
│                                      │
│  ❌ Errors (1):                      │
│  ┌─────────────────────────────┐    │
│  │ Visit — Ladang Maju         │    │
│  │ Failed: server error        │    │
│  │ [Retry]                     │    │
│  └─────────────────────────────┘    │
│                                      │
│  [Sync Now]                          │
└─────────────────────────────────────┘
```

---

## 9. Offline Indicators

The app communicates connectivity state clearly at all times:

| State | Indicator |
|---|---|
| Online, synced | No banner (clean) |
| Online, sync in progress | Blue banner: "Syncing…" with progress |
| Online, sync complete | Brief green banner: "Synced" (auto-dismiss 2s) |
| Offline | Persistent orange banner: "You're offline — data will sync when connected" |
| Sync error | Persistent red banner: "Sync error — tap to view" → Sync Status screen |

---

## 10. Data Size Constraints

| Item | Limit | Handling |
|---|---|---|
| Photo file size (before compression) | No limit | Compressed to max 1280px, JPEG 80% before upload |
| Photo file size (after compression) | ~200-400KB typical | Uploaded via Cloud Storage signed URL (bypasses API) |
| Visits per sync batch | 50 max | Batched automatically |
| Photos per visit | 10 max | Enforced by visit template |
| Local DB size (all visits) | No hard limit | WatermelonDB / SQLite handles large datasets |
| Offline visit queue | No hard limit | All pending visits retained until uploaded |

---

## 11. Security Considerations

- JWT tokens stored in device secure storage (Expo SecureStore)
- Signed photo upload URLs expire after 60 minutes — new URL requested if expired
- Local SQLite database is not encrypted by default — consider enabling SQLCipher for sensitive deployments
- Photos deleted from local filesystem after confirmed upload to Cloud Storage
- Sync endpoint validates JWT on every request — expired tokens trigger re-authentication

---

*AgriField — Commodity Intelligence Platform | Confidential — Internal Use Only*