# Information Architecture
**Version:** 1.0  
**Status:** Draft  
**Date:** February 2026  

---

## Overview

This document defines the full navigation structure, screen hierarchy, and content organisation of the AgriField platform. It covers the mobile app (Phase 1) and web admin dashboard (Phase 2).

---

## 1. Navigation Structure

### 1.1 Field Assistant — Bottom Tab Navigation

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│   [Nearby]   [Visits]   [Sync]   [Help]   [Profile] │
│                                                     │
└─────────────────────────────────────────────────────┘
```

| Tab | Icon | Default Screen | Badge |
|---|---|---|---|
| Nearby | 📍 | Home — GPS Nearby | — |
| Visits | 📋 | Assigned Visits | Overdue count |
| Sync | 🔄 | Sync Status | Pending count |
| Help | 📖 | Help Library | — |
| Profile | 👤 | Profile & Settings | — |

---

### 1.2 Admin — Bottom Tab Navigation (Mobile)

```
┌──────────────────────────────────────────────────────────┐
│                                                          │
│  [Dashboard]  [Approvals]  [Registry]  [Schedule] [More] │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

| Tab | Icon | Default Screen | Badge |
|---|---|---|---|
| Dashboard | 📊 | Admin Dashboard | — |
| Approvals | ✅ | Approval Queue | Pending count |
| Registry | 🗂 | Farm Registry | — |
| Schedule | 📅 | Schedule Overview | Overdue count |
| More | ⋯ | More Menu | — |

**More Menu contains:**
- Agent Overview
- Price Predictions
- Profile & Settings
- Logout

---

### 1.3 Admin — Web Sidebar Navigation (Phase 2)

```
┌─────────────────────────┐
│  🌿 AgriField           │
│─────────────────────────│
│  📊 Dashboard           │
│─────────────────────────│
│  FIELD OPERATIONS       │
│  ├── Approval Queue     │
│  ├── Farm Registry      │
│  ├── Plant Registry     │
│  ├── Schedule Overview  │
│  └── Agent Overview     │
│─────────────────────────│
│  MASTER DATA            │
│  ├── Crop Types         │
│  ├── Visit Templates    │
│  ├── Growth Stages      │
│  ├── Schedule Rules     │
│  ├── Commodities        │
│  ├── Help Content       │
│  └── Regions            │
│─────────────────────────│
│  ORGANISATION           │
│  ├── Farmers            │
│  └── Users              │
│─────────────────────────│
│  INTELLIGENCE           │
│  ├── Price Predictions  │
│  └── Yield Forecasts    │
│─────────────────────────│
│  REPORTS                │
│  ├── Visit Reports      │
│  ├── Prediction History │
│  └── Agent Performance  │
│─────────────────────────│
│  ⚙️  Settings           │
└─────────────────────────┘
```

---

## 2. Screen Hierarchy — Field Assistant

### 2.1 Nearby Tab

```
Home — GPS Nearby
├── Farm Profile
│   ├── Visit History (timeline)
│   │   └── Visit Detail
│   │       └── Visit AI Result
│   ├── Plant List (plant-level crops)
│   │   └── Plant Profile
│   │       ├── Visit History (timeline)
│   │       │   └── Visit Detail
│   │       │       └── Visit AI Result
│   │       └── Start Visit → Visit Execution Flow
│   ├── Season List (field-level crops)
│   │   └── Season Detail
│   └── Start Visit → Visit Execution Flow
│
└── Register New Farm
    ├── Step 1 — Crop Type Select
    ├── Step 2 — GPS Capture & Map Pin
    ├── Step 3 — Farm Details (name, farmer, area)
    ├── Step 4 — Photo (farm overview)
    └── Registration Submitted (pending approval)
```

#### Visit Execution Flow (shared)

