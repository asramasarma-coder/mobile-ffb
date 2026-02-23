# Master Data Catalogue
**Version:** 1.0  
**Status:** Draft  
**Date:** February 2026  

---

## Overview

Master data is the foundation of the AgriField platform. All operational data (farms, visits, plants) depends on master data being correctly configured before the platform can be used. This document defines every master data entity, its attributes, ownership, and default seed data.

---

## Master Data Categories

| Category | Description | Managed By |
|---|---|---|
| System Master Data | Platform configuration — rarely changes | Super Admin |
| Organisational Master Data | Operational structure — grows over time | Admin / Super Admin |

---

## 1. Crop Types

The most critical master data entity. Drives tracking mode, visit templates, and growth stage definitions.

**Owner:** Super Admin  
**Seeded at launch:** Yes  
**Changes require:** Super Admin approval  

| Field | Type | Required | Description |
|---|---|---|---|
| `id` | UUID | ✅ | Unique identifier |
| `code` | string | ✅ | Short code e.g. `PALM`, `PADDY` |
| `name` | multilingual | ✅ | Display name per language |
| `tracking_level` | enum | ✅ | `field` or `plant` |
| `is_seasonal` | boolean | ✅ | True for paddy/corn, false for palm/rubber |
| `typical_lifespan_years` | integer | ✅ | Expected productive lifespan |
| `icon` | string | — | Icon identifier or image URL |
| `status` | enum | ✅ | `active` or `inactive` |
| `created_at` | timestamp | ✅ | Auto |
| `updated_at` | timestamp | ✅ | Auto |

### Seed Data

| Code | Name | Tracking Level | Seasonal | Lifespan |
|---|---|---|---|---|
| `PALM` | Oil Palm | plant | No | 25 years |
| `PADDY` | Paddy / Rice | field | Yes | 1 season |
| `RUBBER` | Rubber | plant | No | 30 years |
| `CORN` | Corn / Maize | field | Yes | 1 season |
| `COFFEE` | Coffee | plant | No | 20 years |
| `DURIAN` | Durian | plant | No | 40 years |

---

## 2. Visit Templates

Defines the data collection steps for each crop type. Rendered dynamically on the mobile app.

**Owner:** Admin (Super Admin publishes)  
**Seeded at launch:** Yes (one default per crop type)  
**Changes require:** New version — existing visits retain old version  

| Field | Type | Required | Description |
|---|---|---|---|
| `id` | UUID | ✅ | |
| `crop_type_id` | UUID → CropType | ✅ | |
| `name` | multilingual | ✅ | Template display name |
| `version` | integer | ✅ | Increments on each publish |
| `status` | enum | ✅ | `draft`, `active`, `archived` |
| `steps` | JSON array | ✅ | Ordered list of step definitions |
| `created_by` | UUID → User | ✅ | |
| `published_at` | timestamp | — | |
| `created_at` | timestamp | ✅ | |

### Visit Template Step Schema

```json
{
  "step_id": "s1",
  "order": 1,
  "type": "photo",
  "label": {
    "en": "Take a close-up photo of the crop",
    "ms": "Ambil gambar close-up tanaman",
    "id": "Ambil foto close-up tanaman"
  },
  "required": true,
  "options": null
}
```

**Step Types:**

| Type | Description | Options field |
|---|---|---|
| `photo` | Guided photo capture | `null` |
| `text` | Free text input | `null` |
| `number` | Numeric input with unit | `{ unit: "kg" }` |
| `select` | Single choice from list | `{ choices: ["Good","Fair","Poor"] }` |
| `multiselect` | Multiple choices | `{ choices: [...] }` |

### Seed Data — Default Steps per Crop Type

**Oil Palm (plant-level)**
1. Photo — bunch close-up (required)
2. Photo — frond overview (required)
3. Photo — field overview (optional)
4. Notes — free text (optional)

**Paddy / Rice (field-level)**
1. Photo — crop overview (required)
2. Photo — soil condition (required)
3. Photo — field wide shot (optional)
4. Notes — free text (optional)

---

## 3. Growth Stages

Defines the growth progression for each crop type. Used for AI result interpretation and schedule triggers.

**Owner:** Super Admin  
**Seeded at launch:** Yes  

