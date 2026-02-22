# AgriField — Commodity Intelligence Platform
## Product Requirements Document
**Version:** 1.0 — Draft  
**Status:** In Review  
**Date:** February 2026  
**Platform:** React Native (iOS & Android) + GCP Serverless  

---

## Document Information

| Field | Detail |
|---|---|
| Document Title | AgriField — Commodity Intelligence Platform PRD |
| Version | 1.0 |
| Status | Draft — Pending Stakeholder Review |
| Date | February 2026 |
| Author | Product Team |
| Reviewers | Engineering Lead, Admin Manager, Field Operations |
| Next Review | March 2026 |

### Revision History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | Feb 2026 | Product Team | Initial draft based on product discovery sessions |

---

## 1. Executive Summary

AgriField is a mobile-first, serverless commodity intelligence platform designed for agricultural field operations. The platform enables field assistants to collect farm and crop data using AI-driven photo analysis, while providing administrators with tools to manage field operations, approve registrations, and view commodity price predictions.

The platform is built on two core principles:

- **Generic and configurable** — any crop type can be added and configured by an administrator without code changes
- **Field-first design** — the mobile experience is optimised for rural field conditions, with offline capability and GPS-driven workflows

The system serves two primary user personas — Field Assistants who collect data in the field, and Administrators who oversee operations, manage master data, and interpret predictions. The AI analysis engine is a pre-existing service that the platform integrates with via API.

---

## 2. Problem Statement

### 2.1 Background

Accurate commodity price prediction for agricultural products requires timely, reliable data from farms and fields. Currently, field data collection is manual, inconsistent, and slow — relying on paper forms, phone calls, or ad-hoc spreadsheets. This results in:

- Delayed and inaccurate commodity price forecasts
- Inability to track crop progression over time at field or plant level
- No standardised data format across different crop types and regions
- Field assistants working without context — no history of previous visits when they arrive at a farm
- Administrators with no real-time visibility into field operations

### 2.2 Opportunity

A structured, AI-powered field data collection platform can transform this process. By equipping field assistants with a GPS-driven mobile app and automating crop analysis through photo AI, the platform can:

- Dramatically reduce manual data entry and human error
- Enable consistent, structured data collection across all crop types
- Provide administrators with real-time operational visibility
- Feed a commodity price prediction engine with reliable, timely field data

---

## 3. Goals & Success Metrics

### 3.1 Product Goals

- Enable field assistants to complete a farm visit in under 3 minutes using the mobile app
- Provide GPS-driven farm discovery so agents never need to manually search for a farm
- Automate crop growth stage, yield estimation, and weather data via AI photo analysis
- Support configurable visit templates per crop type without code changes
- Maintain full visit history per farm and per plant accessible to both agents and admins
- Operate reliably offline with automatic sync when connectivity is restored
- Support multi-language UI from day one

### 3.2 Success Metrics

| Metric | Target | Measurement |
|---|---|---|
| Average visit completion time | < 3 minutes | App analytics — visit start to submit |
| Offline sync success rate | > 99% | Sync engine error logs |
| Visit data completeness | > 95% | % of visits with all required steps completed |
| Admin approval turnaround | < 24 hours | Time from submission to approval decision |
| Field assistant adoption | > 90% weekly active | Login and visit submission rates |
| AI analysis return rate | > 98% | % of visits receiving AI results within 1 hour |

---

## 4. User Personas

### 4.1 Field Assistant 🧑‍🌾

A field agent who regularly visits farms and plantations to collect crop data. Often working in rural areas with limited connectivity. Non-technical user who needs a fast, intuitive mobile experience. Primarily uses the app on Android devices.

**Goals**
- Complete farm visits quickly in the field
- Access previous visit history before starting a new visit
- Register new farms or plants discovered during visits
- Work offline without losing data
- Receive clear guidance on what photos to take

**Pain Points**
- Poor mobile connectivity in rural areas
- Multiple similar-looking plants with no easy way to differentiate
- No context about a farm when arriving for the first time
- Inconsistent data collection leading to rejected submissions
- Language barriers with English-only interfaces

---

### 4.2 Administrator 👨‍💼

An operations manager responsible for overseeing field activities, managing farm registrations, assigning visits, and interpreting commodity price predictions. Works from office primarily but needs mobile access for approvals and oversight.

**Goals**
- Real-time visibility into field operations
- Approve new farm and plant registrations quickly
- Assign and schedule visits to field assistants
- Configure visit templates per crop type
- View commodity price predictions and yield forecasts

**Pain Points**
- No visibility into what field assistants are actually doing
- Manual, slow approval process for new registrations
- Difficult to spot overdue visits across a large portfolio
- No standardised way to configure data collection per crop
- Price predictions based on incomplete or unreliable field data

---