```
Visit Execution
├── Step 1..N  (dynamic — rendered from visit template)
│   ├── Photo Capture Step
│   │   ├── Camera View
│   │   ├── Photo Preview & Retake
│   │   └── Confirm Photo
│   ├── Text Input Step
│   ├── Numeric Input Step
│   └── Select Step
├── Visit Summary (review all steps)
└── Submit
    ├── Saved to local DB
    └── Queued for sync
```

#### Register New Plant Flow

```
Register New Plant
├── Step 1 — Select Parent Farm
├── Step 2 — GPS Capture (tap map or auto)
├── Step 3 — Plant Details (planting date, notes)
├── Step 4 — Photo (plant overview)
└── Registration Submitted (pending approval)
```

---

### 2.2 Visits Tab

```
Assigned Visits
├── Filter (All / Pending / Overdue / Completed)
└── Visit Card → Farm Profile → Visit Execution Flow
```

---

### 2.3 Sync Tab

```
Sync Status
├── Pending Visits (count + list)
├── Pending Photos (count + size)
├── Last Sync Timestamp
├── Manual Sync Button
└── Sync Error List
    └── Error Detail
```

---

### 2.4 Help Tab

```
Help Library
├── Filter by Crop Type
└── Help Content Card
    └── Help Content View
        ├── Video Player
        └── PDF / Article Viewer
```

---

### 2.5 Profile Tab

```
Profile & Settings
├── Name, Role, Region (read only)
├── Language Preference (select)
├── App Version
├── About
└── Logout
```

---

## 3. Screen Hierarchy — Admin Mobile

### 3.1 Dashboard Tab

```
Admin Dashboard
├── KPI Cards
│   ├── Visits Today
│   ├── Pending Approvals
│   ├── Overdue Visits
│   └── Latest Price Prediction
└── Activity Feed
    └── Activity Detail → relevant record
```

---

### 3.2 Approvals Tab

```
Approval Queue
├── Filter (All / Farms / Plants / By Region)
└── Approval Card
    └── Approval Detail
        ├── Map — GPS location of registration
        ├── Photos submitted by agent
        ├── Farm / Plant details
        ├── Submitted by (agent info)
        ├── Approve Button
        │   └── Confirmation → Farm/Plant goes live
        └── Reject Button
            └── Rejection Reason Input → Submitted
```

---

### 3.3 Registry Tab

```
Farm Registry
├── Filter (Region / Crop Type / Agent / Status)
├── Search
└── Farm Card
    └── Farm Detail (Admin)
        ├── Farm Info (name, location, crop, area)
        ├── Assigned Agent (reassign)
        ├── Visit History Timeline
        ├── Plant List (plant-level crops)
        │   └── Plant Detail
        │       └── Visit History
        └── Season List (field-level crops)
            └── Season Detail
```

---

### 3.4 Schedule Tab

```
Schedule Overview
├── View Toggle (Calendar / List)
├── Filter (Region / Agent / Crop / Status)
└── Visit Schedule Item
    ├── Farm / Plant info
    ├── Assigned Agent
    ├── Due Date
    ├── Status (Upcoming / Overdue / Completed)
    └── Reassign Agent
```

---

### 3.5 More Menu

```
More
├── Agent Overview
│   └── Agent Card
│       ├── Farms assigned
│       ├── Visits completed (this week / month)
│       ├── Last active
│       └── Visit Activity Log
├── Price Predictions
│   ├── Filter (Commodity / Region / Date Range)
│   ├── Price Trend Chart
│   ├── Confidence Indicator
│   └── Contributing Factors Breakdown
├── Profile & Settings
└── Logout
```

---

## 4. Screen Hierarchy — Admin Web (Phase 2)

### 4.1 Dashboard

```
Dashboard
├── KPI Summary Row
├── Visits Map (geographic heatmap)
├── Recent Activity Feed
├── Overdue Visits Alert Panel
└── Commodity Price Summary Widget
```

---

### 4.2 Master Data

