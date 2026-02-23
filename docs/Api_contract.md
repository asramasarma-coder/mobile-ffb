# API Contract
**Version:** 1.0  
**Status:** Draft  
**Date:** February 2026  

---

## Overview

This document defines all API endpoints consumed by the AgriField mobile app and web dashboard. All services run on GCP Cloud Run behind an API Gateway.

---

## Conventions

### Base URLs

| Environment | Base URL |
|---|---|
| Development | `https://api.dev.agrifield.com/v1` |
| Staging | `https://api.staging.agrifield.com/v1` |
| Production | `https://api.agrifield.com/v1` |

### Authentication

All requests must include a Firebase JWT token in the Authorization header:

```
Authorization: Bearer <firebase_jwt_token>
```

The token is obtained after SSO login via Firebase Auth. The server validates the token on every request and extracts the user's role and ID.

### Request / Response Format

- All request and response bodies are `application/json`
- All timestamps are ISO 8601 UTC e.g. `2026-02-23T10:30:00Z`
- All IDs are UUIDs
- Multilingual fields return the full language map object: `{ "en": "...", "ms": "..." }`

### Pagination

List endpoints use cursor-based pagination:

```json
{
  "data": [...],
  "pagination": {
    "limit": 20,
    "cursor": "eyJpZCI6Ii4uLiJ9",
    "has_more": true
  }
}
```

Pass `cursor` as a query parameter to get the next page: `?cursor=eyJpZCI6Ii4uLiJ9`