## 5. Scope

### 5.1 In Scope — Phase 1 (Mobile App)

- Company SSO login with role-based routing
- GPS-driven nearby farm and plant discovery
- Farm and plant profile with full visit history
- Dynamic visit execution based on crop-configured templates
- Guided photo capture per visit step
- Automatic submission to AI analysis API
- New farm and plant registration with admin approval workflow
- Offline-first data storage with automatic background sync
- Multi-language UI support (i18n from day one)
- Admin mobile screens: dashboard, approval queue, schedule overview
- Help content viewer per crop type
- Push notifications for assignments, approvals, and overdue alerts

### 5.2 In Scope — Phase 2 (Web Admin Dashboard)

- Full crop type configuration and visit template builder
- Growth stage and schedule rule management
- Commodity and price prediction management
- Farm and plant registry with advanced filtering
- Agent performance reporting and exports
- Help content upload and management
- Region and user management

### 5.3 Out of Scope

- Financial transaction processing or payments
- Direct farmer-facing application (farmers do not use this app)
- Building or training the AI/ML analysis model
- Real-time satellite imagery integration (deferred to future phase)
- Third-party market price feed integration (deferred to Phase 3)
- Web browser version of the field assistant app

---

## 6. Platform Modules

| # | Module | Description | Persona |
|---|---|---|---|
| 1 | Identity & Access | SSO login, role-based routing, language preference, user profile | All users |
| 2 | Master Data Management | Crop types, visit templates, growth stages, schedule rules, commodities, help content, regions | Admin / Super Admin |
| 3 | Organisation Management | Farmers, farms, plants, seasons, region hierarchy, user assignment | Admin |
| 4 | Field Operations | GPS discovery, farm/plant profiles, visit execution, registration | Field Assistant |
| 5 | Approval Workflow | New farm/plant registrations submitted by field assistants, reviewed by admin | Both |
| 6 | Scheduling & Alerts | Visit schedule rules engine — generates upcoming visits, flags overdue, sends notifications | Admin |
| 7 | Sync & Offline | Local storage, upload queue, conflict resolution, sync status | Field Assistant |
| 8 | AI & Analysis | Submits visit data and photos to AI API, receives and stores analysis results | System |
| 9 | Commodity & Predictions | Price prediction display, yield forecasts, trend charts fed by AI analysis results | Admin |
| 10 | Help & Training | Help videos, guides, and reference content per crop type for field assistants | Field Assistant |
| 11 | Reporting & Exports | Visit summaries, farm activity logs, prediction history, agent performance reports | Admin |

---

## 7. Key Feature Requirements

### 7.1 GPS-Driven Farm Discovery

The GPS nearby screen is the entry point for every field assistant session. The app continuously monitors device location and surfaces nearby farms and plants within a configurable radius (default 200m).

- Display nearby farms and plants sorted by distance
- Show last visit date, crop type, and overdue status per item
- Confirm agent is within farm boundary before allowing visit start
- If no nearby farms found, prompt to register a new farm
- Cache nearby results locally for offline access

### 7.2 Configurable Visit Templates

Visit templates define what data a field assistant collects during a visit. Templates are configured per crop type by administrators and rendered dynamically on the mobile app.

- Templates support step types: `photo`, `text`, `number`, `select`
- Each step can be marked required or optional
- Step labels support multiple languages
- Templates are versioned — existing visits retain the template version used
- Field assistants see only the steps relevant to their current crop type

### 7.3 Dual Tracking Modes

The platform supports two fundamentally different crop tracking approaches:

| Mode | Crops | Tracking Unit | History |
|---|---|---|---|
| Field-level | Paddy, Corn, Wheat | Field polygon | Per season / per visit |
| Plant-level | Oil Palm, Rubber, Durian | Individual plant (GPS marker) | Per plant, per visit |

Tracking level is configured per crop type by admin and cannot be changed after farms are registered.

### 7.4 AI Photo Analysis Integration

After a visit is submitted, photos and GPS data are sent to the pre-existing AI analysis API.

```
Mobile App
    ↓
Request signed URL → upload photo directly to Cloud Storage
    ↓
Notify API of upload completion
    ↓
Cloud Tasks → ai-dispatcher service
    ↓
AI API called with: photos + GPS + crop type + timestamp
    ↓
Response: crop health score, predicted yield, growth stage, anomalies
    ↓
Results stored against visit record
```

- Analysis failures are retried up to 3 times before flagging for manual review
- AI results are visible in farm/plant visit history

### 7.5 Offline-First Operation

- All visit data written to local WatermelonDB (SQLite) first
- Photos compressed and queued locally before upload
- Background sync triggered automatically when network detected
- Sync status visible to agent — shows items pending upload
- Conflict resolution: server is authoritative for master data, local is authoritative for new visits

