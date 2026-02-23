# Multilingual Specification
**Version:** 1.0  
**Status:** Draft  
**Date:** February 2026  

---

## Overview

AgriField supports multiple languages from day one. This document defines how i18n is implemented in the mobile app, how admin-configured content handles multilingual labels, and the process for adding new languages.

---

## 1. Supported Languages

| Code | Language | Status |
|---|---|---|
| `en` | English | ✅ Launch |
| `ms` | Bahasa Malaysia | ✅ Launch |
| `id` | Bahasa Indonesia | 🔶 Phase 2 |

Additional languages can be added post-launch without code changes.

**Default language:** `en` (used as fallback if a translation is missing)

---

## 2. Language Detection & Selection

### Priority Order (highest to lowest)

1. User's saved language preference (from `users.language_preference` in DB)
2. Device locale (detected via `expo-localization`)
3. Default fallback: `en`

### Language Selection UI

- Shown on first app launch after login
- Available in Profile & Settings at any time
- Change takes effect immediately — no app restart required
- Preference saved to server via `PATCH /users/me`

---

## 3. Static UI Strings (i18next)

All static UI text (labels, button text, error messages, empty states) is managed via **i18next** with **react-i18next**.

### Directory Structure

```
src/
└── i18n/
    ├── index.ts           ← i18next init
    ├── en/
    │   ├── common.json
    │   ├── navigation.json
    │   ├── auth.json
    │   ├── nearby.json
    │   ├── visit.json
    │   ├── farm.json
    │   ├── plant.json
    │   ├── sync.json
    │   ├── approval.json
    │   ├── admin.json
    │   └── errors.json
    ├── ms/
    │   └── (same structure)
    └── id/
        └── (same structure)
```

### i18next Initialisation

```typescript
import i18n from 'i18next'
import { initReactI18next } from 'react-i18next'
import * as Localization from 'expo-localization'

i18n
  .use(initReactI18next)
  .init({
    compatibilityJSON: 'v3',
    lng: getUserLanguage() || Localization.locale.split('-')[0] || 'en',
    fallbackLng: 'en',
    resources: {
      en: { ... },
      ms: { ... },
      id: { ... }
    },
    interpolation: { escapeValue: false }
  })
```

### Usage in Components

```typescript
import { useTranslation } from 'react-i18next'

function NearbyScreen() {
  const { t } = useTranslation('nearby')
  return <Text>{t('title')}</Text>
}
```

### Translation Key Naming Convention

```
namespace.section.key
```

Examples:
```json
// nearby.json
{
  "title": "Nearby Farms & Plants",
  "searching": "Searching within {{radius}}m...",
  "no_farms_found": "No farms found nearby",
  "register_cta": "Register New Farm / Plant",
  "distance": "{{distance}}m away",
  "last_visit": "Last visit: {{date}}",
  "overdue_badge": "Visit overdue"
}

// errors.json
{
  "gps_unavailable": "Location unavailable — please enable GPS",
  "offline": "You are offline — data will sync when connected",
  "session_expired": "Your session has expired — please log in again",
  "sync_failed": "Sync failed — tap to retry"
}
```

### Interpolation Examples

```typescript
// Simple value
t('nearby.distance', { distance: 145 })
// → "145m away"

// Date formatting
t('nearby.last_visit', { date: formatDate(visit.visited_at) })
// → "Last visit: 10 Feb 2026"

// Plural
t('sync.pending_count', { count: 3 })
// en → "3 items pending upload"
// ms → "3 item menunggu muat naik"
```

---

## 4. Dynamic Content (Admin-Configured Multilingual)

Admin-configured content — crop type names, visit template step labels, growth stage names, help content titles — also supports multiple languages. These are stored as JSON maps in the database.

### Storage Format

```json
{
  "en": "Take a close-up photo of the bunch",
  "ms": "Ambil gambar close-up tandan",
  "id": "Ambil foto close-up tandan"
}
```

### Rendering in App

```typescript
// Utility function
function getLocalizedText(
  textMap: Record<string, string>,
  language: string,
  fallback = 'en'
): string {
  return textMap[language] || textMap[fallback] || Object.values(textMap)[0] || ''
}

// Usage
const stepLabel = getLocalizedText(step.label, currentLanguage)
```

