# Saathi — UI/UX Flow Design Document

**Project:** AI Settlement & Immigration Companion for the Nepalese diaspora in Australia  
**Stack:** Next.js PWA (mobile-first), bilingual (English/Nepali)  
**Date:** August 2026  
**Status:** UX Design Phase — Screen-by-screen flows

---

## Table of Contents

1. [Cross-Cutting Design System](#1-cross-cutting-design-system)
2. [F1 — Visa Tracker](#2-f1--visa-tracker)
3. [F2 — Points Calculator](#3-f2--points-calculator)
4. [F3 — Document Checklist](#4-f3--document-checklist)
5. [F4 — Form Helper](#5-f4--form-helper)
6. [Component Library](#6-component-library)
7. [Accessibility & WCAG 2.1 AA](#7-accessibility--wcag-21-aa)
8. [Reference App Patterns](#8-reference-app-patterns)

---

## 1. Cross-Cutting Design System

### 1.1 Surface Archetypes (per claude-design methodology)

| Feature | Surface | Dominant UX Need |
|---------|---------|-----------------|
| F1 — Visa Tracker | **Monitor** | Watch state change; density + glanceability |
| F2 — Points Calculator | **Configure** | Progressive disclosure wizard; clear validation |
| F3 — Document Checklist | **Operate** | Act on items; selection state dominates |
| F4 — Form Helper | **Configure + Inspect** | Field-level drilling; speed over breadth |

### 1.2 Design Tokens

#### Color Palette — "Safe & Trustworthy"

Inspired by Australian government blues + warm Nepali terracotta accents:

```
--saathi-navy-900:     #0B1E3D    /* Primary text, deep headers */
--saathi-navy-700:     #1A3A6B    /* Secondary text */
--saathi-blue-600:     #0066CC    /* Primary CTA, links */
--saathi-blue-500:     #2B7FFF    /* Accent, active states */
--saathi-blue-100:     #E6F0FF    /* Selected/active backgrounds */
--saathi-blue-50:      #F5F9FF    /* Card backgrounds, subtle highlights */

--saathi-terracotta-500: #D45D3E  /* Urgent: expiring visas, alerts */
--saathi-terracotta-100: #FDF0EC  /* Alert backgrounds */

--saathi-green-600:    #2D8A4E    /* Success: completed, verified */
--saathi-green-100:    #EDF7F0    /* Success backgrounds */
--saathi-amber-500:    #E8A317    /* Warning: attention needed */
--saathi-amber-100:    #FEF8E7    /* Warning backgrounds */

--saathi-gray-900:     #1A1A2E    /* Text primary */
--saathi-gray-600:     #6B7280    /* Text muted */
--saathi-gray-300:     #D1D5DB    /* Borders, dividers */
--saathi-gray-100:     #F3F4F6    /* Surface backgrounds */
--saathi-gray-50:      #FAFBFC    /* Page background */
--saathi-white:        #FFFFFF    /* Card backgrounds */
```

#### Typography — Bilingual First

**Primary Font Stack (Latin):** Inter (Google Fonts) — clean, highly readable, excellent at small sizes for mobile  
**Devanagari Font:** Mukta (Google Fonts) — open-source, designed for Hindi/Nepali, pairs well with Inter  
**Fallback Stack:** `'Inter', 'Mukta', -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Noto Sans Devanagari', sans-serif`

```css
/* Devanagari-specific adjustments */
[lang="ne"] {
  font-family: 'Mukta', 'Noto Sans Devanagari', sans-serif;
  line-height: 1.7;  /* Devanagari needs more vertical space for matras */
  font-size: 1.05em; /* Slightly larger for readability of complex glyphs */
}
```

**Type Scale (mobile-first):**

| Token | Size | Use |
|-------|------|-----|
| `text-xs` | 0.75rem / 12px | Captions, legal disclaimers |
| `text-sm` | 0.875rem / 14px | Body small, secondary text |
| `text-base` | 1rem / 16px | Body, form labels, inputs |
| `text-lg` | 1.125rem / 18px | Card titles, emphasized body |
| `text-xl` | 1.25rem / 20px | Section headers |
| `text-2xl` | 1.5rem / 24px | Page titles |
| `text-3xl` | 1.875rem / 30px | Feature headers, countdown numbers |

#### Spacing (4px grid)

```
--space-1:  4px
--space-2:  8px
--space-3:  12px
--space-4:  16px
--space-5:  20px
--space-6:  24px
--space-8:  32px
--space-10: 40px
--space-12: 48px
--space-16: 64px
```

#### Border Radius

```
--radius-sm:  6px   /* Buttons, inputs, small cards */
--radius-md:  10px  /* Cards, modals */
--radius-lg:  16px  /* Feature cards, dashboard tiles */
--radius-full: 9999px  /* Pills, badges, language toggle */
```

#### Shadows

```
--shadow-sm:  0 1px 2px rgba(0,0,0,0.05)
--shadow-md:  0 4px 6px -1px rgba(0,0,0,0.07), 0 2px 4px -2px rgba(0,0,0,0.05)
--shadow-lg:  0 10px 15px -3px rgba(0,0,0,0.08), 0 4px 6px -4px rgba(0,0,0,0.04)
--shadow-xl:  0 20px 25px -5px rgba(0,0,0,0.1), 0 8px 10px -6px rgba(0,0,0,0.04)
```

### 1.3 Navigation Pattern — Bottom Tab Bar

**Decision:** Bottom tab bar (iOS/Android native pattern) over sidebar or hamburger menu.  
**Rationale:** 
- 4 primary features = clean 4-tab layout
- Mobile-first PWA; thumb-reachable
- Familiar pattern for the target demographic (WhatsApp, Facebook, banking apps)

```
┌─────────────────────────────────────────┐
│  [App Bar: Feature Title + Lang Toggle] │
├─────────────────────────────────────────┤
│                                         │
│         Feature Content Area            │
│                                         │
│                                         │
├─────────────────────────────────────────┤
│  🛂 Visa    📊 Points   📋 Docs   📝 Help│
│  Tracker   Calculator  Checklist  Forms │
└─────────────────────────────────────────┘
```

**Tab icons:** Custom SVG icons — passport/book for Visa, calculator/abacus for Points, clipboard/checklist for Docs, form/question-mark for Help.  
**Active state:** Filled icon + blue-600 label  
**Inactive state:** Outlined icon + gray-600 label  
**Badge support:** Dot badge on Visa tab when visa expiring within 30 days

### 1.4 Language Toggle

**Position:** Top-right of app bar, always visible  
**Design:** Segmented pill button — `EN | ने`  
**Behavior:**
- Toggles ALL UI strings instantly (React context + i18n)
- Persists to localStorage
- Content entered by user in one language is NOT auto-translated
- AI-generated explanations respect the current language setting
- Default: detect browser language, fallback to English

**Component:**
```
┌──────────────────────────────────────────┐
│  Saathi                        [EN | ने] │
└──────────────────────────────────────────┘
```

### 1.5 Trust Signals — Always Visible

Every screen below the fold / at scroll-bottom includes:

1. **MARA disclaimer bar** (fixed on legal screens):
   > *"Saathi is an informational tool, not a registered migration agent (MARA). For official advice, consult immi.homeaffairs.gov.au"*

2. **Source citation pattern:**
   > 📋 *Source: immi.homeaffairs.gov.au/skilled-visa-points-test — Verified August 2026*

3. **"Verified" badge** on any content sourced from official channels:
   ```
   ┌──────────────────────────────┐
   │ ✅ VERIFIED  ·  Updated Jun 2026 │
   └──────────────────────────────┘
   ```

4. **Data privacy badge** on document-upload screens:
   > 🔒 *Your documents never leave your device. Processed locally.*

### 1.6 Onboarding Flow (First Launch)

The first-launch experience introduces all 4 features progressively:

**Screen 1 — Welcome (Decide/Learn surface)**
```
┌─────────────────────────────────────┐
│                                     │
│        🏔️  (Nepali flag + AU flag) │
│                                     │
│     स्वागत छ! Welcome to Saathi     │
│                                     │
│   Your AI companion for settling   │
│      in Australia — from visa      │
│     tracking to form filling.      │
│                                     │
│       Bilingual. Free. Private.    │
│                                     │
│         [ Get Started → ]          │
│                                     │
│   Choose your language: [EN | ने]  │
│                                     │
└─────────────────────────────────────┘
```

**Screen 2 — Feature Tour (swipeable carousel, 4 slides)**
```
┌─────────────────────────────────────┐
│  ● ○ ○ ○    Skip →                  │
│                                     │
│  🛂                                  │
│  Track Your Visa                    │
│                                     │
│  Never miss a visa deadline. See    │
│  conditions at a glance, get        │
│  reminders before expiry.           │
│                                     │
│  [image: visa countdown mockup]     │
│                                     │
└─────────────────────────────────────┘
```
(Swipe for Points Calculator, Document Checklist, Form Helper)

**Screen 3 — Quick Setup (optional)**
- "Add a visa to get started" → skip-able CTA
- "What brings you to Australia?" → select purpose (study/work/family/PR)
- Skip button always visible

**Onboarding completion:** User lands on the Visa Tracker tab with "Add your first visa" empty state.

---

## 2. F1 — Visa Tracker

### 2.1 Surface: Monitor

The Visa Tracker is a **Monitor** surface: the user watches state change — countdown, conditions, alerts. Density and glanceable hierarchy are paramount. No hero, no feature cards — just information that helps the user decide and act.

### 2.2 Screen-by-Screen Flow

#### Flow Map

```
Empty State → Add Visa Wizard → Dashboard → Detail Screens → Settings/Reminders
                ↑                                              │
                └──────── Edit Visa ←──────────────────────────┘

Multi-Visa: Dashboard shows stacked visa cards → tap to switch active
```

#### 2.2.1 Empty State (Screen V0)

Shown when no visa has been added.

```
┌────────────────────────────────────────┐
│  Saathi                          EN|ने │
├────────────────────────────────────────┤
│                                        │
│                                        │
│            🛂 (large icon)             │
│                                        │
│       Track your Australian visa       │
│                                        │
│   Add your current visa to see your   │
│   countdown, conditions, and get      │
│   reminders before it expires.        │
│                                        │
│        ┌─────────────────────┐         │
│        │  + Add My Visa      │         │
│        └─────────────────────┘         │
│                                        │
│   Popular visa types:                  │
│   • Student Visa (Subclass 500)        │
│   • Skilled Graduate (Subclass 485)    │
│   • Skilled Regional (Subclass 491)    │
│                                        │
│   💡 No account needed. Data stays    │
│      on your device.                  │
│                                        │
├────────────────────────────────────────┤
│  🛂 Visa  📊 Points  📋 Docs  📝 Help │
└────────────────────────────────────────┘
```

**Pattern reference:** Todoist empty project state ("Add your first task"), Flighty empty state ("Add a flight")

#### 2.2.2 Add Visa — Step-by-Step Wizard (V1–V4)

A 4-step wizard, not a single long form. Each step is one question with clear forward/back navigation.

**Step V1 — Visa Type**
```
┌────────────────────────────────────────┐
│  ← Back          Add Visa      1 of 4  │
├────────────────────────────────────────┤
│                                        │
│  What type of visa do you have?        │
│                                        │
│  ┌──────────────────────────────────┐  │
│  │ 🎓 Student Visa (Subclass 500)  │  │
│  └──────────────────────────────────┘  │
│  ┌──────────────────────────────────┐  │
│  │ 🎓 Graduate Visa (Subclass 485) │  │
│  └──────────────────────────────────┘  │
│  ┌──────────────────────────────────┐  │
│  │ 💼 Skilled Regional (491/494)   │  │
│  └──────────────────────────────────┘  │
│  ┌──────────────────────────────────┐  │
│  │ 🏠 Skilled Independent (189)    │  │
│  └──────────────────────────────────┘  │
│  ┌──────────────────────────────────┐  │
│  │ 👔 Employer Sponsored (482/186) │  │
│  └──────────────────────────────────┘  │
│  ┌──────────────────────────────────┐  │
│  │ 👨‍👩‍👧 Partner/Family Visa         │  │
│  └──────────────────────────────────┘  │
│  ┌──────────────────────────────────┐  │
│  │ 📋 Other — I'll enter details    │  │
│  └──────────────────────────────────┘  │
│                                        │
│          [ Continue → ]                │
│                                        │
└────────────────────────────────────────┘
```

**Step V2 — Key Dates**
```
┌────────────────────────────────────────┐
│  ← Back       Key Dates        2 of 4  │
├────────────────────────────────────────┤
│                                        │
│  When did your visa start and end?     │
│                                        │
│  Grant Date                            │
│  ┌──────────────────────────────────┐  │
│  │ 📅  15 March 2024               │  │
│  └──────────────────────────────────┘  │
│                                        │
│  Expiry Date                           │
│  ┌──────────────────────────────────┐  │
│  │ 📅  15 March 2026               │  │
│  └──────────────────────────────────┘  │
│                                        │
│  ┌──────────────────────────────────┐  │
│  │ ☐ I don't know the exact dates  │  │
│  │   Estimate based on visa type    │  │
│  └──────────────────────────────────┘  │
│                                        │
│          [ Continue → ]                │
│                                        │
└────────────────────────────────────────┘
```

**Pattern:** Native date pickers (iOS/Android), with manual entry fallback. "Estimate" toggle uses Department of Home Affairs typical grant durations.

**Step V3 — Visa Conditions**
```
┌────────────────────────────────────────┐
│  ← Back       Conditions       3 of 4  │
├────────────────────────────────────────┤
│                                        │
│  What conditions apply to your visa?   │
│  (Check your grant letter)            │
│                                        │
│  ☑ 8105 — Work limitation              │
│    (Max 48 hrs/fortnight during study)  │
│                                        │
│  ☑ 8202 — Meet course requirements     │
│                                        │
│  ☐ 8501 — Maintain health insurance    │
│                                        │
│  ☐ 8533 — Notify change of address     │
│    within 7 days                       │
│                                        │
│  ☐ I'm not sure — detect from visa     │
│    type                                │
│                                        │
│  💡 Each condition explained in        │
│     plain English & Nepali             │
│                                        │
│          [ Continue → ]                │
│                                        │
└────────────────────────────────────────┘
```

**Step V4 — Notification Preferences**
```
┌────────────────────────────────────────┐
│  ← Back       Reminders        4 of 4  │
├────────────────────────────────────────┤
│                                        │
│  When should we remind you?            │
│                                        │
│  ┌──────────────────────────────────┐  │
│  │ 🔔 90 days before expiry         │  │
│  │     ☑ Push  ☐ Email             │  │
│  └──────────────────────────────────┘  │
│  ┌──────────────────────────────────┐  │
│  │ 🔔 60 days before expiry         │  │
│  │     ☑ Push  ☐ Email             │  │
│  └──────────────────────────────────┘  │
│  ┌──────────────────────────────────┐  │
│  │ 🔔 30 days before expiry         │  │
│  │     ☑ Push  ☐ Email             │  │
│  └──────────────────────────────────┘  │
│  ┌──────────────────────────────────┐  │
│  │ 🔔 7 days before expiry          │  │
│  │     ☑ Push  ☑ Email             │  │
│  └──────────────────────────────────┘  │
│                                        │
│  ┌──────────────────────────────────┐  │
│  │ ☐ Weekly condition compliance    │  │
│  │    reminder (e.g. work hours)    │  │
│  └──────────────────────────────────┘  │
│                                        │
│       [ Save & Go to Dashboard ]       │
│                                        │
└────────────────────────────────────────┘
```

**Wizard completion animation:** Confetti/sparkle → "Visa added!" toast → transition to dashboard.

#### 2.2.3 Main Dashboard (Screen V5)

The core Monitor surface. Glanceable, dense, action-oriented.

```
┌────────────────────────────────────────┐
│  Saathi                          EN|ने │
├────────────────────────────────────────┤
│                                        │
│  ┌──────────────────────────────────┐  │
│  │  Student Visa (Subclass 500)     │  │
│  │                                  │  │
│  │        ┌─────────────────┐       │  │
│  │        │                 │       │  │
│  │        │   457 days      │       │  │
│  │        │   remaining     │       │  │
│  │        │                 │       │  │
│  │        │ Expires         │       │  │
│  │        │ 15 Mar 2026     │       │  │
│  │        └─────────────────┘       │  │
│  │                                  │  │
│  │  ████████████░░░░░░  62% used   │  │
│  │                                  │  │
│  │  Grant date: 15 Mar 2024         │  │
│  └──────────────────────────────────┘  │
│                                        │
│  ┌──────────────────────────────────┐  │
│  │ ⚠️  Visa Conditions              │  │
│  │                                  │  │
│  │  ✅ 8105 — Work limit (48h/fn)   │  │
│  │  ✅ 8202 — Course requirements    │  │
│  │  ⚠️  8501 — Health insurance      │  │
│  │     Renew by 30 Jun 2026         │  │
│  │                                  │  │
│  │  [ View All Conditions → ]       │  │
│  └──────────────────────────────────┘  │
│                                        │
│  Next Steps                            │
│  ┌──────────────────────────────────┐  │
│  │ 📝 Apply for 485 Graduate Visa   │  │
│  │    Recommended within 6 months    │  │
│  │                         [Start]  │  │
│  └──────────────────────────────────┘  │
│  ┌──────────────────────────────────┐  │
│  │ 📋 Gather documents for renewal  │  │
│  │    3 items needed                │  │
│  │                         [View]   │  │
│  └──────────────────────────────────┘  │
│                                        │
├────────────────────────────────────────┤
│  🛂 Visa  📊 Points  📋 Docs  📝 Help │
└────────────────────────────────────────┘
```

**Countdown timer design:**
- Visual: Large circular progress ring (SVG) showing percentage used
- Color shifts: Green (> 6 months) → Amber (1–6 months) → Red (< 1 month)
- "457 days remaining" in large type
- Smaller text: "Expires 15 March 2026"
- Pattern reference: Flighty app countdown, medication reminder apps

**Progress bar:** Linear bar below countdown — "62% used" helps contextualize

**Condition cards:** 
- Status icons: ✅ (compliant), ⚠️ (attention needed), ❌ (breach risk)
- Tap to expand full condition text with bilingual explanation

**Next Steps section:**
- Contextual, AI-suggested actions based on visa type + remaining time
- Each suggestion is a card with a CTA button
- Examples: "Apply for 485 before your 500 expires", "Book health check for visa medical"

#### 2.2.4 Condition Detail (Screen V6)

Tapping a condition card opens the detail view:

```
┌────────────────────────────────────────┐
│  ← Dashboard    Condition 8105         │
├────────────────────────────────────────┤
│                                        │
│  ⚠️  Work Limitation                   │
│                                        │
│  Condition 8105                        │
│                                        │
│  ┌──────────────────────────────────┐  │
│  │ English                          │  │
│  │ You must not work more than      │  │
│  │ 48 hours per fortnight while     │  │
│  │ your course is in session.       │  │
│  │                                  │  │
│  │ During scheduled course breaks,  │  │
│  │ you may work unlimited hours.    │  │
│  └──────────────────────────────────┘  │
│                                        │
│  ┌──────────────────────────────────┐  │
│  │ नेपाली                           │  │
│  │ तपाईंले आफ्नो पाठ्यक्रम चलिरहेको │  │
│  │ बेला प्रति पन्ध्र दिन ४८ घण्टाभन्दा│  │
│  │ बढी काम गर्न पाउनुहुन्न।         │  │
│  │                                  │  │
│  │ तोकिएको बिदाको समयमा असीमित घण्टा│  │
│  │ काम गर्न पाइन्छ।                 │  │
│  └──────────────────────────────────┘  │
│                                        │
│  ┌──────────────────────────────────┐  │
│  │ 💡 What this means for you       │  │
│  │ Track your fortnightly hours.    │  │
│  │ Going over 48h risks visa        │  │
│  │ cancellation. Use the work-hour  │  │
│  │ tracker below.                   │  │
│  └──────────────────────────────────┘  │
│                                        │
│  📋 Source: immi.homeaffairs.gov.au    │
│     Verified August 2026              │
│                                        │
│  [ Mark as Understood ]               │
│                                        │
└────────────────────────────────────────┘
```

**Pattern:** Collapsible bilingual display (English expanded by default, Nepali tappable). "What this means" is an AI-generated plain-language summary. Always shows source citation.

#### 2.2.5 Multi-Visa Support (V7)

Users can add multiple visas (current student visa + past tourist visa + future 485). Dashboard shows stacked cards:

```
┌────────────────────────────────────────┐
│  Your Visas                     + Add  │
├────────────────────────────────────────┤
│                                        │
│  ┌──────────────────────────────────┐  │
│  │ 🟢 CURRENT                       │  │
│  │ Student Visa (500)               │  │
│  │ 457 days · Expires 15 Mar 2026   │  │
│  │ ████████░░░░  62%           >    │  │
│  └──────────────────────────────────┘  │
│                                        │
│  ┌──────────────────────────────────┐  │
│  │ ⚪ PAST                          │  │
│  │ Tourist Visa (600)               │  │
│  │ Expired 10 Jan 2024          >   │  │
│  └──────────────────────────────────┘  │
│                                        │
│  ┌──────────────────────────────────┐  │
│  │ 🔵 FUTURE                        │  │
│  │ Graduate Visa (485)              │  │
│  │ Not yet granted · Apply now  >   │  │
│  └──────────────────────────────────┘  │
│                                        │
└────────────────────────────────────────┘
```

**Visual hierarchy:**
- Current visa: full-width, colored border, countdown
- Past visas: collapsed, grayed out, expandable for reference
- Future visas: outlined, dashed border, shows "Apply" state

**Pattern reference:** Flighty app multi-flight view, TripIt itinerary stacking

#### 2.2.6 Notification Settings (V8)

Accessible from dashboard gear icon:

```
┌────────────────────────────────────────┐
│  ← Dashboard    Notifications          │
├────────────────────────────────────────┤
│                                        │
│  Push Notifications                    │
│  ┌──────────────────────────────────┐  │
│  │ Visa expiry reminders     [ON]   │  │
│  │ Condition check reminders [ON]   │  │
│  │ Policy change alerts      [OFF]  │  │
│  │ Weekly digest             [OFF]  │  │
│  └──────────────────────────────────┘  │
│                                        │
│  Email Notifications                   │
│  ┌──────────────────────────────────┐  │
│  │ 7-day warning             [ON]   │  │
│  │ 30-day warning            [OFF]  │  │
│  │ Policy changes            [OFF]  │  │
│  └──────────────────────────────────┘  │
│                                        │
│  Reminder Schedule                     │
│  ┌──────────────────────────────────┐  │
│  │ 90 days before  ☑  Push         │  │
│  │ 60 days before  ☑  Push         │  │
│  │ 30 days before  ☑  Push ☑ Email │  │
│  │ 7 days before   ☑  Push ☑ Email │  │
│  │ 1 day before    ☑  Push         │  │
│  └──────────────────────────────────┘  │
│                                        │
└────────────────────────────────────────┘
```

#### 2.2.7 Loading & Error States

**Loading (Dashboard):**
```
┌────────────────────────────────────────┐
│                                        │
│        ┌──────────┐                    │
│        │ ⟳ Loading │                   │
│        └──────────┘                    │
│                                        │
│     Fetching your visa details...      │
│                                        │
└────────────────────────────────────────┘
```

**Error (No connection):**
```
┌────────────────────────────────────────┐
│                                        │
│            📡 (icon)                   │
│                                        │
│       Can't reach the server           │
│                                        │
│   Your saved visas are still           │
│   available offline.                   │
│                                        │
│         [ Try Again ]                  │
│                                        │
│   Showing data from: 2 Aug 2026        │
│                                        │
└────────────────────────────────────────┘
```

**Offline indicator:** Persistent thin banner at top:
> `⚠️ Offline — Showing cached data from 2 August 2026`

### 2.3 Reference App Patterns for F1

| Pattern | Reference App | Why |
|---------|---------------|-----|
| Countdown timer with color shift | **Flighty** | Elegant flight countdown, shifts from blue → orange → red |
| Medication-style reminders | **Medisafe / Apple Health** | Scheduled notifications with escalating urgency |
| Multi-itinerary stacking | **TripIt** | Clean stack of past/current/future trips |
| Empty state with quick-start | **Todoist** | "Add your first task" with templates |
| Condition cards with status | **Any.do** | Daily planner with task status icons |
| Multi-visa switching | **Flighty** | Tap to switch between tracked flights |

---

## 3. F2 — Points Calculator

### 3.1 Surface: Configure

The Points Calculator is a **Configure** surface: users set things up through progressive disclosure. A wizard structure with clear step indicators, validation at each step, and no surprising jumps. Low decoration; clarity is everything.

### 3.2 Screen-by-Screen Flow

#### Flow Map

```
Landing → Step 1 (Age) → Step 2 (English) 
       → Step 3 (Work Exp - Overseas) → Step 4 (Work Exp - Australia)
       → Step 5 (Education) → Step 6 (Specialist Education)
       → Step 7 (Australian Study) → Step 8 (Regional Study)
       → Step 9 (Partner Skills) → Step 10 (Professional Year)
       → Step 11 (NAATI) → Step 12 (Nomination/Sponsorship)
       → Results → Save/Share → Comparison View
```

#### 3.2.1 Landing Screen (PC0)

```
┌────────────────────────────────────────┐
│  Saathi                          EN|ने │
├────────────────────────────────────────┤
│                                        │
│        📊  Points Calculator           │
│                                        │
│   Find out your Skilled Migration      │
│   points score in under 3 minutes.     │
│                                        │
│   ┌────────────────────────────────┐   │
│   │ ✅ Based on Dept. of Home      │   │
│   │    Affairs points test         │   │
│   │ ✅ Bilingual — English + नेपाली │   │
│   │ ✅ Save & compare results      │   │
│   │ ✅ "How to improve" tips       │   │
│   └────────────────────────────────┘   │
│                                        │
│   ⏱  ~3 minutes · 12 questions        │
│                                        │
│      ┌──────────────────────────┐      │
│      │   Start Calculation →    │      │
│      └──────────────────────────┘      │
│                                        │
│   📋 Source: immi.homeaffairs.gov.au   │
│      Points test — Verified Aug 2026  │
│                                        │
├────────────────────────────────────────┤
│  🛂 Visa  📊 Points  📋 Docs  📝 Help │
└────────────────────────────────────────┘
```

**Pattern reference:** MyFitnessPal onboarding wizard, TaxBee filing flow

#### 3.2.2 Wizard Step Design Pattern (PC1–PC12)

Each step follows an identical pattern for consistency:

```
┌────────────────────────────────────────┐
│  ← Back    Points Calculator   3 of 12 │
├────────────────────────────────────────┤
│  ████████░░░░░░░░░░░░░░  25%          │
│                                        │
│  ┌──────────────────────────────────┐  │
│  │ 🇬🇧 English                      │  │
│  │ How many years of skilled        │  │
│  │ employment have you completed    │  │
│  │ outside Australia?               │  │
│  └──────────────────────────────────┘  │
│  ┌──────────────────────────────────┐  │
│  │ 🇳🇵 नेपाली                        │  │
│  │ तपाईंले अष्ट्रेलिया बाहिर कति    │  │
│  │ वर्षको दक्ष रोजगारी पूरा गर्नुभएको│  │
│  │ छ?                               │  │
│  └──────────────────────────────────┘  │
│                                        │
│  ┌────────────────────────────────┐    │
│  │ ○ Less than 3 years     (0 pts)│    │
│  │ ● 3-4 years             (5 pts)│    │
│  │ ○ 5-7 years            (10 pts)│    │
│  │ ○ 8+ years             (15 pts)│    │
│  └────────────────────────────────┘    │
│                                        │
│  💡 Tip: Skilled employment means      │
│     work in your nominated occupation  │
│     or a closely related field.        │
│                                        │
│  Running total: 20 points              │
│                                        │
│       [ ← Previous ]  [ Next → ]       │
│                                        │
│  📋 DISCLAIMER: This is an estimate    │
│     only. Final points determined by   │
│     a case officer at time of          │
│     invitation.                        │
│                                        │
└────────────────────────────────────────┘
```

**Key Design Decisions:**

1. **Bilingual display: BOTH languages always visible** — not a toggle. Each question shows English + Nepali side by side. This serves two user groups:
   - Nepali-dominant users who need Nepali to understand
   - English-dominant users who want to see both (especially helpful when discussing with family)

2. **Radio buttons over dropdowns** — fewer taps, all options visible for comparison. The points value for each option is shown inline (helps users strategize).

3. **Running total** at the bottom — gives immediate feedback and gamification hook. Updates with subtle animation.

4. **Tip section** — contextual help. Collapsible by default, expand on tap.

5. **Disclaimer: ALWAYS visible** — fixed at the bottom of every step, not hidden in a footer:
   > *"This is an estimate only. Final points determined by a case officer at time of invitation."*

6. **Progress bar** at top — thin bar with percentage. Not a stepped indicator (too many steps for dots).

#### 3.2.3 All Wizard Steps Detail

| Step | Question | Input Type | Points Range |
|------|----------|------------|-------------|
| 1 | Age at time of invitation | Radio: 18-24 (25), 25-32 (30), 33-39 (25), 40-44 (15), 45+ (0) | 0–30 |
| 2 | English language ability | Radio: Superior/IELTS 8+ (20), Proficient/IELTS 7+ (10), Competent/IELTS 6+ (0) | 0–20 |
| 3 | Skilled employment — outside Australia | Radio + numeric: <3y (0), 3-4y (5), 5-7y (10), 8+y (15) | 0–15 |
| 4 | Skilled employment — in Australia | Radio + numeric: <1y (0), 1-2y (5), 3-4y (10), 5-7y (15), 8+y (20) | 0–20 |
| 5 | Educational qualification | Radio: Doctorate (20), Bachelor+ (15), Diploma/Trade (10), Other (0) | 0–20 |
| 6 | Specialist education (STEM/ICT) | Radio: Yes (10), No (0) — conditional on Step 5 | 0–10 |
| 7 | Australian study requirement | Radio: Yes — 2+ years (5), No (0) | 0–5 |
| 8 | Study in regional Australia | Radio: Yes (5), No (0) — conditional on Step 7 | 0–5 |
| 9 | Credentialled community language (NAATI) | Radio: Yes (5), No (0) | 0–5 |
| 10 | Professional Year in Australia | Radio: Yes (5), No (0) | 0–5 |
| 11 | Partner skills | Radio: Single/Partner is AU citizen (10), Partner has skills (10), Partner has Competent English (5), None (0) | 0–10 |
| 12 | Nomination/Sponsorship | Radio: State nomination 190 (5), State/Regional 491 (15), None (0) | 0–15 |

**Conditional logic:** 
- Step 7 appears only if Step 5 has qualifying education
- Step 8 appears only if Step 7 = Yes
- Step 10 appears only if Step 6 = Yes

#### 3.2.4 Results Screen (PC-Results)

```
┌────────────────────────────────────────┐
│  ← Back      Your Results              │
├────────────────────────────────────────┤
│                                        │
│          ┌─────────────────┐           │
│          │                 │           │
│          │      75         │           │
│          │    POINTS       │           │
│          │                 │           │
│          │  of 100 possible│           │
│          │  (65 minimum)   │           │
│          └─────────────────┘           │
│                                        │
│  ████████████████████░░░░  75%        │
│                                        │
│  ┌──────────────────────────────────┐  │
│  │ ✅ You're above the 65-point     │  │
│  │    minimum. You may be eligible   │  │
│  │    for:                          │  │
│  │    • Skilled Independent (189)   │  │
│  │    • Skilled Nominated (190)     │  │
│  │    • Skilled Regional (491)      │  │
│  └──────────────────────────────────┘  │
│                                        │
│  Breakdown                             │
│  ┌──────────────────────────────────┐  │
│  │ Age (32)                    30   │  │
│  │ English (IELTS 7+)          10   │  │
│  │ Overseas work exp (5y)      10   │  │
│  │ Australian work exp (2y)     5   │  │
│  │ Education (Bachelor)        15   │  │
│  │ Australian study             5   │  │
│  │ ─────────────────────────       │  │
│  │ TOTAL                       75   │  │
│  └──────────────────────────────────┘  │
│                                        │
│  📊 SkillSelect Comparison              │
│  ┌──────────────────────────────────┐  │
│  │ Your score: 75                   │  │
│  │ 189 latest invite: 80+ (Jul '26) │  │
│  │ 190 latest invite: 75+ (Jul '26) │  │
│  │ 491 latest invite: 65+ (Jul '26) │  │
│  │                                  │  │
│  │ ⚠️ 189 may be competitive at     │  │
│  │     your score. Consider 190/491.│  │
│  └──────────────────────────────────┘  │
│                                        │
│  💡 How to Improve                     │
│  ┌──────────────────────────────────┐  │
│  │ • Improve English: IELTS 8+ →    │  │
│  │   +10 more points                │  │
│  │ • Gain 1 more year AU experience:│  │
│  │   +5 more points                 │  │
│  │ • NAATI credential: +5 points    │  │
│  └──────────────────────────────────┘  │
│                                        │
│  ┌──────────────────────────────┐      │
│  │  💾 Save Result              │      │
│  └──────────────────────────────┘      │
│  ┌──────────────────────────────┐      │
│  │  📤 Share as Image           │      │
│  └──────────────────────────────┘      │
│  ┌──────────────────────────────┐      │
│  │  🔄 Recalculate              │      │
│  └──────────────────────────────┘      │
│                                        │
│  📋 DISCLAIMER: This is an estimate    │
│     only. Final points determined by   │
│     a case officer.                    │
│                                        │
└────────────────────────────────────────┘
```

**Results screen details:**

- **Large numeric score** — the hero of this screen. Colored: green (above threshold), amber (borderline), red (below minimum).
- **Gauge bar** — visual representation of 75/100 with threshold marker at 65
- **Eligibility verdict** — plain-language: "You may be eligible for..."
- **Breakdown table** — each row tappable to go back and edit that answer
- **SkillSelect comparison** — fetched from latest invitation round data. Shows what scores are currently getting invited. Critical context.
- **"How to Improve"** — calculated gap analysis. Shows exactly which answers to change for maximum point gain.
- **Save/Share/Recalculate** — three CTAs at the bottom

#### 3.2.5 Saved Results & Comparison (PC-Saved)

```
┌────────────────────────────────────────┐
│  ← Results    Saved Calculations        │
├────────────────────────────────────────┤
│                                        │
│  ┌──────────────────────────────────┐  │
│  │ 26 July 2026              75 pts │  │
│  │ Skilled Independent (189) path    │  │
│  │ "After 1 more year AU exp"   >   │  │
│  └──────────────────────────────────┘  │
│                                        │
│  ┌──────────────────────────────────┐  │
│  │ 15 June 2026              65 pts │  │
│  │ State Nomination (190) path       │  │
│  │ "First calculation"          >   │  │
│  └──────────────────────────────────┘  │
│                                        │
│  ┌──────────────────────────────────┐  │
│  │ 10 May 2026               60 pts │  │
│  │ Below threshold                  │  │
│  │ "Before English improvement" >   │  │
│  └──────────────────────────────────┘  │
│                                        │
│  [ Compare Selected ]  [ New Calc ]    │
│                                        │
└────────────────────────────────────────┘
```

**Pattern reference:** MyFitnessPal weight tracking — saved entries with trends. Duolingo progress tracking.

#### 3.2.6 Share as Image

Tapping "Share as Image" generates a clean card:

```
┌────────────────────────────────┐
│       Saathi Points            │
│                                │
│          75 POINTS             │
│                                │
│  Age (32)              30     │
│  English (IELTS 7+)    10     │
│  Overseas work (5y)    10     │
│  AU work (2y)           5     │
│  Education (Bachelor)  15     │
│  AU study               5     │
│                                │
│  Eligible for: 189/190/491    │
│                                │
│  Calculated: 8 Aug 2026       │
│  saathi.app/share/abc123      │
│                                │
│  ⚠️ Estimate only             │
└────────────────────────────────┘
```

Saved to device as PNG. Share sheet opens natively.

### 3.3 Reference App Patterns for F2

| Pattern | Reference App | Why |
|---------|---------------|-----|
| Step-by-step wizard with progress | **MyFitnessPal onboarding** | Familiar wizard pattern, progress % feels achievable |
| Radio buttons with point values | **TaxBee / H&R Block** | Tax calculators show deduction values inline |
| Results with breakdown + improvement | **Credit Karma** | Score breakdown with "how to improve" suggestions |
| Saved calculations history | **MyFitnessPal weight log** | Timeline of entries with trends |
| Shareable results card | **Spotify Wrapped** | Shareable summary card, designed for social |
| Duolingo-style progress | **Duolingo** | Gamified progress + "what to do next" |

---

## 4. F3 — Document Checklist

### 4.1 Surface: Operate

The Document Checklist is an **Operate** surface: users take action on items. Selection state dominates — "needed," "gathered," "uploaded." Clear affordances for checking off, uploading, and exporting.

### 4.2 Screen-by-Screen Flow

#### Flow Map

```
Visa Type Selection → Branching Questions → Personalized Checklist → Item Detail → Upload
                                                         │
                                                    Export/Print
```

#### 4.2.1 Visa Type Selection (DC0)

```
┌────────────────────────────────────────┐
│  Saathi      Document Checklist  EN|ने │
├────────────────────────────────────────┤
│                                        │
│  What visa are you applying for?       │
│  तपाईं कुन भिसाको लागि आवेदन गर्दै    │
│  हुनुहुन्छ?                            │
│                                        │
│  ┌──────────────────────────────────┐  │
│  │ 🎓                              │  │
│  │ Student Visa (Subclass 500)      │  │
│  │ First student visa or renewal    │  │
│  └──────────────────────────────────┘  │
│  ┌──────────────────────────────────┐  │
│  │ 🎓                              │  │
│  │ Graduate Visa (Subclass 485)     │  │
│  │ Post-study work rights           │  │
│  └──────────────────────────────────┘  │
│  ┌──────────────────────────────────┐  │
│  │ 💼                              │  │
│  │ Skilled Independent (189)        │  │
│  │ Points-tested permanent visa     │  │
│  └──────────────────────────────────┘  │
│  ┌──────────────────────────────────┐  │
│  │ 💼                              │  │
│  │ Skilled Nominated (190)          │  │
│  │ State/territory nomination       │  │
│  └──────────────────────────────────┘  │
│  ┌──────────────────────────────────┐  │
│  │ 🏘️                              │  │
│  │ Skilled Regional (491)           │  │
│  │ Regional provisional visa        │  │
│  └──────────────────────────────────┘  │
│  ┌──────────────────────────────────┐  │
│  │ 👨‍👩‍👧                              │  │
│  │ Partner Visa (820/801)           │  │
│  │ Onshore partner migration        │  │
│  └──────────────────────────────────┘  │
│                                        │
├────────────────────────────────────────┤
│  🛂 Visa  📊 Points  📋 Docs  📝 Help │
└────────────────────────────────────────┘
```

**Design: Cards over dropdown** — Cards with icons are more scannable, accessible (larger touch targets), and informative (shows description text). Six cards fit comfortably on mobile.

**Pattern reference:** Airbnb booking wizard (experience type cards), Monzo bank account selection

#### 4.2.2 Branching Questionnaire (DC1–DC4)

After selecting visa type, 3–5 contextual questions refine the checklist:

**Example for Student Visa (500):**

```
┌────────────────────────────────────────┐
│  ← Type     Student Visa        1 of 4 │
├────────────────────────────────────────┤
│                                        │
│  Are you applying from inside or       │
│  outside Australia?                    │
│                                        │
│  ┌──────────────────────────────────┐  │
│  │ 🏠 Inside Australia (onshore)    │  │
│  │ Applying while on current visa   │  │
│  └──────────────────────────────────┘  │
│  ┌──────────────────────────────────┐  │
│  │ ✈️ Outside Australia (offshore)  │  │
│  │ Applying from Nepal or elsewhere │  │
│  └──────────────────────────────────┘  │
│                                        │
│          [ Continue → ]                │
│                                        │
└────────────────────────────────────────┘
```

**Branching questions vary by visa type:**

| Visa Type | Question 1 | Question 2 | Question 3 | Question 4 |
|-----------|------------|------------|------------|------------|
| 500 | Onshore/Offshore? | With dependents? | Under 18? | Course level? |
| 485 | Post-study or Graduate Work? | With dependents? | Have AFP check? | — |
| 189/190/491 | Nominated occupation? | Skills assessed? | With dependents? | Health check done? |
| Partner | Married or De facto? | Onshore/Offshore? | Together > 3 years? | Have children together? |

#### 4.2.3 Personalized Checklist (DC-Checklist)

```
┌────────────────────────────────────────┐
│  ← Type   Student Visa Checklist       │
├────────────────────────────────────────┤
│                                        │
│  ┌──────────────────────────────────┐  │
│  │ ████████████░░░░░░  3 of 12     │  │
│  │ documents gathered               │  │
│  └──────────────────────────────────┘  │
│                                        │
│  Filter: [All] [Needed] [Gathered]     │
│                                        │
│  ━━ ESSENTIAL DOCUMENTS ━━━━━━━━━━━━  │
│                                        │
│  ✅ Confirmation of Enrolment (CoE)    │
│     ┌──────────────────────────────┐   │
│     │ Uploaded: coe_2026.pdf  [👁] │   │
│     └──────────────────────────────┘   │
│                                        │
│  ✅ Passport (bio-data page)            │
│     ┌──────────────────────────────┐   │
│     │ Uploaded: passport.pdf  [👁] │   │
│     └──────────────────────────────┘   │
│                                        │
│  ⬜ Genuine Temporary Entrant (GTE)    │
│     ┌──────────────────────────────┐   │
│     │ Write a statement explaining  │   │
│     │ your study intentions.        │   │
│     │                    [Expand ▼] │   │
│     └──────────────────────────────┘   │
│                                        │
│  ━━ FINANCIAL DOCUMENTS ━━━━━━━━━━━━  │
│                                        │
│  ⬜ Bank statements (last 3 months)     │
│     ┌──────────────────────────────┐   │
│     │ Show funds for tuition +      │   │
│     │ living costs (~AUD $29,710/yr)│   │
│     │                    [Expand ▼] │   │
│     └──────────────────────────────┘   │
│                                        │
│  ⬜ Financial capacity declaration     │
│  ⬜ Scholarship letter (if applicable) │
│                                        │
│  ━━ HEALTH & CHARACTER ━━━━━━━━━━━━━  │
│                                        │
│  ⬜ OSHC health insurance certificate  │
│  ⬜ Health examination (if required)   │
│  ⬜ AFP Police Check (if > 16)         │
│                                        │
│  ━━ ENGLISH & ACADEMIC ━━━━━━━━━━━━━  │
│                                        │
│  ✅ IELTS/PTE Score Report             │
│  ⬜ Academic transcripts               │
│  ⬜ Previous qualifications            │
│                                        │
│  ┌──────────────────────────────────┐  │
│  │ 🖨️  Print Checklist              │  │
│  └──────────────────────────────────┘  │
│  ┌──────────────────────────────────┐  │
│  │ 📤 Export as PDF                 │  │
│  └──────────────────────────────────┘  │
│                                        │
│  📋 Source: immi.homeaffairs.gov.au    │
│     Student Visa document checklist    │
│     Verified August 2026              │
│                                        │
└────────────────────────────────────────┘
```

**Checklist item states:**
- `⬜` Needed — not yet gathered
- `🔄` In progress — partially gathered
- `✅` Gathered — uploaded/confirmed
- `⚠️` Attention — expired or incorrect

**Filter tabs** for quick scanning: All | Needed | Gathered

#### 4.2.4 Document Item Detail (Expanded Accordion)

Tapping `[Expand ▼]` on any checklist item:

```
┌────────────────────────────────────────┐
│  Genuine Temporary Entrant (GTE)        │
├────────────────────────────────────────┤
│                                        │
│  ┌──────────────────────────────────┐  │
│  │ 🇬🇧 What is this?                │  │
│  │ A personal statement (300-500    │  │
│  │ words) explaining why you chose  │  │
│  │ to study in Australia, your      │  │
│  │ circumstances in Nepal, and your │  │
│  │ intention to return after study. │  │
│  └──────────────────────────────────┘  │
│                                        │
│  ┌──────────────────────────────────┐  │
│  │ 🇳🇵 यो के हो?                    │  │
│  │ तपाईंले किन अष्ट्रेलियामा अध्ययन │  │
│  │ गर्न रोज्नुभयो, नेपालमा तपाईंको  │  │
│  │ अवस्था, र अध्ययनपछि फर्कने इरादा │  │
│  │ बताउने व्यक्तिगत विवरण (३००-५००  │  │
│  │ शब्दहरू)।                        │  │
│  └──────────────────────────────────┘  │
│                                        │
│  ┌──────────────────────────────────┐  │
│  │ 📋 How to get this document      │  │
│  │ 1. Write in your own words       │  │
│  │ 2. Keep it genuine — don't copy  │  │
│  │ 3. Include: your background,     │  │
│  │    course choice reasons, ties   │  │
│  │    to Nepal, future plans        │  │
│  │ 4. AI writing assistant → [Try] │  │
│  └──────────────────────────────────┘  │
│                                        │
│  ┌──────────────────────────────────┐  │
│  │ ⚠️ Common Mistakes               │  │
│  │ • Using a template — case        │  │
│  │   officers can tell              │  │
│  │ • Not addressing the "temporary" │  │
│  │   requirement directly           │  │
│  │ • Copying from friends — unique  │  │
│  │   circumstances matter           │  │
│  └──────────────────────────────────┘  │
│                                        │
│  Status: ⬜ Not yet written            │
│                                        │
│  ┌──────────────────────────────────┐  │
│  │ ✍️  Start Writing (AI Assist)    │  │
│  └──────────────────────────────────┘  │
│  ┌──────────────────────────────────┐  │
│  │ 📄 Upload Completed Document     │  │
│  └──────────────────────────────────┘  │
│                                        │
│  📋 Source: immi.homeaffairs.gov.au    │
│                                        │
└────────────────────────────────────────┘
```

**Accordion sections:**
1. **What is this?** — plain-language explanation (bilingual)
2. **How to get this document** — step-by-step instructions + AI writing assistant for statements
3. **Common Mistakes** — community-sourced and official guidance
4. **Status + Actions** — current state + next action CTA

#### 4.2.5 Upload Flow (DC-Upload)

```
┌────────────────────────────────────────┐
│  ← Checklist    Upload Document        │
├────────────────────────────────────────┤
│                                        │
│  Passport — Bio-data Page              │
│                                        │
│  ┌──────────────────────────────────┐  │
│  │                                  │  │
│  │       📄  Tap to Upload          │  │
│  │                                  │  │
│  │    Camera · Files · Gallery      │  │
│  │                                  │  │
│  └──────────────────────────────────┘  │
│                                        │
│  Accepted: PDF, JPG, PNG               │
│  Max size: 10MB                        │
│                                        │
│  🔒 Processed on-device only           │
│                                        │
└────────────────────────────────────────┘
```

After upload:
```
┌────────────────────────────────────────┐
│  ← Checklist    Upload Complete         │
├────────────────────────────────────────┤
│                                        │
│  ✅ passport.pdf uploaded              │
│                                        │
│  ┌──────────────────────────────────┐  │
│  │ [Thumbnail preview]              │  │
│  └──────────────────────────────────┘  │
│                                        │
│  Expiry date detected: 15 Mar 2029     │
│  ⚠️ Valid for this application        │
│                                        │
│  [ View Document ]  [ Replace ]        │
│                                        │
│  ✓ Document added to checklist         │
│                                        │
└────────────────────────────────────────┘
```

**Intelligent detection:** For passports, auto-extract expiry date and validate it covers the visa period. For CoEs, extract course dates. Visual feedback: green checkmark with confirmation.

#### 4.2.6 Export/Print Flow

```
┌────────────────────────────────────────┐
│  Export Checklist                       │
├────────────────────────────────────────┤
│                                        │
│  Format:                               │
│  ● PDF (for printing)                 │
│  ○ Simple text list                    │
│                                        │
│  Include:                              │
│  ☑ Gathered items                      │
│  ☑ Needed items                        │
│  ☑ Tips & common mistakes              │
│  ☐ Uploaded document previews         │
│  ☑ Nepali translations                 │
│                                        │
│     [ Generate & Share → ]            │
│                                        │
└────────────────────────────────────────┘
```

**Export generates a clean PDF:** Saathi-branded header, checklist with status icons, tips section, source citations. Share sheet for WhatsApp, email, print.

### 4.3 Reference App Patterns for F3

| Pattern | Reference App | Why |
|---------|---------------|-----|
| Visa type cards with icons | **Airbnb experience selection** | Card-based selection over dropdown |
| Branching questionnaire | **Typeform** | One question at a time, conditional logic |
| Checklist with status icons | **Notion task lists** | Status states, collapsible details |
| Accordion detail for each item | **myGov document center** | Government service pattern — what/why/how |
| "Common mistakes" section | **TaxBee / H&R Block** | Tax software shows common errors per field |
| Upload with on-device processing | **Apple Notes scan docs** | Privacy-first, local processing |
| Export checklist as PDF | **Service NSW app** | Government checklist → PDF pattern |

---

## 5. F4 — Form Helper

### 5.1 Surface: Configure + Inspect

The Form Helper is a hybrid: **Configure** (selecting forms, toggling language) + **Inspect** (drilling into individual field explanations). Also has an **Operate** component for the auto-fill/upload flow.

### 5.2 Screen-by-Screen Flow

#### Flow Map

```
Form Selection → Form Overview → Field-by-Field Explainer (Chat Layout)
                                     │
                                Document Upload → Auto-Fill Pipeline → Review → Export
```

#### 5.2.1 Form Selection (FH0)

```
┌────────────────────────────────────────┐
│  Saathi         Form Helper      EN|ने │
├────────────────────────────────────────┤
│                                        │
│  Which immigration form do you need?   │
│                                        │
│  🔍 Search forms...                    │
│                                        │
│  Frequently Used                        │
│  ┌──────────────────────────────────┐  │
│  │ 📝 Form 80                       │  │
│  │ Personal particulars for         │  │
│  │ character assessment              │  │
│  │ 30+ fields  ·  45 min to complete │  │
│  └──────────────────────────────────┘  │
│  ┌──────────────────────────────────┐  │
│  │ 📝 Form 1221                     │  │
│  │ Additional personal particulars   │  │
│  │ 25+ fields  ·  35 min            │  │
│  └──────────────────────────────────┘  │
│  ┌──────────────────────────────────┐  │
│  │ 📝 Form 956                      │  │
│  │ Appointment of migration agent    │  │
│  │ 10 fields  ·  15 min             │  │
│  └──────────────────────────────────┘  │
│                                        │
│  All Forms                              │
│  ┌──────────────────────────────────┐  │
│  │ Form 80 — Character Assessment   │  │
│  │ Form 1221 — Additional Info      │  │
│  │ Form 956 — Agent Appointment     │  │
│  │ Form 956A — Agent Withdrawal     │  │
│  │ Form 929 — Change of Address     │  │
│  │ Form 1023 — Notification of      │  │
│  │   incorrect answers              │  │
│  │ Form 1022 — Change of            │  │
│  │   circumstances                   │  │
│  │ Form 1442i — Privacy notice      │  │
│  │ Form 1529 — Consent for child    │  │
│  └──────────────────────────────────┘  │
│                                        │
├────────────────────────────────────────┤
│  🛂 Visa  📊 Points  📋 Docs  📝 Help │
└────────────────────────────────────────┘
```

#### 5.2.2 Form Overview (FH1)

After selecting a form:

```
┌────────────────────────────────────────┐
│  ← Forms       Form 80 Overview        │
├────────────────────────────────────────┤
│                                        │
│  Form 80                                │
│  Personal Particulars for Character    │
│  Assessment                             │
│                                        │
│  ┌──────────────────────────────────┐  │
│  │ ⏱  ~45 min to complete           │  │
│  │ 📋 30+ fields                    │  │
│  │ 🏛️  Required by: Dept. of        │  │
│  │    Home Affairs                   │  │
│  │ 🔄 Updated: January 2025         │  │
│  └──────────────────────────────────┘  │
│                                        │
│  What you'll need:                     │
│  ┌──────────────────────────────────┐  │
│  │ • Passport details                │  │
│  │ • Travel history (10 years)       │  │
│  │ • Address history (10 years)      │  │
│  │ • Employment history              │  │
│  │ • Education history               │  │
│  │ • Family details (parents,        │  │
│  │   siblings, children, partner)    │  │
│  │ • Military service details        │  │
│  │ • Visa refusal history            │  │
│  └──────────────────────────────────┘  │
│                                        │
│  ┌──────────────────────────────────┐  │
│  │ 📄 Start Field-by-Field Guide    │  │
│  └──────────────────────────────────┘  │
│  ┌──────────────────────────────────┐  │
│  │ 📸 Upload Form → Auto-fill       │  │
│  └──────────────────────────────────┘  │
│  ┌──────────────────────────────────┐  │
│  │ 🌐 Open Official Form (PDF)      │  │
│  └──────────────────────────────────┘  │
│                                        │
│  📋 Source: immi.homeaffairs.gov.au    │
│     Form 80 — Verified June 2026      │
│                                        │
└────────────────────────────────────────┘
```

#### 5.2.3 Field-by-Field Explainer — Chat Layout (FH-Chat)

**Design decision:** Chat-like interface over form-like layout. Rationale:
- Users read explanations sequentially, field by field
- Chat format feels more personal/guided than a dense form
- Natural place for bilingual toggle
- Each message = one explanation + one action
- Familiar pattern from WhatsApp/Viber (dominant apps for target users)

```
┌────────────────────────────────────────┐
│  ← Overview   Form 80 Guide     EN|ने  │
├────────────────────────────────────────┤
│                                        │
│  ┌──────────────────────────────────┐  │
│  │ 🤖 Saathi                        │  │
│  │                                  │  │
│  │ Let's go through Form 80         │  │
│  │ field by field. I'll explain     │  │
│  │ what each section means and      │  │
│  │ what you need to fill in.        │  │
│  │                                  │  │
│  │ Use the toggle above to switch   │  │
│  │ between English and Nepali.      │  │
│  │                                  │  │
│  │ Ready? Let's start with Part A.  │  │
│  └──────────────────────────────────┘  │
│                                        │
│  Part A — Your Details                  │
│  ┌──────────────────────────────────┐  │
│  │ Question 1: Your full name       │  │
│  │                                  │  │
│  │ Write your name exactly as it    │  │
│  │ appears on your passport.        │  │
│  │ • Family name / Surname first    │  │
│  │ • Given names as on passport     │  │
│  │                                  │  │
│  │ 🇳🇵 नेपाली: राहदानीमा लेखिए अनुसार│  │
│  │ आफ्नो नाम लेख्नुहोस्।            │  │
│  │                                  │  │
│  │ ⚠️ Common error: Don't add       │  │
│  │    middle names if not on        │  │
│  │    passport.                     │  │
│  │                                  │  │
│  │ 📋 Source: Form 80 instructions  │  │
│  │    immi.gov.au/form-80           │  │
│  │    Verified June 2026            │  │
│  └──────────────────────────────────┘  │
│                                        │
│  Question 1                            │
│  ┌──────────────────────────────────┐  │
│  │ Family Name:  [               ]  │  │
│  │ Given Names:  [               ]  │  │
│  └──────────────────────────────────┘  │
│                                        │
│  ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─  │
│                                        │
│  ┌──────────────────────────────────┐  │
│  │ 🤖 Saathi                        │  │
│  │                                  │  │
│  │ Question 2: Date of birth        │  │
│  │ Use DD/MM/YYYY format            │  │
│  │                                  │  │
│  │ 🇳🇵 नेपाली: जन्म मिति DD/MM/YYYY  │  │
│  │ ढाँचामा लेख्नुहोस्।               │  │
│  │                                  │  │
│  │ 📋 Source: Form 80, Part A, Q2   │  │
│  │    Verified June 2026            │  │
│  └──────────────────────────────────┘  │
│                                        │
│  ┌──────────────────────────────────┐  │
│  │  ◀ Prev Field    Next Field ▶    │  │
│  └──────────────────────────────────┘  │
│                                        │
│  Progress: Q2 of 30+  ·  Part A     │
│                                        │
└────────────────────────────────────────┘
```

**Key chat layout features:**

1. **AI message bubbles** for explanations (left-aligned, blue-50 background)
2. **Inline bilingual:** Nepali under each explanation (tappable language toggle switches which language appears first)
3. **Per-field citation:** Every explanation cites its source
4. **Input fields inline** — user can type directly in the chat (saved locally)
5. **Navigation:** Previous/Next buttons at bottom + jump-to-field menu
6. **Common error block:** Each field shows common mistakes specific to Nepali applicants

**Bilingual toggle behavior:**
```
┌──────────────────────────────────┐
│ 🇬🇧 English Only                 │
│ 🇳🇵 नेपाली मात्र                   │
│ 📱 Side-by-side                  │
└──────────────────────────────────┘
```
A three-position toggle in the chat header. "Side-by-side" shows English first, Nepali in a collapsible block below.

#### 5.2.4 Jump-to-Field Navigation (FH-Nav)

Accessible via a field-list icon in the header:

```
┌────────────────────────────────────────┐
│  ← Chat     Jump to Field              │
├────────────────────────────────────────┤
│                                        │
│  🔍 Filter fields...                   │
│                                        │
│  Part A — Your Details                 │
│  ○ Q1 — Full name                      │
│  ○ Q2 — Date of birth                  │
│  ○ Q3 — Place of birth                 │
│  ○ Q4 — Citizenship                    │
│                                        │
│  Part B — Contact Details              │
│  ○ Q5 — Current address                │
│  ○ Q6 — Phone number                   │
│  ○ Q7 — Email                          │
│                                        │
│  Part C — Travel History               │
│  ○ Q8-Q15 — Countries visited          │
│                                        │
│  Part D — Employment History           │
│  ○ Q16-Q22 — Previous employers        │
│                                        │
│  ...                                    │
│                                        │
└────────────────────────────────────────┘
```

Tapping a field scrolls the chat to that position with a smooth animation.

#### 5.2.5 Document Upload → Auto-Fill Pipeline (FH-Autofill)

```
┌────────────────────────────────────────┐
│  ← Overview   Upload & Auto-fill       │
├────────────────────────────────────────┤
│                                        │
│  Upload your documents and I'll        │
│  extract the information to pre-fill   │
│  your Form 80.                         │
│                                        │
│  ┌──────────────────────────────────┐  │
│  │ 📄 Passport (bio-data page)      │  │
│  │    ┌────────────────────────┐    │  │
│  │    │ Uploaded ✓              │    │  │
│  │    │ Extracted: 12 fields    │    │  │
│  │    │                 [👁 →]  │    │  │
│  │    └────────────────────────┘    │  │
│  └──────────────────────────────────┘  │
│                                        │
│  ┌──────────────────────────────────┐  │
│  │ 📄 Birth Certificate             │  │
│  │    ┌────────────────────────┐    │  │
│  │    │ [Tap to Upload]         │    │  │
│  │    └────────────────────────┘    │  │
│  └──────────────────────────────────┘  │
│                                        │
│  ┌──────────────────────────────────┐  │
│  │ 📄 Current Visa Grant Notice     │  │
│  │    ┌────────────────────────┐    │  │
│  │    │ [Tap to Upload]         │    │  │
│  │    └────────────────────────┘    │  │
│  └──────────────────────────────────┘  │
│                                        │
│  🔒 All processing is on-device.       │
│     Documents never leave your phone.  │
│                                        │
│  ┌──────────────────────────────────┐  │
│  │ 📊 Review Extracted Data         │  │
│  └──────────────────────────────────┘  │
│                                        │
└────────────────────────────────────────┘
```

#### 5.2.6 Review Extracted Data — Side-by-Side (FH-Review)

```
┌────────────────────────────────────────┐
│  ← Upload    Review Extractions        │
├────────────────────────────────────────┤
│                                        │
│  Confidence: 🟢 High  🟡 Medium  🔴 Low │
│                                        │
│  ━━ PASSPORT — Page 1 ━━━━━━━━━━━━━━  │
│                                        │
│  Family Name                           │
│  ┌──────────────┐  ┌────────────────┐  │
│  │ ADHIKARI     │  │  ADHIKARI     │  │
│  │              │  │                │  │
│  │ Extracted    │  │ Source snippet │  │
│  │ ✅ 🟢 High   │  │ [image crop]   │  │
│  └──────────────┘  └────────────────┘  │
│                                        │
│  Given Names                           │
│  ┌──────────────┐  ┌────────────────┐  │
│  │ PRABIN       │  │  PRABIN       │  │
│  │              │  │                │  │
│  │ Extracted    │  │ Source snippet │  │
│  │ ✅ 🟢 High   │  │ [image crop]   │  │
│  └──────────────┘  └────────────────┘  │
│                                        │
│  Date of Birth                         │
│  ┌──────────────┐  ┌────────────────┐  │
│  │ 15/03/19??   │  │  15 MAR 19    │  │
│  │              │  │  [last 2       │  │
│  │ Extracted    │  │   digits blur] │  │
│  │ ⚠️ 🟡 Medium │  │ Source snippet │  │
│  │ [Edit ✏️]     │  │ [image crop]   │  │
│  └──────────────┘  └────────────────┘  │
│                                        │
│  Passport Number                       │
│  ┌──────────────┐  ┌────────────────┐  │
│  │ PA1234567    │  │  PA1234567    │  │
│  │              │  │                │  │
│  │ Extracted    │  │ Source snippet │  │
│  │ ✅ 🟢 High   │  │ [image crop]   │  │
│  └──────────────┘  └────────────────┘  │
│                                        │
│  ━━ BIRTH CERTIFICATE — Page 1 ━━━━━━  │
│                                        │
│  Place of Birth                        │
│  ┌──────────────┐  ┌────────────────┐  │
│  │ — No data —  │  │ काठमाडौँ      │  │
│  │              │  │ (Devanagari)   │  │
│  │ 🔴 Low       │  │ Source snippet │  │
│  │ [Enter ✏️]    │  │ [image crop]   │  │
│  └──────────────┘  └────────────────┘  │
│    ↑ Devanagari OCR is improving        │
│                                        │
│  ┌──────────────────────────────────┐  │
│  │ ✅ Approve All (8 valid fields)  │  │
│  └──────────────────────────────────┘  │
│  ┌──────────────────────────────────┐  │
│  │ 📝 Fix & Continue (2 need input) │  │
│  └──────────────────────────────────┘  │
│                                        │
│  🔒 Review complete. Data stays on     │
│     device until you export.           │
│                                        │
└────────────────────────────────────────┘
```

**Confidence Tier System:**

| Tier | Color | Icon | Meaning | User Action |
|------|-------|------|---------|------------|
| High | 🟢 Green | ✅ | > 95% confidence, clear OCR match | Auto-accepted |
| Medium | 🟡 Amber | ⚠️ | 70-95% confidence, partial match or blur | User reviews, edits inline |
| Low | 🔴 Red | ❌ | < 70% confidence or no data found | User must enter manually |

**Side-by-side layout:**
- Left column: extracted value with confidence indicator
- Right column: zoomed crop of the source document highlighting where the value came from
- Tapping "Edit" opens inline text input
- This pattern builds trust — user can verify extraction against the source

#### 5.2.7 Disclaimer Pattern in F4

Every AI response includes:

```
┌────────────────────────────────────────┐
│                                        │
│  ⚠️ DISCLAIMER                         │
│  This explanation is generated by AI   │
│  and based on Department of Home       │
│  Affairs documentation. Always cross-  │
│  check with the official form          │
│  instructions at                       │
│  immi.homeaffairs.gov.au               │
│                                        │
│  Saathi is not a registered migration  │
│  agent (MARA).                         │
│                                        │
└────────────────────────────────────────┘
```

#### 5.2.8 Export Filled Data

```
┌────────────────────────────────────────┐
│  Export Form Data                       │
├────────────────────────────────────────┤
│                                        │
│  Export format:                        │
│  ● Fillable PDF (auto-fill the        │
│     official form)                     │
│  ○ CSV / Spreadsheet                   │
│  ○ Print-friendly summary              │
│                                        │
│  Include:                              │
│  ☑ Filled field data                   │
│  ☑ Source citations                    │
│  ☐ AI confidence scores                │
│                                        │
│     [ Generate & Download → ]         │
│                                        │
└────────────────────────────────────────┘
```

### 5.3 Reference App Patterns for F4

| Pattern | Reference App | Why |
|---------|---------------|-----|
| Chat-based guided interface | **Intercom / Crisp chat** | Conversational field-by-field feels more approachable |
| Bilingual explanations | **Duolingo** | Language toggle, side-by-side for learning |
| Side-by-side extraction review | **TurboTax document upload** | Shows extracted value + source snippet |
| Confidence tier colors | **Google Lens text selection** | Green/amber/red confidence indicators |
| Field navigation drawer | **Google Forms sections** | Jump to any field/section |
| Per-field citations | **Wikipedia footnotes** | Every claim cites its source |
| On-device processing privacy | **Apple Photos face recognition** | "Processed on-device" trust signal |

---

## 6. Component Library

### 6.1 Core Components (required for all features)

| Component | Description | Used In |
|-----------|------------|---------|
| `LanguageToggle` | Segmented pill `EN \| ने` | Global app bar |
| `TabBar` | Bottom 4-tab navigation with icons + labels | Global |
| `ProgressBar` | Thin horizontal bar with % and fraction | F2 wizard, F3 checklist |
| `CountdownRing` | SVG circular progress ring with color shift | F1 dashboard |
| `VisaCard` | Card with countdown, conditions, status | F1 dashboard, multi-visa |
| `WizardStep` | Question + options + running total + tip | F2 wizard |
| `ChecklistItem` | Status icon + title + expandable detail | F3 checklist |
| `DocumentUpload` | Drop zone + camera/files picker + preview | F3 upload, F4 upload |
| `ChatBubble` | Left-aligned AI message with source citation | F4 chat |
| `ConfidenceIndicator` | 🟢🟡🔴 badge + percentage | F4 review |
| `Disclaimer` | Fixed footer bar with MARA disclaimer | F2 every step, F4 every response |
| `SourceCitation` | Inline link with "Verified [date]" badge | All features |
| `EmptyState` | Icon + headline + description + CTA | All features |
| `TrustBadge` | "✅ VERIFIED · Updated [date]" pill | F2, F3, F4 |
| `OfflineBanner` | Persistent top banner: "⚠️ Offline — showing cached data" | All features |

### 6.2 Form Components

| Component | Description | States |
|-----------|------------|--------|
| `RadioCard` | Selectable card with icon, title, subtitle, points | Default, Selected, Disabled |
| `DatePicker` | Native date picker with manual fallback | Empty, Filled, Error |
| `TextInput` | Bilingual label, placeholder, validation | Default, Focus, Filled, Error, Disabled |
| `Toggle` | iOS-style switch | Off, On, Disabled |
| `Checkbox` | Square checkbox with label | Unchecked, Checked, Indeterminate |
| `SelectCard` | Full-width selectable card (visa type, form) | Default, Selected, Hover |

### 6.3 Feedback Components

| Component | Description | States |
|-----------|------------|--------|
| `Toast` | Bottom-anchored notification | Success, Error, Warning, Info |
| `Spinner` | Loading indicator | Spinning, With label, Full-screen overlay |
| `Skeleton` | Content placeholder while loading | Card, Text block, List item |
| `ErrorBlock` | Inline error with retry action | Network error, Validation error, Server error |
| `SuccessAnimation` | Checkmark animation on completion | Wizard complete, Upload complete |

---

## 7. Accessibility & WCAG 2.1 AA

### 7.1 Bilingual Accessibility Requirements

1. **Language attributes:** Every text block must have correct `lang` attribute (`lang="en"` or `lang="ne"`) for screen reader pronunciation
2. **Devanagari font sizing:** Minimum 16px for body text in Devanagari to ensure legibility of complex conjuncts
3. **Line height:** 1.7 minimum for Devanagari text (accommodates matras above and below)
4. **No image-only text:** All Nepali text must be actual text, not images
5. **Screen reader testing:** Test with both English and Nepali VoiceOver/TalkBack

### 7.2 Color & Contrast

| Element | Requirement | Implementation |
|---------|------------|---------------|
| Body text | 4.5:1 minimum | saathi-gray-900 (#1A1A2E) on white = 15.3:1 ✅ |
| Large text (18px+) | 3:1 minimum | All countdown numbers and headers > 18px |
| Interactive elements | 3:1 against adjacent | Buttons, links distinguished by more than color (underlines, icons) |
| Focus indicators | 3:1 minimum | Blue-500 2px ring on all interactive elements |
| Error states | Not color-only | Red border + icon + text message |
| Green/red indicators | Not color-only | Always paired with icons (✅, ⚠️, ❌) and text |

### 7.3 Touch & Interaction

| Element | Requirement | Implementation |
|---------|------------|---------------|
| Minimum touch target | 44×44px (WCAG 2.5.5) | All tappable elements ≥ 44px |
| Card tap targets | Full card tappable | Not just the CTA button |
| Spacing between targets | ≥ 8px | Cards have 12px gap minimum |
| Swipe gestures | Alternative available | Carousel has dot navigation as alternative to swipe |
| Focus order | Logical DOM order | Tab through fields in reading order |

### 7.4 Motion & Animation

| Element | Requirement | Implementation |
|---------|------------|---------------|
| `prefers-reduced-motion` | Respect OS setting | Disable all non-essential animations |
| Auto-playing animation | Under 5 seconds or pausable | Countdown ring is static SVG, updates on load |
| Page transitions | ≤ 300ms | Simple fade, no parallax or slide |

### 7.5 Screen Reader (VoiceOver / TalkBack)

- All icons have `aria-label` in current language
- Countdown timer has `aria-label`: "457 days remaining until visa expires March 15 2026"
- Progress bars have `aria-valuenow`, `aria-valuemin`, `aria-valuemax`
- Tab bar has `role="tablist"` with `aria-selected` on active tab
- Chat messages have `role="log"` for live region announcements
- Confidence indicators have text alternatives, not just color

### 7.6 Offline Accessibility

- All text content cached via service worker
- Saved visa information and checklist progress available offline
- Offline indicator announced to screen readers
- Stale data timestamp always visible

---

## 8. Reference App Patterns Summary

### 8.1 Pattern → Saathi Feature Mapping

| Reference App | Pattern Borrowed | Applied To | Why It Works |
|--------------|-----------------|------------|-------------|
| **Flighty** | Countdown timer with color shift, multi-itinerary stacking | F1 — Visa dashboard, multi-visa | Flighty's elegance in showing time-sensitive state is directly applicable to visa expiry tracking |
| **Medisafe / Apple Health** | Escalating medication reminders with dose tracking | F1 — Notification preferences | Medication adherence UX is the closest analogue to visa deadline compliance |
| **Todoist** | Clean project/task organization, empty state pattern, priority levels | F1 — Conditions tracking, F3 — Checklist | Todoist's information hierarchy is the gold standard for action-item management |
| **MyFitnessPal** | Onboarding wizard with progress, saved entry history, streak tracking | F2 — Points wizard, saved results | MyFitnessPal's onboarding wizard is the industry standard for step-by-step personalization |
| **Duolingo** | Gamification, progress visualization, language toggle, streak motivation | F2 — Running score, F4 — Bilingual toggle, Cross-cutting — onboarding | Duolingo's bilingual interface and progression system is the closest analogue to Saathi's dual-language needs |
| **Boundless** | Immigration-specific UX patterns, form guidance, eligibility assessment | F2 — Eligibility verdict, F4 — Field explainer | Boundless is the direct competitor in the immigration space — their UX is proven |
| **Typeform** | One question at a time, conditional branching, progress % | F2 — Wizard, F3 — Branching questions | Typeform's UX research shows 1-question-at-a-time reduces abandonment |
| **TaxBee / H&R Block** | Tax calculator with inline point values, common error warnings per field | F2 — Points display, F4 — Common mistakes | Tax software UX handles complex government forms with plain-language explanations |
| **myGov / Service NSW** | Government document checklist, export as PDF, trust signals | F3 — Document checklist, Export flow | Australian government apps establish the trust and usability patterns users expect from official services |
| **TurboTax** | Document upload → auto-fill with confidence indicators, side-by-side review | F4 — Auto-fill pipeline, Review screen | TurboTax's document ingestion UX is the benchmark for scan-and-fill workflows |
| **Intercom** | Chat-based guided interface, contextual help | F4 — Chat layout field explainer | Conversational interfaces reduce anxiety around complex tasks |
| **Notion** | Checklist with status states, collapsible details, filter tabs | F3 — Checklist display | Notion's checklist UX allows complex task management with simple affordances |
| **WhatsApp / Viber** | Chat interface, message bubbles, media upload | F4 — Chat layout | Familiar messaging UI reduces cognitive load for Nepali users who primarily use these apps |
| **Spotify Wrapped** | Shareable result card designed for social sharing | F2 — Share as Image | Social sharing drives organic growth and community discussion |

### 8.2 Anti-Patterns (What to Avoid)

Based on common mistakes in immigration and fintech apps:

| Anti-Pattern | Why it Fails | Saathi's Approach |
|-------------|-------------|-------------------|
| Long single-page forms | Users abandon; no sense of progress | Wizard with progress bar, one question at a time |
| English-only interface | Excludes primary user base (Nepali-dominant) | Bilingual by default, side-by-side option |
| Hidden disclaimers in footer | Legal risk; user mistrust | Disclaimer always visible, per-screen |
| Complex navigation | Users get lost | Simple 4-tab bottom bar |
| No offline support | Useless when users need it most | Offline-first PWA with cached data |
| Generic card UI with no hierarchy | Everything looks equally important | Monitor surface for dashboard, Configure for wizards |
| No source citations | Users can't verify information | Every claim links to source with date verified |
| AI without confidence indicators | Users can't distinguish reliable from uncertain | 🟢🟡🔴 confidence tiers on all AI output |

---

## Appendix: Key Interaction Flows (State Machines)

### A1 — Visa Tracker State Machine

```
[No Visa] ──Add Visa──→ [Wizard: Step 1]
                            │
                            ▼
                      [Wizard: Step 2] ──Back──→ [Step 1]
                            │
                            ▼
                      [Wizard: Step 3] ──Back──→ [Step 2]
                            │
                            ▼
                      [Wizard: Step 4] ──Back──→ [Step 3]
                            │
                       Save │
                            ▼
                      [Dashboard: Active]
                       │    │        │
                  Edit │    │ Add    │ conditions tap
                       ▼    ▼        ▼
                  [Wizard] [Multi-  [Condition
                   Edit]   Visa     Detail]
                            View
```

### A2 — Points Calculator State Machine

```
[Landing] ──Start──→ [Step 1/12] ──Next──→ [Step 2/12] → ... → [Step 12/12]
                         │                                    │
                    Running Total                           Calculate
                    updates each step                           │
                                                               ▼
                                                         [Results]
                                                       │    │     │
                                                  Save │    │Share │Recalculate
                                                       ▼    ▼     ▼
                                                  [Saved] [Share [Wizard
                                                   List]  Sheet]  Reset]
```

### A3 — Document Checklist State Machine

```
[Visa Type Selection] ──Select──→ [Branch Q1] → [Q2] → [Q3] → [Q4]
                                                               │
                                                               ▼
                                                         [Checklist]
                                                       │    │     │
                                                  Item │    │Filter│Export
                                                  Tap  │    │     │
                                                       ▼    ▼     ▼
                                                  [Detail] [Filtered [PDF/
                                                   View]   View]   Print]
                                                     │
                                                Upload│
                                                     ▼
                                               [Upload Flow]
```

### A4 — Form Helper State Machine

```
[Form Selection] ──Select──→ [Form Overview]
                                 │
                    ┌────────────┼────────────┐
                    ▼            ▼            ▼
              [Chat Guide] [Auto-fill] [Open PDF]
                    │            │
                    │       [Upload Docs]
                    │            │
                    │       [Review Data]
                    │            │
                    └─────┬──────┘
                          ▼
                    [Export Filled Form]
```

---

*Document version: 1.0 — August 2026*  
*Next steps: Interactive prototypes (Figma/HTML), user testing with Nepali diaspora in Australia, iterate based on feedback.*