# Roles & Permissions Matrix
**Version:** 1.0  
**Status:** Draft  
**Date:** February 2026  

---

## Overview

The platform has three roles. Role is assigned at login via SSO and cannot be self-modified.

| Role | Description | Primary Interface |
|---|---|---|
| **Super Admin** | Full platform access — manages system configuration, master data, and all users | Web + Mobile |
| **Admin** | Manages field operations — approvals, scheduling, farm registry, predictions | Web + Mobile |
| **Field Assistant** | Collects field data — visits, registrations, photo capture | Mobile only |

### Permission Key

| Symbol | Meaning |
|---|---|
| ✅ | Full access |
| 👁 | View only |
| ✏️ | Create / Edit own records only |
| 🔒 | No access |
| ⏳ | Pending approval required |

---

## 1. Identity & Access

| Action | Super Admin | Admin | Field Assistant |
|---|---|---|---|
| Login via SSO | ✅ | ✅ | ✅ |
| View own profile | ✅ | ✅ | ✅ |
| Change language preference | ✅ | ✅ | ✅ |
| View all users | ✅ | 👁 | 🔒 |
| Create / invite users | ✅ | 🔒 | 🔒 |
| Assign roles to users | ✅ | 🔒 | 🔒 |
| Activate / deactivate users | ✅ | 🔒 | 🔒 |
| Assign users to regions | ✅ | ✅ | 🔒 |
| View user activity log | ✅ | 👁 | 🔒 |

---

## 2. Master Data Management

| Action | Super Admin | Admin | Field Assistant |
|---|---|---|---|
| **Crop Types** | | | |
| View crop types | ✅ | ✅ | 👁 |
| Create crop type | ✅ | 🔒 | 🔒 |
| Edit crop type | ✅ | 🔒 | 🔒 |
| Activate / deactivate crop type | ✅ | 🔒 | 🔒 |
| **Visit Templates** | | | |
| View visit templates | ✅ | ✅ | 👁 |
| Create visit template | ✅ | ✅ | 🔒 |
| Edit visit template | ✅ | ✅ | 🔒 |
| Publish / archive template | ✅ | ✅ | 🔒 |
| **Growth Stages** | | | |
| View growth stages | ✅ | ✅ | 👁 |
| Create / edit growth stages | ✅ | 🔒 | 🔒 |
| **Schedule Rules** | | | |
| View schedule rules | ✅ | ✅ | 🔒 |
| Create / edit schedule rules | ✅ | ✅ | 🔒 |
| **Commodities** | | | |
| View commodities | ✅ | ✅ | 🔒 |
| Create / edit commodities | ✅ | 🔒 | 🔒 |
| **Help Content** | | | |
| View help content | ✅ | ✅ | ✅ |
| Upload / edit help content | ✅ | ✅ | 🔒 |
| Delete help content | ✅ | ✅ | 🔒 |
| **Regions** | | | |
| View regions | ✅ | ✅ | 👁 |
| Create / edit regions | ✅ | 🔒 | 🔒 |

---

## 3. Organisation Management

| Action | Super Admin | Admin | Field Assistant |
|---|---|---|---|
| **Farmers** | | | |
| View all farmers | ✅ | ✅ | 👁 |
| Create farmer record | ✅ | ✅ | ✏️ |
| Edit farmer record | ✅ | ✅ | 🔒 |
| Delete farmer record | ✅ | 🔒 | 🔒 |
| **Farms** | | | |
| View all farms | ✅ | ✅ | 👁 assigned only |
| Create farm (via registration) | ✅ | ✅ | ⏳ pending approval |
| Edit farm details | ✅ | ✅ | 🔒 |
| Assign field assistant to farm | ✅ | ✅ | 🔒 |
| Deactivate farm | ✅ | ✅ | 🔒 |
| **Plants** | | | |
| View all plants | ✅ | ✅ | 👁 assigned farms only |
| Register new plant | ✅ | ✅ | ⏳ pending approval |
| Edit plant details | ✅ | ✅ | 🔒 |
| Deactivate plant | ✅ | ✅ | 🔒 |
| **Seasons** | | | |
| View seasons | ✅ | ✅ | 👁 |
| Create / close season | ✅ | ✅ | 🔒 |