```
Crop Type Manager
├── Crop Type List
└── Crop Type Detail
    ├── Name (multilingual)
    ├── Tracking Level (field / plant)
    ├── Lifespan & Seasonal flag
    ├── Icon / Illustration upload
    └── Linked Visit Templates

Visit Template Builder
├── Template List (by crop type)
└── Template Editor
    ├── Template Name (multilingual)
    ├── Step Builder (drag to reorder)
    │   ├── Add Step
    │   │   ├── Step Type (photo / text / number / select)
    │   │   ├── Label (multilingual)
    │   │   ├── Required toggle
    │   │   └── Options (for select type)
    │   └── Edit / Delete Step
    ├── Version History
    └── Publish / Archive

Growth Stage Manager
├── Stage List (per crop type)
└── Stage Editor
    ├── Name (multilingual)
    ├── Sequence Order
    ├── Typical Duration (days)
    └── Reference Photo upload

Schedule Rule Config
├── Rule List (per crop type)
└── Rule Editor
    ├── Frequency (days)
    ├── Trigger (calendar / growth stage / manual)
    └── Overdue Threshold (days)

Commodity Manager
├── Commodity List
└── Commodity Detail
    ├── Name (multilingual)
    ├── Unit (tonne / kg / bushel)
    ├── Market Code
    └── Linked Crop Types

Help Content Manager
├── Content List (by crop type)
└── Content Editor
    ├── Title (multilingual)
    ├── Type (video / pdf / article)
    ├── File / URL upload
    ├── Target Role
    └── Sort Order

Region Manager
├── Region Tree (hierarchy view)
└── Region Editor
    ├── Name
    ├── Parent Region
    └── Boundary Coordinates

User Manager
├── User Table
└── User Detail
    ├── SSO identity (read only)
    ├── Role assignment
    └── Region assignment
```

---

### 4.3 Intelligence

```
Price Predictions
├── Commodity Selector
├── Region / Date Range Filter
├── Price Trend Line Chart
├── Confidence Band
├── Contributing Factors Panel
│   ├── Growth Stage Distribution
│   ├── Yield Estimates Summary
│   └── Weather Patterns Summary
└── Prediction History Table

Yield Forecasts
├── Crop Type Selector
├── Region Filter
├── Yield Forecast Chart (per farm / aggregate)
└── Comparison vs Previous Season
```

---

### 4.4 Reports

```
Visit Reports
├── Filter (Date Range / Region / Agent / Crop)
├── Summary Table
└── Export (CSV / PDF)

Prediction History
├── Filter (Commodity / Date Range)
├── History Table
└── Export (CSV)

Agent Performance
├── Filter (Date Range / Region)
├── Performance Table
│   ├── Visits completed
│   ├── On-time rate
│   ├── Farms covered
│   └── Data completeness score
└── Export (CSV / PDF)
```

---

## 5. Entry Points & Routing

### App Launch Flow

```
App Launch
    ↓
Check auth token (local)
    ↓
┌─────────────────┬──────────────────┐
│  Token valid    │  No token /      │
│      ↓          │  expired         │
│  Role Router    │      ↓           │
│      ↓          │  SSO Login       │
│                 │      ↓           │
│                 │  Language Select │
│                 │  (first launch)  │
│                 │      ↓           │
│                 │  Role Router     │
└─────────────────┴──────────────────┘
    ↓
┌─────────────────┬──────────────────┐
│ Field Assistant │     Admin        │
│      ↓          │      ↓           │
│  Nearby Tab     │  Dashboard Tab   │
└─────────────────┴──────────────────┘
```

---

### Deep Link Structure

| Deep Link | Destination | Access |
|---|---|---|
| `agrifield://farm/:id` | Farm Profile | All roles |
| `agrifield://plant/:id` | Plant Profile | All roles |
| `agrifield://visit/:id` | Visit Detail | All roles |
| `agrifield://approval/:id` | Approval Detail | Admin only |
| `agrifield://schedule/:id` | Schedule Item | Admin only |
| `agrifield://sync` | Sync Status | Field Assistant |

---

### Push Notification → Screen Mapping

