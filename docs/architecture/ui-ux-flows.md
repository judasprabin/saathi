# Saathi — UI/UX Flow Design

**Version:** 1.0 | **Date:** August 8, 2026

Comprehensive screen-by-screen UX design for all 4 features. Bilingual (EN/NP). Mobile-first PWA.

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
- Based on **shadcn/ui** (Radix primitives + Tailwind)
- Custom components for Saathi-specific patterns (countdown timer, points breakdown, checklist accordion, extraction review table)

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
│  Or upload your form to auto-fill:   │
│  [📤 Upload Documents →]             │
└──────────────────────────────────────┘
```

### 7.2 Field Explainer (Chat-Like Interface)
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

### 7.3 Document Upload (Scan Mode)
```
┌──────────────────────────────────────┐
│         📤  Upload Documents          │
│                                      │
│  Upload your documents and Saathi    │
│  will auto-fill your form fields.    │
│                                      │
│  कागजात अपलोड गर्नुहोस् — Saathi ले  │
│  फारम स्वतः भरिदिनेछ।               │
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
│  [Upload / अपलोड गर्नुहोस्]          │
└──────────────────────────────────────┘
```

### 7.4 Extraction Review (Side-by-Side)
```
┌──────────────────────────────────────┐
│  Review Extractions / निकासी समीक्षा │
│                                      │
│  Form: Form 80  |  Docs: Passport    │
│                                      │
│  ┌──────┬──────────┬─────────────┐   │
│  │Field │Extracted │Source       │   │
│  ├──────┼──────────┼─────────────┤   │
│  │Family│KARKI  🟢 │[passport 🔍]│   │
│  │ Name │          │             │   │
│  ├──────┼──────────┼─────────────┤   │
│  │Given │PRABIN 🟢 │[passport 🔍]│   │
│  │Names │          │             │   │
│  ├──────┼──────────┼─────────────┤   │
│  │DOB   │01/01/ 🟡 │[passport 🔍]│   │
│  │      │1990      │             │   │
│  ├──────┼──────────┼─────────────┤   │
│  │Passp.│PA1234567🟢│[passport 🔍]│   │
│  │No.   │          │             │   │
│  ├──────┼──────────┼─────────────┤   │
│  │Nation│NEPAL  🟢 │[passport 🔍]│   │
│  │ality │          │             │   │
│  ├──────┼──────────┼─────────────┤   │
│  │Employ│________🔴│[payslip 🔍] │   │
│  │er    │Could not │             │   │
│  │      │read      │             │   │
│  └──────┴──────────┴─────────────┘   │
│                                      │
│  🟢 5 high confidence — auto-filled  │
│  🟡 1 medium confidence — review     │
│  🔴 1 failed — enter manually        │
│                                      │
│  [Confirm All 🟢]                    │
│  [Review 🟡🔴 Fields First]          │
│                                      │
│  ⚠️ You are responsible for          │
│  verifying all information before    │
│  submitting to Home Affairs.         │
│                                      │
│  [Download Filled PDF / PDF डाउनलोड] │
└──────────────────────────────────────┘
```

### 7.5 Field Edit Modal
```
┌──────────────────────────────────────┐
│  Edit Field / फिल्ड सम्पादन           │
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
│  Could not extract automatically.    │
│  Please enter from your document.    │
│                                      │
│  [Save / सेभ]  [Cancel / रद्द]       │
└──────────────────────────────────────┘
```

### References for F4 UX Patterns
- **TurboTax** (document upload → review → file — the gold standard for guided form filling)
- **Google Photos** (document scanning, auto-crop, enhance)
- **Zocdoc** (insurance card scan → auto-fill registration forms)
- **Superhuman / Linear** (keyboard shortcuts for power users)
- **GitHub PR review** (side-by-side diff — maps to extraction review table)

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

---

## 9. State Handling

### Loading States
| Component | Loading State |
|-----------|--------------|
| Visa Tracker dashboard | Skeleton cards |
| Points Calculator wizard | Step skeleton |
| Checklist generation | "Generating your checklist..." with progress dots |
| Form Explainer | "Finding the best explanation..." with typing indicator |
| Document upload | Progress bar + "Classifying document..." → "Extracting fields..." |
| PDF generation | "Assembling your PDF..." |

### Error States
| Error | User Message (EN) | User Message (NP) |
|-------|------------------|-------------------|
| Network offline | "You're offline. Some features limited." | "तपाईं अफलाइन हुनुहुन्छ। केही सुविधा सीमित।" |
| AI timeout | "Taking longer than expected. Retrying..." | "अपेक्षा भन्दा बढी समय लाग्यो। पुनः प्रयास..." |
| Extraction failed | "Could not read this field. Enter manually." | "यो फिल्ड पढ्न सकिएन। आफैँ भर्नुहोस्।" |
| Storage full | "Upload limit reached. Free up space." | "अपलोड सीमा पुग्यो। ठाउँ खाली गर्नुहोस्।" |

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

---

*UI/UX flows compiled: August 8, 2026*