### 7.6 Multi-Language Support

- All UI strings managed through i18next translation files
- Admin-configured content (visit template steps, crop type names) supports per-language labels stored as JSON maps
- Language selection persists across sessions
- New languages can be added without app code changes — translation files only

---

## 8. Visit Flow

### Field Assistant Visit Flow

```
Open App → GPS scanning
    ↓
Home — Nearby Screen
    ↓
┌─────────────────────────────┬──────────────────────────┐
│  Existing Farm / Plant      │  New Location             │
│         ↓                  │       ↓                   │
│  Farm / Plant Profile       │  Register Farm / Plant    │
│  (review history)           │       ↓                   │
│         ↓                  │  Submitted for approval    │
│  Start Visit                │                           │
│         ↓                  │                           │
│  Visit Execution            │                           │
│  (dynamic steps per crop)   │                           │
│         ↓                  │                           │
│  Photo Capture              │                           │
│         ↓                  │                           │
│  Review & Submit            │                           │
│         ↓                  │                           │
│  Saved locally → sync queue │                           │
│         ↓                  │                           │
│  Background sync → AI       │                           │
│         ↓                  │                           │
│  Visit Result (AI results)  │                           │
└─────────────────────────────┴──────────────────────────┘
```

### Admin Approval Flow

```
Field Assistant submits registration
    ↓
Admin gets push notification
    ↓
Approval Queue → Approval Detail
(reviews GPS location, photos, crop type, details)
    ↓
┌───────────────┬──────────────────┐
│   Approve     │     Reject       │
│      ↓        │        ↓         │
│  Farm/Plant   │  Agent notified  │
│  goes live    │  with reason     │
└───────────────┴──────────────────┘
```

---

## 9. Screen Inventory

### Field Assistant Screens

| Screen | Module | Description |
|---|---|---|
| SSO Login | 1 | Company SSO redirect, role detection |
| Language Select | 1 | First login language selection |
| Home — Nearby | 4 | GPS-driven list of nearby farms/plants |
| Farm Profile | 4 | Farm details, visit history timeline, start visit CTA |
| Plant Profile | 4 | Plant ID, history, photos, yield timeline |
| Season Profile | 4 | Active season details for field crops |
| Visit Execution | 4 | Dynamic step-by-step data collection |
| Photo Capture | 4 | Guided photo taking per visit step |
| Visit Summary | 4 | Review before submit |
| Visit Result | 8 | AI analysis result after sync |
| Register Farm | 5 | New farm registration — GPS, crop type, details |
| Register Plant | 5 | New plant registration — GPS pin, farm link |
| Registration Submitted | 5 | Confirmation + pending approval status |
| Sync Status | 7 | Queue status, last sync, manual sync trigger |
| Help Library | 10 | List of help content for relevant crop types |
| Help Content View | 10 | Video or document viewer |
| Assigned Visits | 6 | Scheduled/overdue visits assigned by admin |
| Profile & Settings | 1 | Language toggle, app version, logout |

### Admin Mobile Screens

| Screen | Module | Description |
|---|---|---|
| Admin Dashboard | 9 | KPIs, activity feed, alerts |
| Approval Queue | 5 | List of pending farm/plant registrations |
| Approval Detail | 5 | Review GPS, photos, details — approve/reject |
| Farm Registry | 3 | Browse all farms, filter by region/crop/agent |
| Farm Detail | 3 | Full farm record, visit history, assign agent |
| Plant Registry | 3 | Browse all plants for a farm |
| Agent Overview | 3 | Agent activity, visits completed, overdue |
| Schedule Overview | 6 | Upcoming and overdue visits across all farms |
| Price Predictions | 9 | Commodity price trends and forecast |

### Admin Web Screens — Phase 2

| Screen | Module | Description |
|---|---|---|
| Crop Type Manager | 2 | Create/edit crop types, tracking level, lifespan |
| Visit Template Builder | 2 | Add/edit/reorder steps, multilingual labels |
| Growth Stage Manager | 2 | Define stages per crop type |
| Schedule Rule Config | 2 | Set frequency, triggers, overdue threshold |
| Commodity Manager | 2 | Link crop types to market commodities |
| Help Content Manager | 2 | Upload videos/docs per crop type |
| Region Manager | 2 | Geographic hierarchy management |
| User Manager | 2 | Activate SSO users, assign roles and regions |
| Farmer Registry | 3 | Full farmer management |
| Reports — Visits | 11 | Visit activity export |
| Reports — Predictions | 11 | Price prediction history |
| Reports — Agent Performance | 11 | Field assistant activity analytics |

---

## 10. Build Phases