---

## 4. Field Operations

| Action | Super Admin | Admin | Field Assistant |
|---|---|---|---|
| View nearby farms / plants (GPS) | ✅ | ✅ | ✅ |
| View farm profile and history | ✅ | ✅ | ✅ assigned only |
| View plant profile and history | ✅ | ✅ | ✅ assigned only |
| Start a visit | ✅ | ✅ | ✅ assigned only |
| Submit visit data | ✅ | ✅ | ✅ |
| Edit submitted visit | ✅ | ✅ | 🔒 |
| Delete visit record | ✅ | 🔒 | 🔒 |
| View visit AI analysis results | ✅ | ✅ | ✅ own visits |
| Register new farm | ✅ | ✅ | ⏳ |
| Register new plant | ✅ | ✅ | ⏳ |
| View assigned visit schedule | ✅ | ✅ | ✅ own schedule |

---

## 5. Approval Workflow

| Action | Super Admin | Admin | Field Assistant |
|---|---|---|---|
| View approval queue | ✅ | ✅ | 🔒 |
| View own pending registrations | ✅ | ✅ | ✅ |
| Approve farm registration | ✅ | ✅ | 🔒 |
| Reject farm registration | ✅ | ✅ | 🔒 |
| Approve plant registration | ✅ | ✅ | 🔒 |
| Reject plant registration | ✅ | ✅ | 🔒 |
| Add rejection reason / comment | ✅ | ✅ | 🔒 |
| Re-submit rejected registration | ✅ | ✅ | ✅ own only |

---

## 6. Scheduling & Alerts

| Action | Super Admin | Admin | Field Assistant |
|---|---|---|---|
| View visit schedule (all farms) | ✅ | ✅ | 🔒 |
| View own assigned schedule | ✅ | ✅ | ✅ |
| Assign visit to field assistant | ✅ | ✅ | 🔒 |
| Reassign visit | ✅ | ✅ | 🔒 |
| Mark visit as overdue (manual) | ✅ | ✅ | 🔒 |
| View overdue alerts | ✅ | ✅ | ✅ own only |
| Configure alert thresholds | ✅ | 🔒 | 🔒 |
| Receive push notifications | ✅ | ✅ | ✅ |

---

## 7. Sync & Offline

| Action | Super Admin | Admin | Field Assistant |
|---|---|---|---|
| View sync status | ✅ | 👁 | ✅ own device |
| Trigger manual sync | ✅ | 🔒 | ✅ own device |
| View sync error logs | ✅ | 👁 | 🔒 |
| Clear local sync queue | ✅ | 🔒 | ✅ own device |

---

## 8. AI & Analysis

| Action | Super Admin | Admin | Field Assistant |
|---|---|---|---|
| View AI analysis results | ✅ | ✅ | ✅ own visits |
| Trigger manual AI re-analysis | ✅ | ✅ | 🔒 |
| View AI error / failure log | ✅ | 👁 | 🔒 |
| Configure AI API settings | ✅ | 🔒 | 🔒 |

---

## 9. Commodity & Predictions

| Action | Super Admin | Admin | Field Assistant |
|---|---|---|---|
| View commodity price predictions | ✅ | ✅ | 🔒 |
| View yield forecast charts | ✅ | ✅ | 🔒 |
| Filter predictions by region / crop | ✅ | ✅ | 🔒 |
| View historical prediction data | ✅ | ✅ | 🔒 |
| Configure commodity settings | ✅ | 🔒 | 🔒 |

---

## 10. Reporting & Exports