| Notification | Recipient | Navigates To |
|---|---|---|
| New visit assigned | Field Assistant | Assigned Visits |
| Visit overdue | Field Assistant | Assigned Visits (overdue filter) |
| Registration approved | Field Assistant | Farm / Plant Profile |
| Registration rejected | Field Assistant | Registration Submitted (with reason) |
| Approval pending | Admin | Approval Queue |
| Visit overdue | Admin | Schedule Overview (overdue filter) |
| AI result ready | Field Assistant | Visit Result |
| Sync failed | Field Assistant | Sync Status |

---

## 6. Screen Content Specifications

### Home — GPS Nearby

```
┌─────────────────────────────────┐
│  📍 Nearby Farms & Plants        │
│  Searching within 200m...        │
│                                  │
│  ┌───────────────────────────┐  │
│  │ 🌴 LADANG MAJU            │  │
│  │ Oil Palm · 145m away       │  │
│  │ Last visit: 3 weeks ago    │  │
│  │ ⚠️  Visit overdue          │  │
│  └─────────────── [Open →]   ┘  │
│                                  │
│  ┌───────────────────────────┐  │
│  │ 🌾 SAWAH INDAH — Block B  │  │
│  │ Paddy · 380m away          │  │
│  │ Last visit: 2 days ago     │  │
│  └─────────────── [Open →]   ┘  │
│                                  │
│  ─────────────────────────────  │
│  Not seeing your farm?           │
│  [+ Register New Farm / Plant]   │
└─────────────────────────────────┘
```

**Content elements:**
- GPS search radius indicator
- Farm/plant cards sorted by distance
- Distance badge
- Last visit date
- Overdue warning badge
- Crop type icon
- Register CTA at bottom

---

### Farm Profile

**Content elements:**
- Farm name, crop type icon, status badge
- Distance from current GPS (field assistant view)
- Region, area, farmer name, assigned agent
- Small map preview with farm boundary or pin
- Start Visit CTA (if assigned and active)
- Visit history timeline (newest first)
  - Each card: date, agent, AI health score, photo thumbnail
- Plant list (plant-level crops) or Season list (field-level crops)

---

### Visit Execution

**Content elements:**
- Step progress indicator (e.g. Step 2 of 4)
- Farm / Plant name in header
- GPS confirmation badge (green = within boundary)
- Dynamic step content based on template
- Required field indicator
- Back / Next navigation
- Cancel with confirmation dialog

---

### Approval Detail

**Content elements:**
- Registration type badge (New Farm / New Plant)
- Submitted by: agent name, date and time
- Full-screen map with GPS pin
- Photo carousel (all submitted photos)
- Registration details table
- Previous rejection history (if resubmission)
- Approve (green) / Reject (red) action buttons
- Rejection reason text input (shown on reject tap)

---

## 7. Empty States

| Screen | Empty State | CTA |
|---|---|---|
| Home — Nearby | No farms found within 200m | Register New Farm |
| Assigned Visits | No visits assigned yet | — |
| Approval Queue | All caught up | — |
| Farm Registry | No farms registered | Add Farm |
| Visit History | No visits recorded yet | Start First Visit |
| Help Library | No help content for this crop type | — |
| Sync Status | Everything is synced | — |
| Agent Overview | No agents assigned | Go to User Manager |
| Price Predictions | Insufficient data for predictions | — |

---

## 8. Error States

| Scenario | Message | Action |
|---|---|---|
| GPS unavailable | Location unavailable — please enable GPS | Open device settings |
| No internet (sync) | You are offline — data will sync when connected | Manual retry |
| AI analysis failed | Analysis pending — will retry automatically | — |
| Approval failed (server) | Unable to submit — please try again | Retry button |
| Session expired | Your session has expired — please log in again | Login button |
| Farm outside boundary | You don't appear to be at this farm | Override with reason |

---

*AgriField — Commodity Intelligence Platform | Confidential — Internal Use Only*