| Field | Type | Required | Description |
|---|---|---|---|
| `id` | UUID | ✅ | |
| `crop_type_id` | UUID → CropType | ✅ | |
| `name` | multilingual | ✅ | Stage name |
| `code` | string | ✅ | e.g. `GERMINATION`, `VEGETATIVE` |
| `sequence_order` | integer | ✅ | 1 = first stage |
| `typical_duration_days` | integer | — | Average days in this stage |
| `reference_photo_url` | string | — | Example photo for field assistants |
| `created_at` | timestamp | ✅ | |

### Seed Data

**Oil Palm**

| Order | Code | Name | Typical Duration |
|---|---|---|---|
| 1 | `NURSERY` | Nursery | 365 days |
| 2 | `IMMATURE` | Immature | 1095 days |
| 3 | `YOUNG_MATURE` | Young Mature | 730 days |
| 4 | `PRIME` | Prime Productive | 3650 days |
| 5 | `SENESCENT` | Senescent | 1825 days |

**Paddy / Rice**

| Order | Code | Name | Typical Duration |
|---|---|---|---|
| 1 | `LAND_PREP` | Land Preparation | 14 days |
| 2 | `GERMINATION` | Germination | 7 days |
| 3 | `VEGETATIVE` | Vegetative | 40 days |
| 4 | `REPRODUCTIVE` | Reproductive | 30 days |
| 5 | `RIPENING` | Ripening | 30 days |
| 6 | `HARVEST` | Harvest Ready | 7 days |

---

## 4. Visit Schedule Rules

Defines how often each crop type must be visited and how overdue visits are flagged.

**Owner:** Admin  
**Seeded at launch:** Yes  

| Field | Type | Required | Description |
|---|---|---|---|
| `id` | UUID | ✅ | |
| `crop_type_id` | UUID → CropType | ✅ | |
| `frequency_days` | integer | ✅ | Days between visits |
| `trigger` | enum | ✅ | `calendar`, `growth_stage`, `manual` |
| `overdue_threshold_days` | integer | ✅ | Days past due before flagged overdue |
| `status` | enum | ✅ | `active` or `inactive` |
| `created_at` | timestamp | ✅ | |

### Seed Data

| Crop | Frequency | Trigger | Overdue Threshold |
|---|---|---|---|
| Oil Palm | 90 days | calendar | 14 days |
| Paddy | 21 days | growth_stage | 7 days |
| Rubber | 60 days | calendar | 14 days |
| Corn | 14 days | growth_stage | 5 days |
| Coffee | 30 days | calendar | 10 days |
| Durian | 30 days | calendar | 10 days |

---

## 5. Commodities

Links crop types to market commodity categories for price prediction.

**Owner:** Super Admin  
**Seeded at launch:** Yes  

| Field | Type | Required | Description |
|---|---|---|---|
| `id` | UUID | ✅ | |
| `name` | multilingual | ✅ | Commodity name |
| `code` | string | ✅ | Market code e.g. `CPO`, `RICE` |
| `unit` | enum | ✅ | `tonne`, `kg`, `bushel`, `litre` |
| `linked_crop_type_ids` | UUID[] | ✅ | One or more crop types |
| `status` | enum | ✅ | `active` or `inactive` |
| `created_at` | timestamp | ✅ | |

### Seed Data

| Code | Name | Unit | Linked Crops |
|---|---|---|---|
| `CPO` | Crude Palm Oil | tonne | Oil Palm |
| `RICE` | White Rice | tonne | Paddy |
| `RSS` | Ribbed Smoked Sheet (Rubber) | kg | Rubber |
| `CORN` | Corn / Maize | bushel | Corn |
| `COFFEE` | Arabica / Robusta Coffee | kg | Coffee |
| `DURIAN` | Fresh Durian | kg | Durian |

---

## 6. Help Content

Training and reference material for field assistants, organised per crop type.

**Owner:** Admin  
**Seeded at launch:** Optional — can be added post-launch  