| Phase | Name | Scope | Priority |
|---|---|---|---|
| 1 | Core Field Operations | SSO login, GPS discovery, farm/plant profiles, visit execution, photo capture, offline sync, new registration, admin approval queue, basic admin dashboard | Must Have |
| 2 | Admin Mobile | Full admin mobile screens — approvals, farm registry, agent overview, schedule, price predictions | Must Have |
| 3 | AI & Predictions | AI API integration, analysis results display, commodity price screen, yield trend charts | Must Have |
| 4 | Web Admin Dashboard | Full crop type config, visit template builder, growth stage manager, schedule rules, reports, region/user management | Should Have |
| 5 | Polish & Scale | Multi-language content management, advanced notifications, performance optimisation, analytics | Could Have |

---

## 11. Technical Stack

| Layer | Technology | Purpose |
|---|---|---|
| Mobile | React Native + Expo | iOS & Android cross-platform |
| Local Storage | WatermelonDB (SQLite) | Offline-first data persistence |
| State Management | Zustand | Lightweight global state |
| Camera | react-native-vision-camera | Guided photo capture |
| Maps | react-native-maps | GPS tagging, farm boundaries |
| Internationalisation | i18next | Multi-language support |
| Authentication | Firebase Auth + SSO (OIDC) | Company SSO bridge, JWT tokens |
| Backend Services | Cloud Run (Node.js) | Serverless API — scales to zero |
| Database | Cloud SQL — PostgreSQL | Server-side relational data |
| File Storage | Cloud Storage (GCP) | Photos, help videos, exports |
| Async Queue | Cloud Tasks | AI dispatch, upload retry logic |
| Events | Pub/Sub | Event-driven processing |
| Push Notifications | Firebase Cloud Messaging | iOS & Android notifications |
| Scheduling | Cloud Scheduler | Cron jobs — visit generation |
| Caching | Memorystore (Redis) | Master data, session cache |
| CI/CD | Cloud Build + Artifact Registry | Automated build and deploy |
| Monitoring | Cloud Logging + Cloud Monitoring | Observability |

### GCP Environment Strategy

| Environment | GCP Project | Purpose |
|---|---|---|
| Development | `agrifield-dev` | Feature development and testing |
| Staging | `agrifield-staging` | UAT and pre-production validation |
| Production | `agrifield-prod` | Live production environment |

### Cloud Run Services

| Service | Responsibility |
|---|---|
| `api-core` | Main REST API — farms, visits, users, master data |
| `api-sync` | Handles bulk offline sync payloads from mobile |
| `ai-dispatcher` | Receives visit data, calls AI API, stores results |
| `scheduler` | Generates upcoming visits from schedule rules |
| `notification-service` | Sends push notifications via Firebase |
| `admin-api` | Admin-specific operations — approvals, reports |
| `export-service` | Generates CSV/PDF reports on demand |

---

## 12. Assumptions & Constraints

### Assumptions

- The AI analysis API is already built and accessible via HTTPS — the platform is a consumer, not a builder
- The company has an existing SSO provider (Microsoft Entra, Google Workspace, or Okta) compatible with OIDC
- Field assistants carry Android or iOS smartphones with camera and GPS capability
- Farm and plant GPS coordinates are accurate enough to support 200m proximity detection
- At least 2 languages are required at launch — additional languages can be added post-launch
- Admin users have reliable internet connectivity and will primarily use web dashboard in Phase 2

### Constraints

- Mobile app must function fully offline — no feature should be blocked by lack of connectivity
- All backend infrastructure must be deployed on GCP using serverless services only
- The platform must not store sensitive personal information beyond what is necessary for operations
- Photo uploads must be compressed to reduce data usage on limited mobile data plans
- The AI API rate limits must be respected — Cloud Tasks queue manages dispatch pacing

---

## 13. Open Questions

| # | Question | Owner | Status |
|---|---|---|---|
| 1 | Which specific crop types must be supported at launch? | Product Owner | Open |
| 2 | Which languages are required at launch and who provides translations? | Product Owner | Open |
| 3 | What is the company SSO provider — Microsoft Entra, Google Workspace, Okta, or other? | Engineering | Open |
| 4 | Can field assistants collect data on a newly registered farm before admin approval (provisional mode)? | Product Owner | Open |
| 5 | What is the maximum number of plants per farm? Affects map rendering performance. | Engineering | Open |
| 6 | What are the AI API rate limits, SLA, and authentication mechanism? | Engineering | Open |
| 7 | Are there data residency or privacy regulations affecting GCP region selection? | Legal / Engineering | Open |

---

## 14. Approval

| Name | Role | Signature | Date |
|---|---|---|---|
| | Product Owner | | |
| | Engineering Lead | | |
| | Field Operations Manager | | |
| | Admin Manager | | |

---

*AgriField — Commodity Intelligence Platform | Confidential — Internal Use Only*
