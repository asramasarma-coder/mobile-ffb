# User Stories & Acceptance Criteria
**Version:** 1.0  
**Status:** Draft  
**Date:** February 2026  

---

## Overview

User stories are written per module and persona. Each story follows the format:

> As a **[persona]**, I want to **[action]** so that **[outcome]**.

Acceptance criteria define the conditions that must be true for a story to be considered done.

**Story Status:** `Draft` | `Ready` | `In Progress` | `Done`  
**Priority:** `Must Have` | `Should Have` | `Could Have`

---

## Module 1 — Identity & Access

---

### US-001 — SSO Login
**As a** field assistant or admin,  
**I want to** log in using my company SSO credentials,  
**so that** I don't need a separate password and my access is centrally managed.

**Priority:** Must Have | **Status:** Draft

**Acceptance Criteria:**
- [ ] App presents SSO login button on launch if no valid session exists
- [ ] Tapping SSO login opens the company identity provider (Microsoft Entra / Google Workspace / Okta) in a browser
- [ ] After successful SSO, app receives Firebase JWT and stores it in SecureStore
- [ ] If user role is pending, app shows "Your account is pending activation" message
- [ ] If user role is active, app routes to the correct home screen for the role
- [ ] Login state persists across app restarts (token valid)
- [ ] Expired tokens trigger re-authentication without losing unsaved data

---

### US-002 — Language Selection
**As a** field assistant,  
**I want to** select my preferred language on first launch,  
**so that** the app is usable in my native language.

**Priority:** Must Have | **Status:** Draft

**Acceptance Criteria:**
- [ ] Language selection screen shown after first successful login
- [ ] Supported languages displayed (English, Bahasa Malaysia)
- [ ] Selected language applied immediately to all UI text
- [ ] Language preference saved to user profile on server
- [ ] Language can be changed anytime in Profile & Settings
- [ ] Language persists across app restarts
- [ ] If device locale matches a supported language, it is pre-selected

---

### US-003 — Role-Based Routing
**As the** system,  
**I want to** route users to the correct home screen based on their role,  
**so that** field assistants see the field experience and admins see the admin experience.

**Priority:** Must Have | **Status:** Draft

**Acceptance Criteria:**
- [ ] Field assistants land on the Nearby (GPS) tab after login
- [ ] Admins land on the Dashboard tab after login
- [ ] Field assistants cannot navigate to any admin screen
- [ ] Admins can access both admin screens and field assistant screens
- [ ] Role determined from server response — not stored locally in a modifiable way

---

## Module 4 — Field Operations

---

### US-004 — GPS Nearby Discovery
**As a** field assistant,  
**I want to** see nearby farms and plants automatically based on my GPS location,  
**so that** I can identify my visit target without manually searching.

**Priority:** Must Have | **Status:** Draft

**Acceptance Criteria:**
- [ ] App requests location permission on first use with clear explanation
- [ ] Nearby farms and plants within 200m displayed automatically on Home screen
- [ ] Results sorted by distance (closest first)
- [ ] Each result shows: farm/plant name, crop type icon, distance, last visit date, overdue badge if applicable
- [ ] GPS updates every 10 seconds while on Nearby screen
- [ ] If no farms found within 200m, empty state shown with "Register New Farm" CTA
- [ ] If GPS unavailable, error banner shown with link to device settings
- [ ] Results cached locally — available if connectivity lost while viewing
- [ ] Works fully offline using cached farm locations

---

### US-005 — View Farm Profile
**As a** field assistant,  
**I want to** view a farm's profile and visit history before starting a visit,  
**so that** I can confirm I'm at the right farm and understand its history.

**Priority:** Must Have | **Status:** Draft

**Acceptance Criteria:**
- [ ] Farm profile shows: name, crop type, region, area, farmer name, assigned agent
- [ ] Small map view shows farm GPS location or boundary polygon
- [ ] Visit history shown as timeline (newest first) with date, agent, AI health score, photo thumbnail
- [ ] Start Visit button visible if farm is assigned to this agent and status is active
- [ ] For plant-level crops: plant list shown below visit history
- [ ] For field-level crops: season list shown below visit history
- [ ] Profile accessible offline using cached data
- [ ] "Last updated" timestamp shown when viewing cached data offline