| Field | Type | Required | Description |
|---|---|---|---|
| `id` | UUID | ✅ | |
| `crop_type_id` | UUID → CropType | — | null = applies to all crops |
| `title` | multilingual | ✅ | Content title |
| `type` | enum | ✅ | `video`, `pdf`, `article` |
| `url` | string | ✅ | Cloud Storage URL or external URL |
| `target_role` | enum | ✅ | `assistant`, `admin`, `all` |
| `sort_order` | integer | ✅ | Display order within crop type |
| `status` | enum | ✅ | `active` or `inactive` |
| `created_by` | UUID → User | ✅ | |
| `created_at` | timestamp | ✅ | |

---

## 7. Regions

Geographic hierarchy for grouping farms and assigning field assistants.

**Owner:** Super Admin  
**Seeded at launch:** Yes — must be configured before any farms can be registered  

| Field | Type | Required | Description |
|---|---|---|---|
| `id` | UUID | ✅ | |
| `name` | string | ✅ | Region name |
| `code` | string | ✅ | Short identifier |
| `parent_region_id` | UUID → Region | — | Null for top-level |
| `level` | enum | ✅ | `country`, `state`, `district`, `zone` |
| `boundary_coordinates` | GeoJSON | — | Optional polygon boundary |
| `status` | enum | ✅ | `active` or `inactive` |
| `created_at` | timestamp | ✅ | |

**Hierarchy example:**
```
Country: Malaysia
  └── State: Sabah
        └── District: Tawau
              └── Zone: Tawau North
```

---

## 8. Users

Platform users, linked to company SSO identities.

**Owner:** Super Admin (role assignment), SSO (identity)  
**Seeded at launch:** Created automatically on first SSO login  

| Field | Type | Required | Description |
|---|---|---|---|
| `id` | UUID | ✅ | Platform user ID |
| `sso_id` | string | ✅ | Identity from SSO provider |
| `email` | string | ✅ | From SSO |
| `name` | string | ✅ | From SSO |
| `role` | enum | ✅ | `super_admin`, `admin`, `field_assistant` |
| `language_preference` | string | ✅ | ISO 639-1 code e.g. `en`, `ms` |
| `assigned_region_id` | UUID → Region | — | Primary region |
| `status` | enum | ✅ | `pending`, `active`, `inactive` |
| `last_login_at` | timestamp | — | |
| `created_at` | timestamp | ✅ | |

**Status flow:**
```
SSO first login → status: pending
Super Admin assigns role → status: active
Super Admin deactivates → status: inactive
```

---

## 9. Farmers

Farm owners — the people who own the land being monitored.

**Owner:** Admin (created by Admin or Field Assistant, editable by Admin only)  

| Field | Type | Required | Description |
|---|---|---|---|
| `id` | UUID | ✅ | |
| `name` | string | ✅ | Full name |
| `phone` | string | — | Contact number |
| `national_id` | string | — | National ID or registration number |
| `region_id` | UUID → Region | ✅ | Home region |
| `status` | enum | ✅ | `active` or `inactive` |
| `created_by` | UUID → User | ✅ | |
| `created_at` | timestamp | ✅ | |

---

## Master Data Ownership Summary

| Entity | Create | Edit | Delete / Deactivate | Seed at Launch |
|---|---|---|---|---|
| Crop Types | Super Admin | Super Admin | Super Admin | ✅ |
| Visit Templates | Admin | Admin | Admin | ✅ |
| Growth Stages | Super Admin | Super Admin | Super Admin | ✅ |
| Schedule Rules | Admin | Admin | Admin | ✅ |
| Commodities | Super Admin | Super Admin | Super Admin | ✅ |
| Help Content | Admin | Admin | Admin | Optional |
| Regions | Super Admin | Super Admin | Super Admin | ✅ |
| Users | SSO auto + Super Admin | Super Admin | Super Admin | Auto |
| Farmers | Admin / Field Assistant | Admin | Admin | No |

---

## Master Data Change Policy

- **Crop type tracking level** cannot be changed after any farm has been registered against it
- **Visit template steps** cannot be deleted if visits exist using that template version — create a new version instead
- **Growth stage sequence** should not be reordered after AI analysis results have been stored referencing those stages
- **Region hierarchy** changes require review of all farm assignments in affected regions
- **User role changes** take effect on next login

---

*AgriField — Commodity Intelligence Platform | Confidential — Internal Use Only*