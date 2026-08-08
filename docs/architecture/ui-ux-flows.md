# Saathi — UI/UX Flow Design

**Version:** 1.1 | **Date:** August 8, 2026 (F4 revised for manaslu integration)

Comprehensive screen-by-screen UX design for all 4 features. Bilingual (EN/NP). Mobile-first PWA.
F1–F3 are Saathi's own deterministic tools; F4 (§7) is a thin consumer of `manaslu`'s
REST+SSE API (separate repo — see §7.0) rather than a pipeline Saathi implements itself.
`ux-design-subagent.md` (archived) has been merged in and superseded by this document.

**Visual companion:** [`diagrams/saathi-screen-designs.html`](../../diagrams/saathi-screen-designs.html)
renders every screen in this spec as an annotated phone-frame mockup (options, states, worked
examples per page); [`diagrams/saathi-ui-mockup.html`](../../diagrams/saathi-ui-mockup.html) is the
interactive 4-tab demo.

---

## Table of Contents

1. [Design System](#1-design-system)
2. [Navigation & App Shell](#2-navigation--app-shell)
3. [Onboarding Flow](#3-onboarding-flow)
4. [F1 — Visa Tracker UX](#4-f1--visa-tracker-ux)
5. [F2 — Points Calculator UX](#5-f2--points-calculator-ux)
6. [F3 — Document Checklist UX](#6-f3--document-checklist-ux)
7. [F4 — Form Helper + Scan UX](#7-f4--form-helper--scan-ux)
8. [Component Patterns](#8-component-patterns)
9. [State Handling](#9-state-handling)
10. [Accessibility](#10-accessibility)
11. [F5 — News, Seminars & Opportunities UX (Phase 2)](#11-f5--news-seminars--opportunities-ux-phase-2)
12. [F6 — Connect to an Agent UX (Phase 2, English-only)](#12-f6--connect-to-an-agent-ux-phase-2-english-only)

---

## 1. Design System

### Typography
```
Primary (English):    Inter (sans-serif) — clean, modern, highly readable
Primary (Nepali):     Mukta (Devanagari) — open-source, pairs well with Inter
Monospace (data):     JetBrains Mono — for passport numbers, dates, codes
```

### Color System
```
Brand Primary:     #DC2626 (red — Nepal flag red)
Brand Secondary:   #1E3A8A (blue — Nepal flag blue border)
Surface:           #FFFFFF / #0F172A (dark mode)
Success (HIGH):    #16A34A (green — confident extraction)
Warning (MEDIUM):  #D97706 (amber — needs review)
Error (LOW):       #DC2626 (red — cannot extract)
Info:              #2563EB (blue)
```

### Component Library
Based on **shadcn/ui** (Radix primitives + Tailwind). Custom components:

| Component | Description | Used In |
|-----------|------------|---------|
| `CountdownRing` | SVG circular progress ring with color shift | F1 dashboard |
| `WizardStep` | Question + options + running total + tip | F2 wizard |
| `ChecklistItem` | Status icon + title + expandable detail | F3 checklist |
| `DocumentUpload` | Drop zone + camera/files picker + preview | F3 upload, F4 upload |
| `ChatBubble` | Left-aligned AI message with source citation | F4 field explainer |
| `ConfidenceIndicator` | 🟢🟡🔴 badge, renders manaslu's tier label (not an invented %) | F4 extraction review |
| `SourceCropViewer` | Zoomed, tappable crop of the source document a value came from | F4 extraction review |
| `VaultPrefillBadge` | "✨ Pre-filled from your saved profile" + provenance note (doc + when) | F4 extraction review |
| `TransliterationPicker` | Devanagari source + ranked candidates + free-text box | F4 transliteration sub-screen |
| `Disclaimer` | Fixed footer bar with MARA disclaimer | F2 every step, F4 every screen |
| `SourceCitation` | Inline link with "Verified [date]" badge | All features |
| `TrustBadge` | AU-region / encrypted / audit-logged pill — never "on-device" | F4 upload |
| `EmptyState` | Icon + headline + description + CTA | All features |
| `OfflineBanner` | Persistent top banner: "⚠️ Offline — showing cached data" | All features |

### Bilingual Pattern
Every UI string uses this pattern:
```typescript
const labels = {
  familyName: {
    en: "Family Name",
    np: "थर"
  }
}
// Usage: <Label primary={labels.familyName.en} secondary={labels.familyName.np} />
```
Language toggle in header: 🇬🇧 EN | 🇳🇵 NP
Default: NP if user's browser language is `ne`; EN otherwise.

---

## 2. Navigation & App Shell

### Bottom Tab Bar (Primary Navigation)
```
┌──────────────────────────────────────────────┐
│  [📋]      [🧮]       [⏱️]       [📝]        │
│ Checklist Calculator Tracker   Form Helper   │
└──────────────────────────────────────────────┘
```
- 4 tabs = 4 features
- Active tab uses brand red
- Badge on Tracker if visa expiring within 90 days

### Top Bar
```
┌──────────────────────────────────────────────┐
│  ☰  Saathi                          🇳🇵/🇬🇧  │
└──────────────────────────────────────────────┘
```
- Hamburger: Settings, About, Legal, Logout
- Language toggle: immediate switch without page reload

---

## 3. Onboarding Flow

### Screen 1: Welcome
```
┌──────────────────────────────────────┐
│                                      │
│         🏔️  साथी / Saathi            │
│                                      │
│    Your AI companion for the         │
│    Australian immigration journey    │
│                                      │
│    तपाईंको अष्ट्रेलिया आप्रवासन      │
│    यात्राको AI साथी                  │
│                                      │
│         [Get Started / सुरु गर्नुहोस्]│
│                                      │
│      ⚠️ Saathi provides information  │
│      only. Not migration advice.     │
│                                      │
└──────────────────────────────────────┘
```

### Screen 2: Feature Discovery (Swipeable Cards)
```
┌──────────────────────────────────────┐
│         ← Swipe through →            │
│                                      │
│  ┌──────────────────────────────┐    │
│  │         ⏱️                    │    │
│  │    Visa Tracker               │    │
│  │    भिसा ट्रयाकर               │    │
│  │                              │    │
│  │  Never miss a deadline.      │    │
│  │  Track expiry, conditions,   │    │
│  │  and next steps.             │    │
│  │                              │    │
│  │  म्याद कहिल्यै नछुटाउनुहोस्  │    │
│  └──────────────────────────────┘    │
│                                      │
│         ● ○ ○ ○  (page indicator)    │
│                                      │
│         [Skip / छोड्नुहोस्]          │
└──────────────────────────────────────┘
```
- 4 swipeable cards (one per feature)
- Each card: icon + English name + Nepali name + one-line value prop in both languages
- Skip button available from card 1

### Screen 3: Quick Setup (Optional)
```
┌──────────────────────────────────────┐
│     Quick Setup / द्रुत सेटअप        │
│                                      │
│  What brings you here?               │
│  तपाईं यहाँ किन आउनुभयो?            │
│                                      │
│  [ ] Tracking my current visa        │
│  [ ] Calculating points for PR       │
│  [ ] Preparing documents for a visa  │
│  [ ] Understanding visa forms        │
│                                      │
│         [Continue / अगाडि बढ्नुहोस्] │
│                                      │
│         [Skip for now]               │
└──────────────────────────────────────┘
```
- Multi-select checkboxes
- Maps to default tab shown after onboarding

---

## 4. F1 — Visa Tracker UX

### 4.1 Empty State (No Visa Added)
```
┌──────────────────────────────────────┐
│         ⏱️  Visa Tracker             │
│                                      │
│         ┌────────────┐               │
│         │   📋       │               │
│         │  No visa   │               │
│         │  added yet │               │
│         │            │               │
│         │ कुनै भिसा  │               │
│         │ थपिएको छैन │               │
│         └────────────┘               │
│                                      │
│    Add your first visa to start      │
│    tracking deadlines and conditions │
│                                      │
│    [ + Add Visa / भिसा थप्नुहोस् ]   │
└──────────────────────────────────────┘
```

### 4.2 Add Visa — Step 1: Select Subclass
```
┌──────────────────────────────────────┐
│  ← Add Visa          Step 1 of 3     │
│                                      │
│  Select your visa subclass           │
│  भिसा उपवर्ग छान्नुहोस्              │
│                                      │
│  ┌────────────────────────────────┐  │
│  │ 🎓 500 — Student               │  │
│  │    विद्यार्थी                   │  │
│  ├────────────────────────────────┤  │
│  │ 🎓 485 — Temporary Graduate    │  │
│  │    अस्थायी स्नातक              │  │
│  ├────────────────────────────────┤  │
│  │ 💼 482 — TSS / Skills in Demand│  │
│  │    सीपको मागमा                 │  │
│  ├────────────────────────────────┤  │
│  │ 🏠 189 — Skilled Independent   │  │
│  │    दक्ष स्वतन्त्र              │  │
│  ├────────────────────────────────┤  │
│  │ 🏠 190 — Skilled Nominated     │  │
│  │    दक्ष मनोनीत                 │  │
│  ├────────────────────────────────┤  │
│  │ 🏠 491 — Skilled Regional      │  │
│  │    दक्ष क्षेत्रीय              │  │
│  ├────────────────────────────────┤  │
│  │ 🔄 Bridging Visa A/B/C/E       │  │
│  │    ब्रिजिङ भिसा                │  │
│  └────────────────────────────────┘  │
│                                      │
│  Don't see your visa? [Other / अन्य] │
└──────────────────────────────────────┘
```

### 4.3 Add Visa — Step 2: Enter Dates
```
┌──────────────────────────────────────┐
│  ← Back              Step 2 of 3     │
│                                      │
│  Subclass 485 — Temporary Graduate   │
│                                      │
│  Grant Date / प्रदान मिति            │
│  ┌────────────────────────────────┐  │
│  │ 📅 15 March 2025               │  │
│  └────────────────────────────────┘  │
│                                      │
│  Expiry Date / म्याद सकिने मिति      │
│  ┌────────────────────────────────┐  │
│  │ 📅 15 March 2027               │  │
│  └────────────────────────────────┘  │
│                                      │
│  ℹ️ Find this on your visa grant     │
│  letter from immi (PDF/email).       │
│  भिसा grant letter मा हेर्नुहोस्।    │
│                                      │
│         [Next / अगाडि]               │
└──────────────────────────────────────┘
```

### 4.4 Add Visa — Step 3: Conditions (Auto-Detected)
```
┌──────────────────────────────────────┐
│  ← Back              Step 3 of 3     │
│                                      │
│  Visa Conditions / भिसा सर्तहरू       │
│                                      │
│  Based on Subclass 485:              │
│                                      │
│  ✅ 8107 — Work limitation           │
│     Unlimited work rights            │
│     असीमित काम गर्न पाइने            │
│                                      │
│  ✅ 8501 — Health insurance          │
│     Must maintain OSHC                │
│     OSHC राख्नै पर्छ                 │
│                                      │
│  ┌────────────────────────────────┐  │
│  │ + Add custom condition         │  │
│  │   अन्य सर्त थप्नुहोस्          │  │
│  └────────────────────────────────┘  │
│                                      │
│  Source: immi.homeaffairs.gov.au     │
│                                      │
│       [Save Visa / भिसा सेभ गर्नुहोस्]│
└──────────────────────────────────────┘
```
- Conditions auto-populated from visa type knowledge base
- User can add custom conditions (e.g., specific work hours on 500 visa)
- Source cited at bottom

### 4.5 Dashboard (Visa Active)
```
┌──────────────────────────────────────┐
│         ⏱️  Visa Tracker             │
│                                      │
│  ┌────────────────────────────────┐  │
│  │  485 — Temporary Graduate      │  │
│  │                                │  │
│  │  ┌──────────────────────┐      │  │
│  │  │  247 days remaining  │      │  │
│  │  │  ████████████░░ 82%  │      │  │
│  │  │  Expires: 15 Mar 2027│      │  │
│  │  └──────────────────────┘      │  │
│  │                                │  │
│  │  Conditions:                   │  │
│  │  ✅ 8107 — Full work rights    │  │
│  │  ✅ 8501 — Maintain OSHC       │  │
│  │                                │  │
│  │  Next Steps:                   │  │
│  │  → Apply for 189/190 before    │  │
│  │    March 2027                  │  │
│  │  → Check points: 75 needed     │  │
│  │                                │  │
│  │  Reminders / सूचनाहरू:          │  │
│  │  🔔 180 days (Sep 2026)        │  │
│  │  🔔 90 days  (Dec 2026)        │  │
│  │  🔔 30 days  (Feb 2027)        │  │
│  │  🔔 7 days   (Mar 2027)        │  │
│  │                                │  │
│  │       [Edit]    [Delete]       │  │
│  └────────────────────────────────┘  │
│                                      │
│  [+ Add Another Visa]               │
│                                      │
│  ⚠️ Dates are user-entered. Verify  │
│  against your visa grant letter.    │
└──────────────────────────────────────┘
```

### 4.6 Expiry Warning State (< 30 Days)
```
┌──────────────────────────────────────┐
│         ⏱️  Visa Tracker             │
│                                      │
│  ┌────────────────────────────────┐  │
│  │  ⚠️ 485 — Temporary Graduate   │  │
│  │                                │  │
│  │  ┌──────────────────────┐      │  │
│  │  │  ⚠️ 21 days remaining│      │  │
│  │  │  ██░░░░░░░░░░░░░ 5% │      │  │
│  │  │  Expires: 29 Aug 2026│      │  │
│  │  └──────────────────────┘      │  │
│  │                                │  │
│  │  🚨 Your visa expires soon!    │  │
│  │  तपाईंको भिसा चाँडै सकिँदै छ!  │  │
│  │                                │  │
│  │  What to do now:               │  │
│  │  1. Apply for next visa ASAP   │  │
│  │  2. Check bridging visa options│  │
│  │  3. Consult migration agent    │  │
│  │     (verify at mara.gov.au)    │  │
│  │                                │  │
│  │  [Find a MARA Agent →]        │  │
│  └────────────────────────────────┘  │
└──────────────────────────────────────┘
```
- Red-amber color scheme for urgency
- Actionable next steps, not just panic
- MARA agent handoff (information only — link to mara.gov.au search)

### 4.7 Push Notification Content
```
🔔 Saathi: Your 485 visa expires in 90 days (15 Dec 2026).
Start preparing your next application now.

🔔 Saathi Visa Reminder:
तपाईंको 485 भिसा ९० दिनमा सकिँदै छ।
अर्को आवेदनको तयारी सुरु गर्नुहोस्।
```

### References for F1 UX Patterns
- **Flight Tracker apps** (countdown + status color coding)
- **Todoist** (deadline visualization, project templates)
- **Medication reminder apps** (MyTherapy — recurring reminders + missed dose tracking)
- **Duolingo** (streak counter for habit formation)

---

## 5. F2 — Points Calculator UX

### 5.1 Start Screen
```
┌──────────────────────────────────────┐
│         🧮  Points Calculator         │
│                                      │
│  Estimate your skilled migration     │
│  points score before paying an agent │
│                                      │
│  एजेन्टलाई पैसा तिर्नु अघि आफ्नो     │
│  स्किल माइग्रेसन पोइन्ट अनुमान गर्नुहोस्│
│                                      │
│       [Start / सुरु गर्नुहोस्]        │
│                                      │
│  ⚠️ This is an estimate only.        │
│  For formal assessment, consult      │
│  a registered migration agent.       │
└──────────────────────────────────────┘
```

### 5.2 Question Step
```
┌──────────────────────────────────────┐
│  ← Back     Step 3 of 12    80% ████ │
│                                      │
│  Age / उमेर                           │
│                                      │
│  How old are you?                    │
│  तपाईंको उमेर कति हो?                │
│                                      │
│  ○ 18–24 years  —  25 points        │
│  ● 25–32 years  —  30 points        │
│  ○ 33–39 years  —  25 points        │
│  ○ 40–44 years  —  15 points        │
│  ○ 45+ years    —   0 points        │
│                                      │
│  Points so far: 30                   │
│                                      │
│  ℹ️ Age is calculated at the time     │
│  of invitation, not application.     │
│                                      │
│         [Next / अगाडि]               │
└──────────────────────────────────────┘
```

**Key UX patterns for each question type:**
- **Age:** Radio buttons with points shown inline
- **English score:** Dropdown → auto-maps IELTS/PTE/TOEFL scores to points
- **Work experience:** Number input (years) + location selector (AU/overseas)
- **Education:** Radio buttons (PhD/Bachelors/Diploma/etc.)
- **Partner skills:** Conditional (only shown if user indicates they have a partner)
- **State nomination:** Dropdown of states/territories

### 5.3 English Score Input (Complex Field)
```
┌──────────────────────────────────────┐
│  English Language / अंग्रेजी भाषा     │
│                                      │
│  Test Type / परीक्षा प्रकार:          │
│  ┌────────────────────────────────┐  │
│  │ IELTS ▼                        │  │
│  │ ├ IELTS                        │  │
│  │ ├ PTE Academic                 │  │
│  │ ├ TOEFL iBT                    │  │
│  │ └ OET                          │  │
│  └────────────────────────────────┘  │
│                                      │
│  Overall Score / समग्र स्कोर:         │
│  ┌────────────────────────────────┐  │
│  │ 7.0 ▼                          │  │
│  │ ├ 8.0+ → 20 points             │  │
│  │ ├ 7.0  → 10 points             │  │
│  │ └ 6.0  →  0 points             │  │
│  └────────────────────────────────┘  │
│                                      │
│  Points: 10                         │
│                                      │
│  ℹ️ Your test must be less than      │
│  3 years old at time of invitation. │
└──────────────────────────────────────┘
```

### 5.4 Results Screen
```
┌──────────────────────────────────────┐
│         🧮  Your Results              │
│                                      │
│  ┌────────────────────────────────┐  │
│  │                                │  │
│  │     Your Score: 75 points      │  │
│  │     ████████████████░░░░ 75%   │  │
│  │                                │  │
│  └────────────────────────────────┘  │
│                                      │
│  Breakdown / विवरण:                  │
│  ┌────────────────────────────────┐  │
│  │ Age (25-32)             30 pts │  │
│  │ English (IELTS 7.0)     10 pts │  │
│  │ Overseas work (5 yrs)   10 pts │  │
│  │ AU work (1 yr)           5 pts │  │
│  │ Education (Bachelors)   15 pts │  │
│  │ Partner skills            5 pts │  │
│  └────────────────────────────────┘  │
│                                      │
│  📊 Latest Invitation Rounds:         │
│  ┌────────────────────────────────┐  │
│  │ 189 (Jun 2026): 85+ points     │  │
│  │ 190 (Jun 2026): 70+ points     │  │
│  │ 491 (Jun 2026): 65+ points     │  │
│  └────────────────────────────────┘  │
│                                      │
│  💡 How to improve:                  │
│  • NAATI CCL test: +5 pts           │
│  • Professional Year: +5 pts        │
│  • Retake IELTS for 8.0: +10 pts   │
│                                      │
│  [Save]  [Share]  [Recalculate]      │
│                                      │
│  ⚠️ Estimate only. Consult MARA agent│
└──────────────────────────────────────┘
```

### 5.5 Compare Mode (Saved Results)
```
┌──────────────────────────────────────┐
│   Saved Calculations / सेभ गरिएका     │
│                                      │
│  ┌──────────────────────────────┐    │
│  │ Jul 2026: 75 pts  [View]     │    │
│  ├──────────────────────────────┤    │
│  │ Jun 2026: 70 pts  [View]     │    │
│  ├──────────────────────────────┤    │
│  │ May 2026: 65 pts  [View]     │    │
│  └──────────────────────────────┘    │
│                                      │
│  [+ New Calculation]                 │
└──────────────────────────────────────┘
```

### References for F2 UX Patterns
- **Typeform** (one-question-at-a-time wizard, progress bar)
- **Australia SkillSelect Calculator** (official — learn what's confusing about it and fix it)
- **Tax calculators** (TaxBee, H&R Block — step-by-step data gathering)
- **Credit Karma** (score visualization, breakdown, improvement suggestions)

---

## 6. F3 — Document Checklist UX

### 6.1 Visa Selection
```
┌──────────────────────────────────────┐
│         📋  Document Checklist        │
│                                      │
│  Select your visa type               │
│  भिसा प्रकार छान्नुहोस्              │
│                                      │
│  ┌────────────────────────────────┐  │
│  │ 🎓 485 Temporary Graduate      │  │
│  │    485 अस्थायी स्नातक          │  │
│  │    Most common for recent grads│  │
│  ├────────────────────────────────┤  │
│  │ 🎓 500 Student Visa Extension   │  │
│  │    500 विद्यार्थी भिसा नवीकरण   │  │
│  ├────────────────────────────────┤  │
│  │ 🏠 189 Skilled Independent     │  │
│  │    189 दक्ष स्वतन्त्र           │  │
│  ├────────────────────────────────┤  │
│  │ 🏠 190 Skilled Nominated       │  │
│  │    190 दक्ष मनोनीत             │  │
│  ├────────────────────────────────┤  │
│  │ 🏠 491 Skilled Regional        │  │
│  │    491 दक्ष क्षेत्रीय          │  │
│  ├────────────────────────────────┤  │
│  │ 💼 482 TSS / Skills in Demand  │  │
│  │    482 सीपको मागमा            │  │
│  └────────────────────────────────┘  │
└──────────────────────────────────────┘
```

### 6.2 Branching Questions
```
┌──────────────────────────────────────┐
│  ← Back     485 Checklist   Q 2 of 4 │
│                                      │
│  Which stream? / कुन stream?          │
│                                      │
│  ● Graduate Work                     │
│    (skills assessment required)      │
│                                      │
│  ○ Post-Study Work                   │
│    (no skills assessment needed)     │
│                                      │
│         [Next / अगाडि]               │
└──────────────────────────────────────┘
```

### 6.3 Generated Checklist
```
┌──────────────────────────────────────┐
│  ← Back    485 Graduate Work         │
│                                      │
│  Your Checklist / तपाईंको चेकलिस्ट    │
│                                      │
│  📋 Identity / पहिचान (3 items)       │
│  ┌────────────────────────────────┐  │
│  │ ✅ Certified passport copy     │  │
│  │ ✅ Birth certificate           │  │
│  │ ☐ Passport photo (recent)     │  │
│  └────────────────────────────────┘  │
│                                      │
│  📋 Study / अध्ययन (4 items)         │
│  ┌────────────────────────────────┐  │
│  │ ☐ Completion letter            │  │
│  │ ☐ Academic transcript          │  │
│  │ ☐ AFP police check             │  │
│  │ ☐ English test result          │  │
│  └────────────────────────────────┘  │
│                                      │
│  📋 Skills Assessment / सीप (2 items) │
│  ┌────────────────────────────────┐  │
│  │ ☐ Skills assessment outcome    │  │
│  │ ☐ Employment references        │  │
│  └────────────────────────────────┘  │
│                                      │
│  📋 Health / स्वास्थ्य (1 item)       │
│  ┌────────────────────────────────┐  │
│  │ ☐ OSHC certificate             │  │
│  └────────────────────────────────┘  │
│                                      │
│  3 of 10 gathered / १० मध्ये ३ तयार │
│  ████░░░░░░░░░░░░ 30%               │
│                                      │
│  [Print]  [Save]  [Share as PDF]     │
│                                      │
│  Last verified: June 2026            │
│  Source: immi.homeaffairs.gov.au     │
└──────────────────────────────────────┘
```

### 6.4 Checklist Item Detail (Expanded)
```
┌──────────────────────────────────────┐
│  ← Checklist                         │
│                                      │
│  Certified copy of passport          │
│  पासपोर्टको प्रमाणित प्रतिलिपि       │
│                                      │
│  📖 What is this? / यो के हो?        │
│  A certified copy of the bio-data    │
│  page of your current passport.      │
│  तपाईंको हालको पासपोर्टको जैविक     │
│  डाटा पृष्ठको प्रमाणित प्रतिलिपि।    │
│                                      │
│  🛠️ How to get it / कसरी पाउने:      │
│  1. Photocopy your passport bio page │
│  2. Take original + copy to a JP     │
│     (Justice of the Peace)           │
│  3. JP will stamp + sign the copy    │
│  4. Cost: Free at most libraries     │
│     and police stations              │
│                                      │
│  ⚠️ Common mistakes / सामान्य गल्ती: │
│  • Not certified by authorised person│
│  • Expired certification (>6 months) │
│  • Copying wrong page                │
│                                      │
│  📎 Source: immi.homeaffairs.gov.au   │
│  Verified: June 2026                 │
│                                      │
│  [Mark as Gathered / तयार छ]          │
│  [I need help with this]             │
└──────────────────────────────────────┘
```

### 6.5 Empty/Complete States
- **All items gathered:** Confetti animation + "Ready to Apply! / आवेदन गर्न तयार!" + reminder to double-check with migration agent
- **Partial progress:** Progress bar + motivational message in Nepali
- **No visa selected:** Browse cards → pick one

### References for F3 UX Patterns
- **Notion databases** (categorized lists, progress tracking, inline status)
- **Trello** (kanban-style card progression: needed → in progress → gathered)
- **IKEA assembly instructions** (step-by-step visual checklist)
- **Packing list apps** (PackPoint — item categorization, smart suggestions)

---

## 7. F4 — Form Helper + Scan UX

### 7.0 Architecture Note — F4 Is a Consumer, Not a Pipeline

Saathi does **not** implement classify/extract/validate/transliterate/fill itself.
That entire pipeline is owned by `manaslu` (separate repo, headless), exposed as a
REST+SSE API (`manaslu/docs/architecture/06-service-api.md`). Every screen below is
a rendering of what that API returns — Saathi's job is presentation, not inference.

- `POST /v1/sessions` + document upload → starts a session; Saathi forwards the
  end-user's auth JWT.
- SSE events drive screen state:

  | Event | Screen it drives |
  |-------|-------------------|
  | `tool.started` / `tool.finished` | Progress states (§7.3, §7.4) — "Checking your profile…", "Reading your passport…" |
  | `extraction.ready` | Populates the review screen (§7.5) with fields, confidence tiers, source-region refs |
  | `review.required` | Pauses the session; Saathi renders the specific ask — a field to confirm, a transliteration choice (§7.6), or an unsourced field (§7.7) — and must eventually `POST /confirmations` |
  | `fill.completed` | Success screen with artifact IDs for the filled PDF + audit annex (§7.8) |
  | `session.error` | Error state; retry or fall back to manual entry |

- `GET /v1/sessions/{id}` always reflects pending `review.required` items, so if a
  user closes the app mid-review and comes back, Saathi re-renders the same pause —
  no lost state, no re-upload.
- Two distinct sources of bilingual content, don't conflate them:
  - **§7.2 Field Explainer** (browse mode, no documents involved) is Saathi's own
    knowledge service — RAG over Home Affairs pages, per `architecture-services-and-features.md`
    §4.6. Deep explanations + common mistakes, written/curated by Saathi.
  - **§7.5 Extraction Review** labels + short explanations are **not** Saathi
    content — they ship inside manaslu's `extraction.ready`/`review.required`
    payloads, sourced from the field manifest (curated per-form, bilingual,
    versioned in manaslu). Saathi renders whatever the manifest says; it doesn't
    author or store this copy.

### 7.1 Form Selection
```
┌──────────────────────────────────────┐
│         📝  Form Helper               │
│                                      │
│  What form are you filling?          │
│  कुन फारम भर्दै हुनुहुन्छ?          │
│                                      │
│  ┌────────────────────────────────┐  │
│  │ 📄 Form 80                     │  │
│  │ Personal particulars for       │  │
│  │ character assessment           │  │
│  │ व्यक्तिगत विवरण फारम           │  │
│  │                                │  │
│  │ 127 fields · Used for most     │  │
│  │ visa types                     │  │
│  ├────────────────────────────────┤  │
│  │ 📄 Form 1221                   │  │
│  │ Additional personal particulars│  │
│  │ अतिरिक्त व्यक्तिगत विवरण       │  │
│  │                                │  │
│  │ 85 fields · Commonly required  │  │
│  ├────────────────────────────────┤  │
│  │ 🌐 485 Online Application      │  │
│  │ Key sections explained         │  │
│  │ मुख्य भागहरूको व्याख्या        │  │
│  ├────────────────────────────────┤  │
│  │ 🌐 189/190 EOI (SkillSelect)   │  │
│  │ Expression of Interest sections│  │
│  │ EOI का भागहरूको व्याख्या       │  │
│  └────────────────────────────────┘  │
│                                      │
│  Or upload your documents — we'll    │
│  check what we already know first:   │
│  [📤 Upload Documents →]             │
└──────────────────────────────────────┘
```
- Form list = manaslu's `get_form_manifest` catalog. MVP ships Form 80 only
  (manaslu M2); Form 1221 is the fast-follow — deliberately, since it's the
  vault-reuse demo (§7.4).

### 7.2 Field Explainer (Chat-Like Interface) — Browse Mode
```
┌──────────────────────────────────────┐
│  ← Form 80         Question 4 of 127 │
│                                      │
│  ┌────────────────────────────────┐  │
│  │ 🇬🇧 Date of Birth               │  │
│  │ 🇳🇵 जन्म मिति                   │  │
│  │                                │  │
│  │ 📖 What this means:            │  │
│  │ Enter your date of birth in    │  │
│  │ DD/MM/YYYY format as shown on  │  │
│  │ your passport or birth cert.   │  │
│  │                                │  │
│  │ तपाईंको पासपोर्ट वा जन्म       │  │
│  │ दर्ता प्रमाणपत्रमा भएको        │  │
│  │ जन्म मिति DD/MM/YYYY ढाँचामा   │  │
│  │ लेख्नुहोस्।                    │  │
│  │                                │  │
│  │ ⚠️ Common mistakes:            │  │
│  │ • Using MM/DD/YYYY format      │  │
│  │   (this is a US format — AU    │  │
│  │    uses DD/MM/YYYY)            │  │
│  │ • Date differs from passport   │  │
│  │   (DHA cross-checks this)       │  │
│  │                                │  │
│  │ 📎 Source: immi.homeaffairs     │  │
│  │ .gov.au/form-80                │  │
│  │ Verified: June 2026            │  │
│  └────────────────────────────────┘  │
│                                      │
│  ┌────────────────────────────────┐  │
│  │ Was this helpful?              │  │
│  │ [👍 Yes]  [👎 No — tell us why]│  │
│  └────────────────────────────────┘  │
│                                      │
│  ⚠️ For information only. Not       │
│  migration advice.                  │
│                                      │
│  [← Prev Field]  [Next Field →]      │
└──────────────────────────────────────┘
```
This mode never touches manaslu — it's field-by-field reading, no documents
involved. See §7.0 for why its content is sourced differently from §7.5.

### 7.3 Document Upload — Starts a Manaslu Session
```
┌──────────────────────────────────────┐
│         📤  Upload Documents          │
│                                      │
│  Upload your documents. We'll check  │
│  your saved profile first, and only  │
│  ask you to scan what's missing.     │
│                                      │
│  कागजात अपलोड गर्नुहोस् — पहिले      │
│  तपाईंको सेभ गरिएको प्रोफाइल जाँच    │
│  गर्छौं, अनि बाँकी मात्र सोध्छौं।    │
│                                      │
│  Supported documents:                │
│  ┌────────────────────────────────┐  │
│  │ 🛂 Passport                    │  │
│  │ 📄 Visa Grant Letter           │  │
│  │ 💰 Payslip                     │  │
│  │ 🏦 Bank Statement              │  │
│  │ 🎓 Academic Transcript         │  │
│  │ 🏥 Birth Certificate           │  │
│  │ 📊 English Test Result         │  │
│  └────────────────────────────────┘  │
│                                      │
│  ┌────────────────────────────────┐  │
│  │                                │  │
│  │     📤 Drop files here or      │  │
│  │     tap to browse              │  │
│  │                                │  │
│  │     Max 10 files, 20MB each    │  │
│  └────────────────────────────────┘  │
│                                      │
│  🔒 Encrypted, stored & processed    │
│  in Australia. Audit-logged. Never   │
│  sold. [Privacy details →]           │
│                                      │
│  [Upload / अपलोड गर्नुहोस्]          │
└──────────────────────────────────────┘
```
- Upload triggers `POST /v1/sessions` + document handshake. Progress states
  ("Classifying document…" → "Reading your passport…") are literal renders of
  `tool.started`/`tool.finished` events — Saathi has no classifier of its own.
- **Privacy copy is load-bearing — do not soften into "processed on your
  device" or "never leaves your device."** manaslu's extraction runs on Claude
  Vision in an AU-region (`australia-southeast1`) GCP project; documents leave
  the device by design. The accurate claim is AU-region storage + processing,
  encrypted at rest, audit-logged, never sold — see manaslu
  `docs/architecture/10-security-privacy.md`.

### 7.4 Vault Pre-Check — Where the Vault Value Prop Becomes Visible
```
┌──────────────────────────────────────┐
│  Checking your saved profile...      │
│  तपाईंको प्रोफाइल जाँच गर्दै...      │
│                                      │
│  ┌────────────────────────────────┐  │
│  │  ✨ 14 of 20 fields already     │  │
│  │     have data from your        │  │
│  │     profile                    │  │
│  │  ✨ २० मध्ये १४ फिल्ड तयार छ    │  │
│  └────────────────────────────────┘  │
│                                      │
│  We'll only ask you to scan          │
│  documents for the remaining 6.      │
│                                      │
│  [See what's pre-filled →]           │
└──────────────────────────────────────┘
```
- This is the screen-level expression of manaslu's vault-first agent ordering
  (`recall_facts` before any new extraction — doc 11 §3): "the *second* form a
  user fills arrives ~80% pre-filled" only *lands as a product feature* if a
  user can actually see it happening. Without this screen the vault is
  invisible backend logic.
- First-ever session for a brand-new user: this screen is skipped (nothing to
  recall yet) — go straight to §7.3's upload flow.
- "See what's pre-filled" opens straight into §7.5 with vault-sourced fields
  already expanded and badged.

### 7.5 Extraction Review — Side-by-Side Value ↔ Source
```
┌──────────────────────────────────────┐
│  Review Your Information             │
│  तपाईंको जानकारी समीक्षा             │
│                                      │
│  Form 80 · Passport + your profile   │
│                                      │
│  ── Family Name / थर ──────────────  │
│  "Your surname as it appears on      │
│  official documents."                │
│  ┌────────────┐  ┌────────────────┐  │
│  │ KARKI      │  │ [passport crop]│  │
│  │ 🟢 High    │  │ zoomed to name │  │
│  │            │  │ field           │  │
│  └────────────┘  └────────────────┘  │
│                                      │
│  ── Passport Number / पासपोर्ट नं ── │
│  "Found in your profile — no need    │
│  to rescan."                         │
│  ┌────────────┐  ┌────────────────┐  │
│  │ PA1234567  │  │ ✨ Pre-filled   │  │
│  │ ✨ From    │  │ from your saved │  │
│  │ your vault │  │ profile · from  │  │
│  │            │  │ Form 80 scan,   │  │
│  │ [Edit ✏️]  │  │ 3 weeks ago     │  │
│  └────────────┘  └────────────────┘  │
│                                      │
│  ── Date of Birth / जन्म मिति ─────  │
│  "Must match your passport exactly." │
│  ┌────────────┐  ┌────────────────┐  │
│  │ 01/01/1990 │  │ [passport crop]│  │
│  │ 🟡 Medium  │  │ zoomed to DOB   │  │
│  │ [Edit ✏️]  │  │ field           │  │
│  └────────────┘  └────────────────┘  │
│                                      │
│  ── Employer / रोजगारदाता ─────────  │
│  "Your current employer's legal      │
│  name."                              │
│  ┌────────────┐  ┌────────────────┐  │
│  │ — no data  │  │ [payslip crop] │  │
│  │ 🔴 Not     │  │ text unreadable │  │
│  │  found     │  │                 │  │
│  │ [Enter ✏️] │  │                 │  │
│  └────────────┘  └────────────────┘  │
│                                      │
│  🟢 3 high  🟡 1 review  🔴 1 manual │
│  ✨ 1 from your saved profile        │
│                                      │
│  [Approve All Ready Fields]          │
│  [Review 🟡🔴 Fields First]          │
│                                      │
│  ⚠️ You are responsible for          │
│  verifying all information before    │
│  submitting to Home Affairs.         │
└──────────────────────────────────────┘
```
- **Every field row is one of three provenance states**, and the UI must
  distinguish them, not just show a value:
  1. **Freshly extracted** — value + confidence tier (badge, not a %, since
     manaslu owns the thresholds) + the source document crop it came from,
     shown side-by-side, per the API contract's "consumer renders review UI"
     obligation (manaslu doc 06).
  2. **Vault pre-filled** (`recall_facts`, not this session's scan) — the
     "✨ Pre-filled from your saved profile" badge plus a one-line provenance
     note (which document, how long ago) instead of a crop, since there's no
     crop from *this* session. Still editable — recall is a starting point,
     not a lock.
  3. **Unsourced / failed** — routed to a `review.required` (`ask_user`) event;
     manual entry required (§7.7).
- Bilingual label + one-line explanation per field are the manifest content
  described in §7.0 — never hardcoded per-form copy in Saathi.
- Confidence tiers render whatever label manaslu returns (High/Medium/Low) —
  Saathi does not invent or display numeric thresholds it doesn't own.

### 7.6 Transliteration Picker — Devanagari Source + Candidates
```
┌──────────────────────────────────────┐
│  Confirm Spelling                    │
│  हिज्जे पुष्टि गर्नुहोस्              │
│                                      │
│  Family Name — from Birth Certificate│
│                                      │
│  Source (your document):             │
│  ┌────────────────────────────────┐  │
│  │        श्रेष्ठ                  │  │
│  │   [zoomed Devanagari crop]      │  │
│  └────────────────────────────────┘  │
│                                      │
│  Suggested spellings:                │
│  ○ Shrestha    — commonly used       │
│  ○ Shreshtha   — generated           │
│  ○ Śreṣṭha     — ISO 15919, generated│
│                                      │
│  Or type it yourself:                │
│  ┌────────────────────────────────┐  │
│  │ [                              ]│  │
│  └────────────────────────────────┘  │
│                                      │
│  ℹ️ If this name is on your          │
│  passport, that spelling always      │
│  wins — check your passport first.   │
│                                      │
│  [Confirm Spelling / पुष्टि गर्नुहोस्]│
└──────────────────────────────────────┘
```
- Appears **only** as a last resort in manaslu's priority order (doc 04):
  MRZ/passport spelling → other official EN documents → this picker → free
  text. Most users never see it — passport names dominate.
- The candidates are deterministic (Aksharamukha-generated), never invented by
  Claude; Claude's role is limited to *ordering* them by conventional
  likelihood (doc 04) — the "commonly used"/"generated" labels reflect that,
  and must not be dropped, since they're the legibility of that constraint.
  Nothing auto-fills; every path ends in explicit user confirmation.
- Confirmed spelling is stored as a `user_entry` with provenance
  `transliteration: generated + user-confirmed` in the audit annex (§7.8).

### 7.7 Manual Entry Modal — Unsourced Field (`review.required`)
```
┌──────────────────────────────────────┐
│  Enter Field / फिल्ड भर्नुहोस्        │
│                                      │
│  Employer Name                       │
│  रोजगारदाताको नाम                     │
│                                      │
│  ┌────────────────────────────────┐  │
│  │ [Type employer name here]      │  │
│  └────────────────────────────────┘  │
│                                      │
│  Source document (for reference):     │
│  ┌────────────────────────────────┐  │
│  │ [Payslip image — zoomable]     │  │
│  └────────────────────────────────┘  │
│                                      │
│  Could not extract automatically —   │
│  neither a document nor your saved   │
│  profile had this. Please enter it.  │
│                                      │
│  [Save / सेभ]  [Cancel / रद्द]       │
└──────────────────────────────────────┘
```
- This is the UI rendering of a `review.required` event where neither
  extraction nor `recall_facts` produced a value. Saving here `POST`s a
  confirmation (`{request_id, field, value}`) and resumes the paused session.

### 7.8 Fill Completed
```
┌──────────────────────────────────────┐
│  ✅ Form 80 is ready                  │
│  फारम ८० तयार छ                       │
│                                      │
│  All fields confirmed. Every value    │
│  is traceable to a document or your   │
│  saved profile.                       │
│                                      │
│  [📥 Download Filled PDF]            │
│  [📄 Download Audit Annex]           │
│                                      │
│  The annex lists the source of every  │
│  field — useful if Home Affairs or    │
│  your migration agent asks.           │
│                                      │
│  ⚠️ You are responsible for verifying │
│  all information before submitting.   │
└──────────────────────────────────────┘
```
- Rendered on `fill.completed`; both files are signed-URL artifacts from
  `GET /v1/sessions/{id}/artifacts/{artifact_id}` — Saathi doesn't generate or
  store the PDF itself.

### References for F4 UX Patterns
- **TurboTax** (document upload → review → file — the gold standard for guided form filling)
- **Google Photos** (document scanning, auto-crop, enhance)
- **Zocdoc** (insurance card scan → auto-fill registration forms)
- **Superhuman / Linear** (keyboard shortcuts for power users)
- **GitHub PR review** (side-by-side diff — maps to extraction review layout)
- **Google Input Tools / Gboard** (ranked transliteration candidates + free-text fallback — maps to §7.6)

---

## 8. Component Patterns

### Pattern 1: Disclaimer Banner
```
┌──────────────────────────────────────────────┐
│  ⚠️ This is for information only. Not         │
│  migration advice. Consult a registered       │
│  migration agent (verify at mara.gov.au).     │
│                                              │
│  यो जानकारीको लागि मात्र हो। आप्रवासन         │
│  सल्लाह होइन। दर्ता भएको migration agent सँग  │
│  परामर्श गर्नुहोस् (mara.gov.au मा जाँच गर्नुहोस्)।│
└──────────────────────────────────────────────┘
```
- Fixed at bottom of every F4 screen
- Bilingual
- Non-dismissible on F4; dismissible (with cookie) on F1/F2/F3

### Pattern 2: Confidence Badge
```
🟢 HIGH    → bg-green-100 text-green-800 border-green-300
🟡 MEDIUM  → bg-amber-100 text-amber-800 border-amber-300
🔴 LOW     → bg-red-100 text-red-800 border-red-300
⚪ BLANK   → bg-gray-100 text-gray-500 border-gray-200
```

### Pattern 3: Source Citation Footer
```
┌──────────────────────────────────────────────┐
│  📎 Source: immi.homeaffairs.gov.au/form-80   │
│  ✅ Verified: June 2026                        │
└──────────────────────────────────────────────┘
```
- Always visible at bottom of content cards
- Red indicator if last_verified > 90 days

### Pattern 4: Loading States
- **Skeleton screens** (not spinners) for initial load
- **Progress bar** for multi-step processes (upload → classify → extract → review)
- Estimated time: "Processing... usually takes 5-10 seconds / सामान्यतया ५-१० सेकेन्ड लाग्छ"

### Pattern 5: Empty States
Every empty state has:
1. Icon (contextual)
2. Title (EN + NP)
3. Description (EN + NP) — what the user should do
4. CTA button (primary action)
5. Secondary link ("Learn more", "See example")

### Pattern 6: Anti-Patterns to Avoid
Common failure modes in immigration/fintech apps, and Saathi's counter:

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
| Overclaiming "on-device" / "never leaves your device" privacy | False claim once documents hit a cloud extraction pipeline; erodes trust when discovered | State the real posture — AU-region storage + processing, encrypted, audit-logged, never sold (§7.3, manaslu doc 10) |

---

## 9. State Handling

### Loading States
| Component | Loading State |
|-----------|--------------|
| Visa Tracker dashboard | Skeleton cards |
| Points Calculator wizard | Step skeleton |
| Checklist generation | "Generating your checklist..." with progress dots |
| Form Explainer | "Finding the best explanation..." with typing indicator |
| Vault check (F4, manaslu `recall_facts`) | "Checking your saved profile..." (§7.4) |
| Document upload (F4, manaslu session) | Progress bar rendering `tool.started`/`tool.finished`: "Classifying document..." → "Extracting fields..." |
| PDF generation | "Assembling your PDF..." |

### Error States
| Error | User Message (EN) | User Message (NP) |
|-------|------------------|-------------------|
| Network offline | "You're offline. Some features limited." | "तपाईं अफलाइन हुनुहुन्छ। केही सुविधा सीमित।" |
| AI timeout | "Taking longer than expected. Retrying..." | "अपेक्षा भन्दा बढी समय लाग्यो। पुनः प्रयास..." |
| Extraction failed | "Could not read this field. Enter manually." | "यो फिल्ड पढ्न सकिएन। आफैँ भर्नुहोस्।" |
| Storage full | "Upload limit reached. Free up space." | "अपलोड सीमा पुग्यो। ठाउँ खाली गर्नुहोस्।" |
| Reconnect mid-review (F4) | "Picking up where you left off..." — `GET /sessions/{id}` restores any pending `review.required` item, so nothing is lost | "जहाँबाट छोड्नुभएको थियो त्यहीँबाट सुरु गर्दै..." |

### Empty States
| Context | Empty State |
|---------|------------|
| No visas added | "Add your first visa to start tracking" / "ट्रयाकिङ सुरु गर्न पहिलो भिसा थप्नुहोस्" |
| No saved calculations | "Calculate your points for the first time" / "पहिलो पटक पोइन्ट गणना गर्नुहोस्" |
| No checklists generated | "Generate a document checklist for your visa" / "भिसाको लागि कागजात चेकलिस्ट बनाउनुहोस्" |
| No extractions | "Upload a document to auto-fill your form" / "फारम स्वतः भर्न कागजात अपलोड गर्नुहोस्" |

---

## 10. Accessibility

### WCAG 2.1 AA Compliance
- **Color contrast:** All text ≥ 4.5:1 ratio (verified with axe DevTools)
- **Keyboard navigation:** All features fully operable via Tab/Enter/Escape
- **Screen readers:** All icons have aria-labels; bilingual content has `lang="ne"` attributes
- **Focus indicators:** Visible focus ring on all interactive elements
- **Font sizing:** Relative units (rem); supports browser zoom to 200%
- **Devangari readability:** Minimum 16px for Nepali text (smaller is unreadable for complex scripts)

### Bilingual Accessibility
- `lang` attribute switches between `en` and `ne` per component
- Screen readers use correct pronunciation per language
- RTL not needed (both English and Nepali are LTR scripts)

### Touch & Interaction
| Element | Requirement | Implementation |
|---------|------------|---------------|
| Minimum touch target | 44×44px (WCAG 2.5.5) | All tappable elements ≥ 44px |
| Card tap targets | Full card tappable | Not just the CTA button |
| Spacing between targets | ≥ 8px | Cards have 12px gap minimum |
| Swipe gestures | Alternative available | Carousel has dot navigation as alternative to swipe |

### Motion & Animation
- `prefers-reduced-motion` respected — disables all non-essential animation (countdown ring becomes a static SVG that updates on load)
- Auto-playing animation under 5 seconds or pausable
- Page transitions ≤ 300ms — simple fade, no parallax/slide

### Screen Reader Specifics (F4)
- Chat messages and streamed `message.delta` text use `role="log"` for live-region announcements — critical since F4's content arrives incrementally over SSE, not all at once
- Confidence indicators (🟢🟡🔴) always paired with a text alternative ("High confidence"), never color-only
- The extraction review's vault-prefill badge (§7.5) has its own `aria-label` ("Pre-filled from your saved profile, from Form 80, three weeks ago") — not just a visual sparkle icon

### Offline Accessibility
- All text content cached via service worker
- Saved visa information and checklist progress available offline
- Offline indicator announced to screen readers; stale-data timestamp always visible
- F4 (manaslu-backed) is explicitly **not** offline-capable — session/SSE requires connectivity; the offline banner must say so rather than implying a retry will work locally

---

## 11. F5 — News, Seminars & Opportunities UX (Phase 2)

> Traction-gated (PRD §4/§9). Visual designs: [`diagrams/saathi-screen-designs.html`](../../diagrams/saathi-screen-designs.html) §F5, screens F5.1–F5.6. Summary spec here; the board is the source for layout detail.

### Navigation change
Tab bar grows 4 → 5: **Home · Tracker · Points · Checklist · Forms**. Home is the digest surface carrying news + events; no core tool loses its tab. (Rejected: a dedicated News tab — buries the digest; a top-bar bell — too hidden for a retention feature.)

### Screens
| # | Screen | Purpose | Key rules |
|---|--------|---------|-----------|
| F5.1 | Home / Today digest | Ties all features: visa countdown → resume form-fill session → top personalised news → next event. Strict card order; tracker always first | Personalisation = subclass-tag match, why-line always shown ("तपाईं ५०० मा हुनुभएकाले") |
| F5.2 | News feed | Allowlisted sources; headline + labelled AI Nepali summary + source/date + link out; category chips (Visa rules · Students · SkillSelect · Fees) | Never full text. SkillSelect items deep-link to F2 compare |
| F5.3 | News detail | Summary + "does this affect you" (informational phrasing + MARN line) + prominent link-out + follow-topic push toggle | AI-generated label + "source is authoritative" on every item |
| F5.4 | Events list | Curated seminars/expos/workshops; city + online filters; free/paid + audience chips; NAATI/PY events show F2 points chip | **Migration seminars: MARN-verified presenters only**, number shown, register link. "Saathi is not the organiser" |
| F5.5 | Event detail | Full listing, verification trail ("listing verified [date]", organiser named, report-a-problem); register = link out; remind-me (FCM) + .ics | No attendee data, no payments in Saathi |
| F5.6 | Student corner | Deadline-first: scholarships, intake dates, PY/NAATI sessions, open days; every item sourced + last-verified | No course/college recommendations — bilingual disclaimer |

### Component reuse
Citation footer (Pattern 3), disclaimer banner (Pattern 1), staleness rules (§9), and the F3 content contract (what/where/source/verified) all apply unchanged to F5 listings. New atoms: relevance chip ("affects 485"), MARN-verified chip (links to mara.gov.au register), AI-summary label.

### Anti-patterns (additions to Pattern 6)
- Republishing article text (aggregation = headline + ≤2-sentence summary + attribution + link out)
- Listing a migration-topic seminar without a verifiable MARN
- Framing personalisation as advice ("you should attend/apply") — it's relevance filtering, labelled as such

---

## 12. F6 — Connect to an Agent UX (Phase 2, English-only)

> Traction-gated; the PRD §7 monetisation surface, so trust rules are strictest here. Visual designs: [`diagrams/saathi-screen-designs.html`](../../diagrams/saathi-screen-designs.html) §F6, screens F6.1–F6.6. **English-only UI** by product decision — agent correspondence happens in English; the bilingual MARN-handoff lines across the app are the entry points and stay bilingual.

### Entry points (no new tab)
Every MARN handoff line becomes tappable → F6 with topic context carried: F1 expiry-warning action list, F2 results disclaimer, F4a circumstance-dependent fields, F5 news "does this affect you" cards, plus a Home digest card. No 6th tab.

### Screens
| # | Screen | Purpose | Key rules |
|---|--------|---------|-----------|
| F6.1 | Agent directory | MARA-registered agents only; filters: topic, "Speaks Nepali", city/online; fees + response time upfront | Sort = specialisation match, **never pay-to-rank** (stated on screen); lapsed MARN = auto-delisted |
| F6.2 | Agent profile | Verification block first (MARN + mara.gov.au verify link + Saathi's last-verified date), practicalities, then CTAs | Referral-fee disclosure before any CTA; no public star-ratings at launch (defamation/gaming risk) |
| F6.3 | Enquiry / request-a-call | Topic (pre-filled from entry context) + optional ≤500-char message + contact preference + call windows | An introduction, not a case file — copy nudges detail to the consult |
| F6.4 | **Share-details consent review** | Item-by-item opt-in: contact + enquiry pre-ticked; visa/points/checklist summaries **off by default**; consent per-agent per-enquiry, versioned, revocable | **Documents/filled forms never flow through F6** (no manaslu API path — structural). Referral disclosure repeats at send |
| F6.5 | Confirmation | Response-time expectation + status timeline (Delivered → Viewed → Responded) + the boundary card: from here it's client ↔ agent directly, Saathi can't see the advice | 4 business days silent → nudge agent + offer two alternatives |
| F6.6 | My Enquiries & data controls | All enquiries with status; accept proposed call time (.ics + FCM, reuses F5 plumbing); per-agent **Revoke** of shared data | Revoke → agent notified with deletion obligation, enquiry closes, full history in audit log; private post-consult feedback feeds delisting, not ratings |

### Anti-patterns (additions to Pattern 6)
- Pay-to-rank or undisclosed placement in the directory
- Sharing any data item the user didn't individually tick
- Public reviews/ratings of regulated professionals without a moderation plan
- Any UI implying Saathi participates in or endorses the advice given

---

*UI/UX flows compiled: August 8, 2026 · F4 revised for manaslu API integration · F5 + F6 (Phase 2) added August 8, 2026*