### Error Format

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Human readable message",
    "details": { "field": "reason" }
  }
}
```

### Standard Error Codes

| HTTP | Code | Description |
|---|---|---|
| 400 | `VALIDATION_ERROR` | Invalid request body or params |
| 401 | `UNAUTHORIZED` | Missing or invalid JWT |
| 403 | `FORBIDDEN` | Valid token but insufficient role |
| 404 | `NOT_FOUND` | Resource does not exist |
| 409 | `CONFLICT` | Duplicate record (e.g. client_id already exists) |
| 422 | `UNPROCESSABLE` | Business rule violation |
| 500 | `INTERNAL_ERROR` | Server error |

---

## 1. Auth & Users

### `GET /users/me`

Returns the authenticated user's profile and role.

**Response `200`**
```json
{
  "id": "uuid",
  "sso_id": "string",
  "email": "user@company.com",
  "name": "Ahmad Farid",
  "role": "field_assistant",
  "language_preference": "ms",
  "assigned_region": {
    "id": "uuid",
    "name": "Tawau North",
    "code": "TWN"
  },
  "status": "active"
}
```

---

### `PATCH /users/me`

Update language preference.

**Request**
```json
{
  "language_preference": "ms"
}
```

**Response `200`** — updated user object

---

## 2. Master Data

Master data is downloaded on login and cached locally. Mobile app refreshes on each session start.

### `GET /master-data/sync`

Returns all active master data needed by the mobile app in a single call. Used for initial sync and refresh.

**Query params:** `?updated_after=2026-02-01T00:00:00Z` (optional — for delta sync)

**Response `200`**
```json
{
  "crop_types": [...],
  "visit_templates": [...],
  "growth_stages": [...],
  "schedule_rules": [...],
  "help_content": [...],
  "synced_at": "2026-02-23T10:30:00Z"
}
```

---

### `GET /crop-types`

**Query params:** `status=active`

**Response `200`**
```json
{
  "data": [
    {
      "id": "uuid",
      "code": "PALM",
      "name": { "en": "Oil Palm", "ms": "Kelapa Sawit" },
      "tracking_level": "plant",
      "is_seasonal": false,
      "typical_lifespan_years": 25,
      "icon": "palm",
      "status": "active"
    }
  ]
}
```

---

### `GET /visit-templates`

**Query params:** `crop_type_id=uuid&status=active`

**Response `200`**
```json
{
  "data": [
    {
      "id": "uuid",
      "crop_type_id": "uuid",
      "name": { "en": "Oil Palm Visit", "ms": "Lawatan Kelapa Sawit" },
      "version": 3,
      "status": "active",
      "steps": [
        {
          "step_id": "s1",
          "order": 1,
          "type": "photo",
          "label": { "en": "Take a close-up of the bunch", "ms": "Ambil gambar tandan" },
          "required": true,
          "options": null
        }
      ]
    }
  ]
}
```

---

## 3. Farms

### `GET /farms/nearby`

Returns farms and plants within a given radius of a GPS coordinate. Primary GPS discovery endpoint.

**Query params:**
- `lat` (required) — decimal latitude
- `lng` (required) — decimal longitude
- `radius_m` — radius in metres, default `200`

**Response `200`**
```json
{
  "data": [
    {
      "id": "uuid",
      "type": "farm",
      "name": "Ladang Maju",
      "crop_type": { "code": "PALM", "name": { "en": "Oil Palm" }, "tracking_level": "plant" },
      "gps_latitude": 4.2468,
      "gps_longitude": 117.8892,
      "distance_metres": 145,
      "status": "active",
      "last_visit": {
        "visited_at": "2026-01-10T08:00:00Z",
        "health_score": 82.5
      },
      "is_overdue": true,
      "assigned_agent_id": "uuid"
    }
  ]
}
```

---

### `GET /farms/:id`

Full farm profile.

**Response `200`**
```json
{
  "id": "uuid",
  "name": "Ladang Maju",
  "farmer": { "id": "uuid", "name": "Haji Roslan" },
  "region": { "id": "uuid", "name": "Tawau North" },
  "crop_type": { "id": "uuid", "code": "PALM", "name": {...} },
  "tracking_level": "plant",
  "boundary_polygon": { "type": "Polygon", "coordinates": [[...]] },
  "gps_latitude": 4.2468,
  "gps_longitude": 117.8892,
  "total_area_hectares": 12.5,
  "assigned_agent": { "id": "uuid", "name": "Ahmad Farid" },
  "status": "active",
  "created_at": "2025-03-01T00:00:00Z"
}
```

---

### `GET /farms/:id/visits`

Visit history for a farm.

**Query params:** `limit=20&cursor=...`

**Response `200`**
```json
{
  "data": [
    {
      "id": "uuid",
      "subject_type": "farm",
      "agent": { "id": "uuid", "name": "Ahmad Farid" },
      "visited_at": "2026-02-10T09:30:00Z",
      "status": "analysed",
      "analysis": {
        "crop_health_score": 82.5,
        "growth_stage_code": "PRIME",
        "predicted_yield_kg": 2400,
        "anomalies": []
      },
      "photo_count": 3
    }
  ],
  "pagination": { "limit": 20, "cursor": "...", "has_more": false }
}
```

---

### `POST /farms/register`

Register a new farm. Creates with `pending_approval` status.

**Request**
```json
{
  "client_id": "uuid",
  "name": "Ladang Baru 1",
  "farmer_id": "uuid",
  "crop_type_id": "uuid",
  "gps_latitude": 4.2468,
  "gps_longitude": 117.8892,
  "total_area_hectares": 8.0,
  "notes": "Near the river"
}
```

**Response `201`**
```json
{
  "id": "uuid",
  "status": "pending_approval",
  "created_at": "2026-02-23T10:30:00Z"
}
```

---

### `PATCH /farms/:id`

Update farm details. Admin only.

### `PATCH /farms/:id/assign`

Assign a field assistant to a farm. Admin only.

**Request**
```json
{ "agent_id": "uuid" }
```

---

## 4. Plants

### `GET /farms/:farm_id/plants`

List all plants for a farm.

**Query params:** `status=active&limit=50&cursor=...`

**Response `200`**
```json
{
  "data": [
    {
      "id": "uuid",
      "plant_code": "A-042",
      "gps_latitude": 4.24685,
      "gps_longitude": 117.88925,
      "planting_date": "2019-01-15",
      "status": "active",
      "last_visit": {
        "visited_at": "2026-01-10T08:00:00Z",
        "health_score": 90.0
      }
    }
  ],
  "pagination": { "limit": 50, "cursor": null, "has_more": false }
}
```

---

### `GET /plants/:id`

Full plant profile including visit history.

### `POST /plants/register`

Register a new plant.

**Request**
```json
{
  "client_id": "uuid",
  "farm_id": "uuid",
  "gps_latitude": 4.24685,
  "gps_longitude": 117.88925,
  "planting_date": "2019-01-15",
  "notes": "Near east boundary"
}
```

**Response `201`**
```json
{
  "id": "uuid",
  "plant_code": "A-043",
  "status": "pending_approval"
}
```

---

## 5. Visits

### `POST /visits`

Submit a completed visit. Idempotent via `client_id`.

**Request**
```json
{
  "client_id": "uuid",
  "subject_type": "plant",
  "subject_id": "uuid",
  "template_id": "uuid",
  "template_version": 3,
  "gps_latitude": 4.24685,
  "gps_longitude": 117.88925,
  "gps_accuracy_meters": 4.2,
  "visit_data": {
    "s1": { "type": "photo", "photo_ids": ["uuid", "uuid"] },
    "s2": { "type": "photo", "photo_ids": ["uuid"] },
    "s3": { "type": "text", "value": "Looks healthy" }
  },
  "notes": "Some additional observations",
  "visited_at": "2026-02-23T09:30:00Z"
}
```

**Response `201`**
```json
{
  "id": "uuid",
  "status": "submitted",
  "created_at": "2026-02-23T10:31:00Z"
}
```

---

### `GET /visits/:id`

Full visit detail including AI analysis results.

**Response `200`**
```json
{
  "id": "uuid",
  "subject_type": "plant",
  "subject_id": "uuid",
  "agent": { "id": "uuid", "name": "Ahmad Farid" },
  "visited_at": "2026-02-23T09:30:00Z",
  "visit_data": { ... },
  "notes": "Some additional observations",
  "photos": [
    {
      "id": "uuid",
      "step_id": "s1",
      "url": "https://storage.googleapis.com/..."
    }
  ],
  "analysis": {
    "status": "complete",
    "crop_health_score": 87.5,
    "growth_stage_code": "PRIME",
    "predicted_yield_kg": 2600,
    "anomalies": [
      { "type": "pest_detected", "confidence": 0.62, "description": "Possible bagworm signs" }
    ],
    "analysed_at": "2026-02-23T10:35:00Z"
  }
}
```

---

### `GET /visits/:id/photos/:photo_id/signed-url`

Get a signed upload URL for a photo. Mobile uploads directly to Cloud Storage.

**Response `200`**
```json
{
  "photo_id": "uuid",
  "upload_url": "https://storage.googleapis.com/agrifield-photos/...",
  "expires_at": "2026-02-23T11:30:00Z"
}
```

---

### `POST /visits/:id/photos/:photo_id/confirm`

Notify server that photo upload to Cloud Storage is complete.

**Response `200`**
```json
{
  "photo_id": "uuid",
  "upload_status": "uploaded"
}
```

---

## 6. Sync (Bulk)

### `POST /sync/push`

Bulk upload of offline-collected visits and registrations. Used by the sync engine.

**Request**
```json
{
  "visits": [
    { ...visit object... },
    { ...visit object... }
  ],
  "registrations": [
    { "type": "farm", ...farm registration... },
    { "type": "plant", ...plant registration... }
  ]
}
```

**Response `200`**
```json
{
  "visits": [
    { "client_id": "uuid", "server_id": "uuid", "status": "synced" },
    { "client_id": "uuid", "server_id": null, "status": "error", "error": "DUPLICATE" }
  ],
  "registrations": [
    { "client_id": "uuid", "server_id": "uuid", "status": "submitted" }
  ]
}
```

---

### `GET /sync/pull`

Pull latest server-side updates for the agent's assigned farms and plants.

**Query params:** `updated_after=2026-02-22T00:00:00Z`

**Response `200`**
```json
{
  "farms": [...],
  "plants": [...],
  "scheduled_visits": [...],
  "approval_updates": [
    {
      "client_id": "uuid",
      "server_id": "uuid",
      "type": "farm",
      "status": "approved",
      "rejection_reason": null
    }
  ]
}
```

---

## 7. Approvals

### `GET /approvals`

List pending registrations. Admin only.

**Query params:** `type=farm|plant&region_id=uuid&limit=20&cursor=...`

**Response `200`**
```json
{
  "data": [
    {
      "id": "uuid",
      "type": "farm",
      "submitted_by": { "id": "uuid", "name": "Ahmad Farid" },
      "submitted_at": "2026-02-23T08:00:00Z",
      "details": {
        "name": "Ladang Baru 1",
        "crop_type": { "code": "PALM", "name": {...} },
        "gps_latitude": 4.2468,
        "gps_longitude": 117.8892
      },
      "photo_count": 2
    }
  ],
  "pagination": { "limit": 20, "cursor": null, "has_more": false }
}
```

---

### `POST /approvals/:id/approve`

Approve a registration. Admin only.

**Response `200`**
```json
{
  "id": "uuid",
  "status": "approved",
  "approved_at": "2026-02-23T10:00:00Z"
}
```

---

### `POST /approvals/:id/reject`

Reject a registration. Admin only.

**Request**
```json
{ "reason": "GPS location does not match known farm boundaries" }
```

**Response `200`**
```json
{
  "id": "uuid",
  "status": "rejected",
  "rejection_reason": "GPS location does not match known farm boundaries"
}
```

---

## 8. Scheduled Visits

### `GET /scheduled-visits`

**Query params:** `agent_id=uuid&status=overdue|upcoming&limit=20`

**Response `200`**
```json
{
  "data": [
    {
      "id": "uuid",
      "subject_type": "farm",
      "subject": { "id": "uuid", "name": "Ladang Maju", "crop_type": {...} },
      "agent": { "id": "uuid", "name": "Ahmad Farid" },
      "due_date": "2026-02-15",
      "status": "overdue",
      "days_overdue": 8
    }
  ]
}
```

---

### `POST /scheduled-visits`

Create a scheduled visit. Admin only.

**Request**
```json
{
  "subject_type": "farm",
  "subject_id": "uuid",
  "agent_id": "uuid",
  "due_date": "2026-03-15"
}
```

---

### `PATCH /scheduled-visits/:id/assign`

Reassign to a different agent. Admin only.

**Request**
```json
{ "agent_id": "uuid" }
```

---

## 9. Price Predictions

### `GET /predictions`

**Query params:** `commodity_id=uuid&region_id=uuid&from=2026-01-01&to=2026-03-01`

**Response `200`**
```json
{
  "data": [
    {
      "id": "uuid",
      "commodity": { "id": "uuid", "code": "CPO", "name": {...}, "unit": "tonne" },
      "region": { "id": "uuid", "name": "Sabah" },
      "predicted_price": 3850.00,
      "price_unit": "MYR/tonne",
      "confidence": 78.5,
      "prediction_date": "2026-02-23",
      "forecast_period": "monthly",
      "contributing_data": {
        "farms_sampled": 142,
        "avg_health_score": 81.2,
        "growth_stage_distribution": { "PRIME": 0.68, "YOUNG_MATURE": 0.22, "SENESCENT": 0.10 }
      }
    }
  ]
}
```

---

## 10. AI Integration (Internal — ai-dispatcher service)

This endpoint is called internally by `api-core` via Cloud Tasks. Not directly accessible by the mobile app.

### `POST /internal/analyse`

**Request**
```json
{
  "visit_id": "uuid",
  "crop_type_code": "PALM",
  "gps_latitude": 4.24685,
  "gps_longitude": 117.88925,
  "visited_at": "2026-02-23T09:30:00Z",
  "photo_urls": [
    "https://storage.googleapis.com/agrifield-photos-raw/..."
  ]
}
```

**Response `200`** (from AI API — stored in `visit_analyses`)
```json
{
  "ai_request_id": "ai-abc-123",
  "crop_health_score": 87.5,
  "predicted_yield_kg": 2600,
  "growth_stage_code": "PRIME",
  "anomalies": [
    {
      "type": "pest_detected",
      "confidence": 0.62,
      "description": "Possible bagworm signs detected in photo 2"
    }
  ]
}
```

---

## 11. Notifications

### `POST /notifications/register-device`

Register a device FCM token for push notifications.

**Request**
```json
{
  "fcm_token": "string",
  "platform": "android | ios",
  "app_version": "1.0.0"
}
```

**Response `200`**
```json
{ "registered": true }
```

---

*AgriField — Commodity Intelligence Platform | Confidential — Internal Use Only*