### Admin Input in Template Builder

When an admin creates or edits a visit template step, they fill in labels for each active language:

```
Step Label:
  English:           [Take a close-up photo of the bunch     ]
  Bahasa Malaysia:   [Ambil gambar close-up tandan           ]
  Bahasa Indonesia:  [Ambil foto close-up tandan             ] (optional)
```

- English is always required
- Other language fields are optional — fallback to English if empty
- Language tabs shown in the template builder based on active languages

---

## 5. Date, Time & Number Formatting

All date, time, and number display is localised using the device locale via **Intl** API (built into React Native's Hermes engine).

```typescript
// Date formatting
const formatDate = (isoDate: string, language: string) =>
  new Intl.DateTimeFormat(language, {
    day: 'numeric', month: 'short', year: 'numeric'
  }).format(new Date(isoDate))

// → en: "23 Feb 2026"
// → ms: "23 Feb 2026" (same format, different locale context)

// Number formatting (yield, area)
const formatNumber = (value: number, language: string, unit?: string) =>
  new Intl.NumberFormat(language, {
    maximumFractionDigits: 2
  }).format(value) + (unit ? ` ${unit}` : '')

// → en: "2,600 kg"
// → ms: "2,600 kg"

// Relative time (last visit)
const formatRelativeTime = (date: string, language: string) => {
  const rtf = new Intl.RelativeTimeFormat(language, { numeric: 'auto' })
  const days = Math.round((Date.now() - new Date(date).getTime()) / 86400000)
  if (days === 0) return rtf.format(0, 'day')   // "today"
  if (days < 7) return rtf.format(-days, 'day') // "3 days ago"
  // ... weeks, months
}
```

---

## 6. RTL Support

Current target languages (English, Bahasa Malaysia, Bahasa Indonesia) are all LTR. RTL support is not required at launch but the codebase should be structured to avoid hardcoded directional assumptions:

- Use `flexDirection: 'row'` instead of hardcoded `left`/`right` margins where possible
- Use `I18nManager.isRTL` from React Native when directional logic is needed
- Do not hardcode text alignment to `left` — use `text-align: start` pattern

---

## 7. Adding a New Language

Steps to add a new language (e.g. Thai — `th`):

1. **Create translation files** — Copy `src/i18n/en/` folder to `src/i18n/th/` and translate all keys
2. **Register in i18next** — Add `th` resources to `i18n/index.ts`
3. **Add to language selector** — Add Thai option to Profile & Settings language picker
4. **Add to admin template builder** — Add Thai tab to multilingual label inputs
5. **Update `users.language_preference`** — Add `th` as valid enum value
6. **Translate seed data** — Add `"th": "..."` entries to all master data seed JSON files
7. **QA** — Test all screens in Thai including RTL check (Thai is LTR)

No app code changes required beyond steps 1-3. Server-side changes are configuration only.

---

## 8. Translation Workflow

### Who Provides Translations?

| Content | Owner |
|---|---|
| Static UI strings (JSON files) | Product team / professional translator |
| Visit template step labels | Admin user (when configuring templates) |
| Crop type names | Super Admin (when setting up master data) |
| Help content titles | Admin user (when uploading content) |
| Error messages | Product team / professional translator |

### Translation Review Process

1. English copy finalised by product team
2. Sent to translator (human or verified MT)
3. Translator provides all target language JSON files
4. Developer imports JSON files — no string changes in code
5. QA reviews all screens in each language
6. Sign-off before launch

### Missing Translation Handling

If a translation key is missing in the active language, the app falls back gracefully:

```
Active language: ms
Key: nearby.new_feature_label (not yet translated)
Fallback: en value used
Log: warning in development builds only
```

No untranslated keys should reach production — CI/CD pipeline includes a translation completeness check.

---

## 9. Translation Completeness Check (CI/CD)

Add to the build pipeline:

```bash
#!/bin/bash
# Check all keys in en/ exist in all other language files
node scripts/check-translations.js --base en --check ms,id
# Exits with error if any key is missing
# This runs on every PR targeting main
```

---

*AgriField — Commodity Intelligence Platform | Confidential — Internal Use Only*