| Action | Super Admin | Admin | Field Assistant |
|---|---|---|---|
| View visit summary reports | ✅ | ✅ | 🔒 |
| View agent performance reports | ✅ | ✅ | 🔒 |
| View prediction history reports | ✅ | ✅ | 🔒 |
| Export reports as CSV | ✅ | ✅ | 🔒 |
| Export reports as PDF | ✅ | ✅ | 🔒 |
| Schedule automated reports | ✅ | 🔒 | 🔒 |

---

## Summary Matrix — Screen Access

| Screen | Super Admin | Admin | Field Assistant |
|---|---|---|---|
| **Shared** | | | |
| SSO Login | ✅ | ✅ | ✅ |
| Language Select | ✅ | ✅ | ✅ |
| Profile & Settings | ✅ | ✅ | ✅ |
| **Field Assistant Only** | | | |
| Home — Nearby (GPS) | ✅ | ✅ | ✅ |
| Farm Profile | ✅ | ✅ | ✅ assigned |
| Plant Profile | ✅ | ✅ | ✅ assigned |
| Season Profile | ✅ | ✅ | ✅ assigned |
| Visit Execution | ✅ | ✅ | ✅ assigned |
| Photo Capture | ✅ | ✅ | ✅ |
| Visit Summary | ✅ | ✅ | ✅ |
| Visit Result | ✅ | ✅ | ✅ own |
| Register Farm | ✅ | ✅ | ✅ |
| Register Plant | ✅ | ✅ | ✅ |
| Registration Submitted | ✅ | ✅ | ✅ |
| Sync Status | ✅ | 🔒 | ✅ |
| Help Library | ✅ | ✅ | ✅ |
| Help Content View | ✅ | ✅ | ✅ |
| Assigned Visits | ✅ | ✅ | ✅ own |
| **Admin Mobile** | | | |
| Admin Dashboard | ✅ | ✅ | 🔒 |
| Approval Queue | ✅ | ✅ | 🔒 |
| Approval Detail | ✅ | ✅ | 🔒 |
| Farm Registry | ✅ | ✅ | 🔒 |
| Farm Detail (Admin) | ✅ | ✅ | 🔒 |
| Plant Registry | ✅ | ✅ | 🔒 |
| Agent Overview | ✅ | ✅ | 🔒 |
| Schedule Overview | ✅ | ✅ | 🔒 |
| Price Predictions | ✅ | ✅ | 🔒 |
| **Admin Web — Phase 2** | | | |
| Crop Type Manager | ✅ | 🔒 | 🔒 |
| Visit Template Builder | ✅ | ✅ | 🔒 |
| Growth Stage Manager | ✅ | 🔒 | 🔒 |
| Schedule Rule Config | ✅ | ✅ | 🔒 |
| Commodity Manager | ✅ | 🔒 | 🔒 |
| Help Content Manager | ✅ | ✅ | 🔒 |
| Region Manager | ✅ | 🔒 | 🔒 |
| User Manager | ✅ | 🔒 | 🔒 |
| Farmer Registry | ✅ | ✅ | 🔒 |
| Reports — Visits | ✅ | ✅ | 🔒 |
| Reports — Predictions | ✅ | ✅ | 🔒 |
| Reports — Agent Performance | ✅ | ✅ | 🔒 |

---

## Role Assignment Rules

- Roles are assigned by Super Admin only
- A user can have only one role at a time
- Role is determined at SSO login — stored in the platform user record
- New SSO users default to no role and cannot access the app until a Super Admin assigns a role
- Role changes take effect on next login

## Field Assistant Access Restrictions

Field assistants can only see data within their assigned scope:

- **Farms** — only farms assigned to them by admin
- **Plants** — only plants belonging to their assigned farms
- **Visits** — only their own visits
- **Nearby screen** — shows all nearby farms but visiting is restricted to assigned farms
- **Schedules** — only their own assigned schedule

---

*AgriField — Commodity Intelligence Platform | Confidential — Internal Use Only*