---

### US-006 — Execute a Visit
**As a** field assistant,  
**I want to** complete a structured visit using guided steps,  
**so that** I collect all required data consistently for every visit.

**Priority:** Must Have | **Status:** Draft

**Acceptance Criteria:**
- [ ] GPS confirmation shown before visit steps begin (green badge = within farm boundary)
- [ ] Steps rendered dynamically from the active visit template for that crop type
- [ ] Each step shows: label (in user's language), input type, required indicator
- [ ] Required steps cannot be skipped — Next button disabled until input provided
- [ ] Optional steps can be skipped
- [ ] Photo steps launch the device camera
- [ ] Step progress indicator shows current step number and total
- [ ] Agent can navigate back to previous steps to change responses
- [ ] Visit summary shown before final submission
- [ ] Visit saved to local DB immediately on submit — not dependent on connectivity
- [ ] Confirmation shown: "Visit saved. Will upload when connected."

---

### US-007 — Guided Photo Capture
**As a** field assistant,  
**I want to** receive guidance on what photos to take for each step,  
**so that** I capture the right photos for the AI analysis.

**Priority:** Must Have | **Status:** Draft

**Acceptance Criteria:**
- [ ] Photo step shows the step label as an overlay on the camera view
- [ ] Agent can take photo or pick from camera roll
- [ ] Preview shown after capture with "Retake" and "Use Photo" options
- [ ] Photo compressed to max 1280px on longest edge before local storage
- [ ] Multiple photos can be taken per step (up to template maximum)
- [ ] Photo thumbnails shown in visit summary screen
- [ ] Blurry or very dark photos surface a warning (optional confirmation to proceed)

---

### US-008 — Register a New Farm
**As a** field assistant,  
**I want to** register a new farm I discover in the field,  
**so that** it can be added to the platform and monitored.

**Priority:** Must Have | **Status:** Draft

**Acceptance Criteria:**
- [ ] Registration flow accessible from Nearby screen (empty state) and from a button in the app
- [ ] GPS location auto-captured at start of registration
- [ ] Agent selects crop type from list of active crop types
- [ ] Required fields: name, crop type, GPS (auto), area (optional), farmer (select or create)
- [ ] At least one photo required (farm overview)
- [ ] Registration saved locally and submitted to server when online
- [ ] Server creates farm with status `pending_approval`
- [ ] Agent sees confirmation: "Registration submitted — pending admin approval"
- [ ] Pending registration visible in agent's registration history

---

### US-009 — Register a New Plant
**As a** field assistant,  
**I want to** register a new plant I discover on a farm,  
**so that** it can be tracked individually going forward.

**Priority:** Must Have | **Status:** Draft

**Acceptance Criteria:**
- [ ] Registration accessible from farm profile (plant-level farms only)
- [ ] GPS captured — agent can also tap on a map to pin exact location
- [ ] Required: parent farm (auto-filled if launched from farm profile), GPS coordinates
- [ ] Optional: planting date, notes
- [ ] At least one photo required
- [ ] Submitted for admin approval
- [ ] Plant code auto-assigned by server after approval

---

## Module 5 — Approval Workflow

---

### US-010 — View Approval Queue
**As an** admin,  
**I want to** see all pending farm and plant registrations in one place,  
**so that** I can process them promptly.

**Priority:** Must Have | **Status:** Draft

**Acceptance Criteria:**
- [ ] Approval Queue tab shows count badge for pending items
- [ ] List shows: type (farm/plant), submitted by, submission date, crop type, region
- [ ] Filterable by type, region
- [ ] Sorted by submission date (oldest first — FIFO processing)
- [ ] Tapping item opens Approval Detail screen

---

### US-011 — Approve a Registration
**As an** admin,  
**I want to** approve a farm or plant registration,  
**so that** it becomes active and the field assistant can start collecting visits.

**Priority:** Must Have | **Status:** Draft

**Acceptance Criteria:**
- [ ] Approval Detail shows GPS pin on map, submitted photos, all registration details
- [ ] "Approve" button visible and accessible
- [ ] Tapping Approve shows confirmation dialog
- [ ] After confirmation: farm/plant status changes to `active`
- [ ] Field assistant receives push notification: "Your registration was approved"
- [ ] Farm/plant appears in field assistant's assigned list on next sync
- [ ] Approval action logged with admin ID and timestamp

---

### US-012 — Reject a Registration
**As an** admin,  
**I want to** reject a registration with a reason,  
**so that** the field assistant knows what to fix and can resubmit.

**Priority:** Must Have | **Status:** Draft

**Acceptance Criteria:**
- [ ] "Reject" button visible on Approval Detail screen
- [ ] Rejection requires a reason (text input — cannot be empty)
- [ ] After rejection: farm/plant status changes to `rejected`
- [ ] Field assistant receives push notification: "Your registration was rejected" with reason
- [ ] Rejection reason visible to field assistant in their registration history
- [ ] Field assistant can resubmit a corrected registration

---

## Module 6 — Scheduling & Alerts

---

### US-013 — View Assigned Visit Schedule
**As a** field assistant,  
**I want to** see my scheduled upcoming and overdue visits,  
**so that** I know which farms I need to visit and when.

**Priority:** Must Have | **Status:** Draft

**Acceptance Criteria:**
- [ ] Assigned Visits tab shows list of visits: upcoming and overdue
- [ ] Each item shows: farm/plant name, crop type, due date, status badge
- [ ] Overdue items highlighted in red at top of list
- [ ] Completed visits visible with green status badge
- [ ] Tapping item opens farm/plant profile
- [ ] List available offline using cached data
- [ ] Badge count on Visits tab shows overdue count

---

### US-014 — Admin Assigns Visit to Agent
**As an** admin,  
**I want to** assign a visit to a specific field assistant,  
**so that** they know they are responsible for visiting that farm.

**Priority:** Must Have | **Status:** Draft

**Acceptance Criteria:**
- [ ] Admin can select farm/plant and assign to a field assistant with a due date
- [ ] Scheduled visit created and visible on admin Schedule Overview
- [ ] Field assistant receives push notification: "New visit assigned: [farm name] due [date]"
- [ ] Visit appears in field assistant's Assigned Visits tab on next sync
- [ ] Admin can reassign to a different agent if needed

---

## Module 7 — Sync & Offline

---

### US-015 — View Sync Status
**As a** field assistant,  
**I want to** see what data is pending upload and whether sync is working,  
**so that** I have confidence my data will reach the server.

**Priority:** Must Have | **Status:** Draft

**Acceptance Criteria:**
- [ ] Sync tab shows: pending visits count, pending photos (count + size), pending registrations
- [ ] Last successful sync timestamp shown
- [ ] If items are pending: list of pending items shown
- [ ] If sync errors exist: error list with "Retry" button per error
- [ ] Manual "Sync Now" button triggers immediate sync attempt
- [ ] Offline banner shown persistently when no connectivity detected
- [ ] Badge count on Sync tab shows total pending items

---

### US-016 — Automatic Background Sync
**As a** field assistant,  
**I want** my data to sync automatically when connectivity is restored,  
**so that** I don't have to manually trigger syncs.

**Priority:** Must Have | **Status:** Draft

**Acceptance Criteria:**
- [ ] Sync triggered automatically when network connectivity is detected
- [ ] Sync runs in background — user can continue using app during sync
- [ ] Sync runs on app foreground if last sync > 15 minutes ago
- [ ] Photos uploaded separately from visit metadata (independent retry)
- [ ] Failed items retried with exponential backoff
- [ ] User notified of sync completion via in-app banner (auto-dismiss)
- [ ] Sync errors persisted until resolved — not lost on app restart

---

## Module 8 — AI & Analysis

---

### US-017 — View AI Analysis Results
**As a** field assistant,  
**I want to** see the AI analysis results for my submitted visits,  
**so that** I can understand the health and condition of the crops I visited.

**Priority:** Must Have | **Status:** Draft

**Acceptance Criteria:**
- [ ] Visit Result screen accessible from visit history and via push notification
- [ ] Results show: crop health score (0-100), growth stage, predicted yield, anomalies
- [ ] Anomalies displayed with type and description (e.g. "Possible bagworm signs")
- [ ] Health score visualised (e.g. colour-coded gauge)
- [ ] Results available once AI analysis is complete (not instant — shows "Analysis pending" until ready)
- [ ] Push notification sent when results ready
- [ ] If AI analysis fails after retries: "Analysis unavailable for this visit" message shown

---

## Module 9 — Commodity & Predictions

---

### US-018 — View Commodity Price Predictions
**As an** admin,  
**I want to** view commodity price predictions for the crops in my region,  
**so that** I can advise on pricing and procurement decisions.

**Priority:** Must Have | **Status:** Draft

**Acceptance Criteria:**
- [ ] Price Predictions screen accessible from admin More menu
- [ ] Filter by commodity type and region
- [ ] Line chart showing price trend over time with confidence band
- [ ] Latest predicted price prominently displayed with confidence score
- [ ] Contributing factors panel: farms sampled, avg health score, growth stage distribution
- [ ] Date range filter (last 30 days, 90 days, 12 months)
- [ ] If insufficient data: "Insufficient visit data for prediction" message

---

## Module 10 — Help & Training

---

### US-019 — Access Help Content
**As a** field assistant,  
**I want to** access training videos and guides relevant to the crops I visit,  
**so that** I can improve the quality of my data collection.

**Priority:** Should Have | **Status:** Draft

**Acceptance Criteria:**
- [ ] Help Library accessible from Help tab
- [ ] Content filtered by crop types assigned to the agent
- [ ] Each content item shows: title, type (video/PDF/article), crop type
- [ ] Video content plays in-app video player
- [ ] PDF content opens in-app PDF viewer
- [ ] Content accessible offline if previously viewed (cached)
- [ ] Uncached content shows "Available when online" if offline

---

## Story Summary by Phase

### Phase 1 — Must Have

| Story | Title | Persona |
|---|---|---|
| US-001 | SSO Login | All |
| US-002 | Language Selection | Field Assistant |
| US-003 | Role-Based Routing | System |
| US-004 | GPS Nearby Discovery | Field Assistant |
| US-005 | View Farm Profile | Field Assistant |
| US-006 | Execute a Visit | Field Assistant |
| US-007 | Guided Photo Capture | Field Assistant |
| US-008 | Register a New Farm | Field Assistant |
| US-009 | Register a New Plant | Field Assistant |
| US-010 | View Approval Queue | Admin |
| US-011 | Approve a Registration | Admin |
| US-012 | Reject a Registration | Admin |
| US-013 | View Assigned Visit Schedule | Field Assistant |
| US-014 | Admin Assigns Visit | Admin |
| US-015 | View Sync Status | Field Assistant |
| US-016 | Automatic Background Sync | Field Assistant |
| US-017 | View AI Analysis Results | Field Assistant |
| US-018 | View Commodity Price Predictions | Admin |

### Phase 2 — Should Have

| Story | Title | Persona |
|---|---|---|
| US-019 | Access Help Content | Field Assistant |
| US-020 | Configure Crop Types | Super Admin |
| US-021 | Build Visit Template | Admin |
| US-022 | Manage Schedule Rules | Admin |
| US-023 | Export Visit Reports | Admin |
| US-024 | View Agent Performance | Admin |

---

## Definition of Done

A user story is **Done** when all of the following are true:

- [ ] All acceptance criteria pass
- [ ] Code reviewed and merged to main
- [ ] Unit tests written and passing
- [ ] Tested on both iOS and Android
- [ ] Tested in offline mode (where applicable)
- [ ] All UI strings translated to all active languages
- [ ] QA sign-off received
- [ ] No critical or high severity bugs open

---

*AgriField — Commodity Intelligence Platform | Confidential — Internal Use Only*