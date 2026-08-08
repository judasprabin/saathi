# FALLBACK ANSWER SHEET — Form 80 & Form 1221

**Version:** 1.0  
**Date:** July 4, 2026  
**Purpose:** Unambiguously legal fallback architecture for Saathi — a PDF/web guide that lets users hand-copy their own data into immigration forms.

---

## WHAT THE USER SEES

### 1.1 Product Description

The **Answer Sheet** is a bilingual (English / Nepali), print-ready PDF or web view that:

1. Lists every question from Australian immigration **Form 80** and **Form 1221**
2. Displays the field label in English **and** Nepali (नेपाली)
3. Shows the **extracted factual value** (from the user's uploaded documents) alongside each field as a **reference** — for the user's information only
4. Provides a **blank column** for the user to hand-write their answer
5. Is clearly labelled: **"This is your own work — Saathi is not filling in this form for you"**

The user prints the Answer Sheet and hand-copies their data into their actual visa forms. Saathi never touches the official form.

---

### 1.2 Visual Structure

```
┌─────────────────────────────────────────────────────────────────────┐
│                                                                      │
│           FORM 80 — PERSONAL DETAILS FOR VISA APPLICANTS             │
│           Answer Sheet | Your Reference Guide                        │
│                                                                      │
│  IMPORTANT: This is YOUR answer sheet. Saathi is NOT filling       │
│  in this form for you. Complete each field by hand in your          │
│  own handwriting.                                                    │
│                                                                      │
├──────────────────────────┬─────────────────┬──────────────────────┤
│ Field Label (English)    │ Field Label (NP)│ Your Answer           │
├──────────────────────────┼─────────────────┼────────────────────────┤
│ 1. Family Name           │ थर               │ __________________   │
│ 2. Given Names           │ दिइएको नाम        │ __________________   │
│ 3. Other Names           │ अन्य नाम         │ __________________   │
│ 4. Date of Birth         │ जन्म मिति        │ __________________   │
│ 5. Place of Birth        │ जन्म स्थान       │ __________________   │
│ 6. Country of Birth      │ जन्म भएको देश    │ __________________   │
│ 7. Sex                   │ लिङ्ग            │ __________________   │
│ ...                      │ ...              │ ...                  │
└──────────────────────────┴─────────────────┴──────────────────────────┘
```

---

## FORM 80 — FIELD LIST

*Based on the publicly available Form 80 PDF from Department of Home Affairs*

### Section A: Personal Details

| Field Label (EN) | Field Label (NP) | Notes |
|-----------------|------------------|-------|
| 1. Family Name | थर | From passport |
| 2. Given Names | दिइएको नाम | From passport |
| 3. Other Names (including aliases) | अन्य नामहरू | If any |
| 4. Date of Birth | जन्म मिति | DD/MM/YYYY |
| 5. Place of Birth — Town/City | जन्म स्थान — गाउँ/शहर | |
| 6. Place of Birth — State/Province | जन्म स्थान — प्रदेश | |
| 7. Country of Birth | जन्म भएको देश | |
| 8. Sex | लिङ्ग | Male/Female/Other |
| 9. Height (cm) | अाइट (सेमी) | |
| 10. Eye Colour | आँखाको रंग | |
| 11. Relationship Status | सम्बन्धको स्थिति | |
| 12. Place of Intended Stay in Australia | अस्ट्रेलियामा बस्ने इच्छा | |
| 13. Passport Number | राहदानी नम्बर | |
| 14. Passport Country of Issue | राहदानी जारी भएको देश | |
| 15. Date of Issue | जारी मिति | |
| 16. Date of Expiry | म्याद सकिने मिति | |
| 17. Nationality | राष्ट्रियता | |
| 18. Citizenship — Current | हालको नागरिकता | |
| 19. Citizenship — Other (if any) | अन्य नागरिकता | |

### Section B: Contact Details in Australia

| Field Label (EN) | Field Label (NP) | Notes |
|-----------------|------------------|-------|
| 20. Residential Address — Line 1 | घरको ठेगाना — लाइन १ | |
| 21. Residential Address — Line 2 | घरको ठेगाना — लाइन २ | |
| 22. Suburb/Town | उपशहर/गाउँ | |
| 23. State | प्रदेश | |
| 24. Postcode | पोस्टकोड | |
| 25. Country | देश | |
| 26. Postal Address (if different) | हुलको ठेगाना | |
| 27. Telephone — Home | फोन — घर | |
| 28. Telephone — Work | फोन — काम | |
| 29. Mobile/Cell | मोबाइल | |
| 30. Email Address | इमेल ठेगाना | |

### Section C: Family Members

| Field Label (EN) | Field Label (NP) | Notes |
|-----------------|------------------|-------|
| 31. Name of Partner | जोडीको नाम | |
| 32. Date of Birth — Partner | जन्म मिति — जोडी | |
| 33. Relationship to you | तपाईंसँगको सम्बन्ध | |
| 34. Nationality — Partner | राष्ट्रियता — जोडी | |
| 35. Visa Type — Partner (if applicable) | भिसा प्रकार — जोडी | |
| 36. Name of Children (if any) | बच्चाहरूको नाम | |
| 37. Date of Birth — Children | जन्म मिति — बच्चाहरू | |
| 38. Nationality — Children | राष्ट्रियता — बच्चाहरू | |
| 39. Name of Other Dependants | अन्य आश्रितहरूको नाम | |
| 40. Relationship — Other Dependants | सम्बन्ध — अन्य आश्रितहरू | |

### Section D: Education and Employment

| Field Label (EN) | Field Label (NP) | Notes |
|-----------------|------------------|-------|
| 41. Current Occupation | हालको पेशा | |
| 42. Name of Employer/Current School | हालको कामदार/विद्यालयको नाम | |
| 43. Address — Employer/School | ठेगाना — कामदार/विद्यालय | |
| 44. Telephone — Employer/School | फोन — कामदार/विद्यालय | |
| 45. Highest Qualification Achieved | प्राप्त गरेको उच्च योग्यता | |
| 46. Date Course Completed | कोर्स सकिएको मिति | |
| 47. Country Where Qualification Obtained | योग्यता प्राप्त गरेको देश | |
| 48. English Proficiency | अंग्रेजी दक्षता | |

### Section E: Employment History

| Field Label (EN) | Field Label (NP) | Notes |
|-----------------|------------------|-------|
| 49. Previous Employment — Employer 1 Name | अघिल्लो रोजगार — कामदार १ नाम | |
| 50. Address — Employer 1 | ठेगाना — कामदार १ | |
| 51. Position Held | पद | |
| 52. Dates Employed — From/To | काम गरेको मिति — देखि/सम्म | |
| 53. Previous Employment — Employer 2 Name | अघिल्लो रोजगार — कामदार २ नाम | |
| 54. Address — Employer 2 | ठेगाना — कामदार २ | |
| 55. Position Held | पद | |
| 56. Dates Employed — From/To | काम गरेको मिति — देखि/सम्म | |

### Section F: Financial Information

| Field Label (EN) | Field Label (NP) | Notes |
|-----------------|------------------|-------|
| 57. Source of Funds | कोषको स्रोत | |
| 58. Amount of Funds Available | उपलब्ध रकम | |
| 59. Name of Financial Institution | वित्तीय संस्थाको नाम | |
| 60. Bank Account Number | बैंक खाता नम्बर | |
| 61. Account Type | खाता प्रकार | |
| 62. Relationship to Sponsor/Supporter (if applicable) | प्रायोजक/समर्थकसँगको सम्बन्ध | |

### Section G: Health and Character

| Field Label (EN) | Field Label (NP) | Notes |
|-----------------|------------------|-------|
| 63. Have you ever had a serious illness? | के तपाईंले कहिल्यै गम्भीर रोग लागेको छ? | |
| 64. Details of illness | रोगको विवरण | |
| 65. Have you undertaken a health examination? | के तपाईंले स्वास्थ्य परीक्षण गर्नुभएको छ? | |
| 66. Details of health examination | स्वास्थ्य परीक्षणको विवरण | |
| 67. Have you ever been charged with any criminal offence? | के तपाईं कहिल्यै फौजदारी अपराधमा चार्ज भएको छ? | |
| 68. Details of criminal charges | फौजदारी चार्जको विवरण | |
| 69. Have you ever been convicted? | के तपाईं कहिल्यै दोषी ठहरिएको छ? | |
| 70. Details of convictions | दोषी ठहरिएको विवरण | |

### Section H: Australian Immigration History

| Field Label (EN) | Field Label (NP) | Notes |
|-----------------|------------------|-------|
| 71. Have you ever been in Australia? | के तपाईं कहिल्यै अस्ट्रेलियामा हुनुभएको छ? | |
| 72. Arrival Date (if applicable) | आगमन मिति | |
| 73. Departure Date (if applicable) | प्रस्थान मिति | |
| 74. Visa Previously Held | पहिले राखेको भिसा | |
| 75. Current Visa (if applicable) | हालको भिसा | |
| 76. Visa Grant Date | भिसा दिइएको मिति | |
| 77. Visa Expiry Date | भिसा म्याद सकिने मिति | |
| 78. Have you ever had a visa cancelled? | के तपाईंको भिसा कहिल्यै रद्द गरिएको छ? | |
| 79. Details of cancellation | रद्दीकरणको विवरण | |
| 80. Have you ever been refused a visa? | के तपाईं कहिल्यै भिसा अस्वीकृत भएको छ? | |
| 81. Details of refusal | अस्वीकृतिको विवरण | |

---

## FORM 1221 — FIELD LIST

*Based on the publicly available Form 1221 PDF from Department of Home Affairs*

### Section A: Personal Details

| Field Label (EN) | Field Label (NP) | Notes |
|-----------------|------------------|-------|
| 1. Family Name | थर | |
| 2. Given Names | दिइएको नाम | |
| 3. Date of Birth | जन्म मिति | |
| 4. Place of Birth — Town/City | जन्म स्थान — गाउँ/शहर | |
| 5. Place of Birth — State/Province | जन्म स्थान — प्रदेश | |
| 6. Country of Birth | जन्म भएको देश | |
| 7. Sex | लिङ्ग | |
| 8. Country of Citizenship | नागरिकता देश | |
| 9. Passport Number | राहदानी नम्बर | |
| 10. Passport Country of Issue | राहदानी जारी भएको देश | |
| 11. Passport Date of Issue | राहदानी जारी मिति | |
| 12. Passport Date of Expiry | राहदानी म्याद सकिने मिति | |

### Section B: Contact Details

| Field Label (EN) | Field Label (NP) | Notes |
|-----------------|------------------|-------|
| 13. Address in Home Country | घर देशमा ठेगाना | |
| 14. Telephone — Home | फोन — घर | |
| 15. Telephone — Mobile | मोबाइल | |
| 16. Email Address | इमेल ठेगाना | |
| 17. Address in Australia (if applicable) | अस्ट्रेलियामा ठेगाना | |
| 18. Telephone — Australia | फोन — अस्ट्रेलिया | |

### Section C: Visa and Travel History

| Field Label (EN) | Field Label (NP) | Notes |
|-----------------|------------------|-------|
| 19. Current Visa Subclass | हालको भिसा उपवर्ग | |
| 20. Current Visa Grant Date | हालको भिसा दिइएको मिति | |
| 21. Date of Last Entry to Australia | अस्ट्रेलियामा पछिल्लो प्रवेश मिति | |
| 22. Place of Last Entry | पछिल्लो प्रवेश स्थान | |
| 23. Conditions on Current Visa | हालको भिसामा सर्तहरू | |
| 24. Have you had any other Australian visas? | के तपाईंसँग अन्य अस्ट्रेलियाली भिसाहरू छन्? | |

### Section D: Supporting Documents

| Field Label (EN) | Field Label (NP) | Notes |
|-----------------|------------------|-------|
| 25. Attached: Certified copy of passport | साथमा: राहदानीको प्रमाणित प्रतिलिपि | |
| 26. Attached: Passport-size photograph | साथमा: राहदानी आकारको फोटो | |
| 27. Attached: Evidence of financial capacity | साथमा: आर्थिक क्षमताको प्रमाण | |
| 28. Attached: Health insurance evidence | साथमा: स्वास्थ्य बीमाको प्रमाण | |
| 29. Attached: Other documents | साथमा: अन्य कागजातहरू | |

---

## HOW IT IS GENERATED (PIPELINE STAGE)

### Stage 5 — Answer Sheet Renderer

```
Stage 1: Document Upload (user uploads passport, payslips, bank statements)
    ↓
Stage 2: OCR + Document Intelligence (extracts structured data: name, DOB, etc.)
    ↓
Stage 3: Structured Data Store (structured JSON with extracted values)
    ↓
Stage 4a: Autofill PDF Writer (generates pre-filled PDF for user review)
    ↓
Stage 4b: Answer Sheet Renderer ← WE ARE HERE
    → Pulls from same Stage 3 structured JSON
    → Maps to Form 80 / Form 1221 field schema
    → Renders bilingual EN/NP field labels
    → Displays extracted value as REFERENCE only
    → Leaves "Your Answer" column BLANK
    → Outputs print-ready PDF or web view
    ↓
Stage 5: User receives Answer Sheet → prints → hand-copies → submits own form
```

**Key difference from autofill:**  
The Answer Sheet uses the same underlying extracted data, but renders it differently:
- **Autofill:** Writes the extracted value INTO the form field, ready to submit
- **Answer Sheet:** Shows the extracted value BESIDE the field, for reference, with a blank to fill by hand

---

## HOW IT AVOIDS THE IMMIGRATION ASSISTANCE LEGAL ISSUE

### Legal Analysis

The Answer Sheet architecture is **unambiguously legal** for the following reasons:

| Legal Issue | Answer Sheet Position | Why It's Legal |
|-------------|----------------------|----------------|
| s.276(1)(a) — "represent a person before the Department" | Saathi never interacts with the Department | No representation |
| s.276(1)(b) — "prepare, or authorise the preparation of, a form" | Saathi prepares a REFERENCE GUIDE, not the form itself; user prepares the actual form | Saathi does NOT prepare the form |
| s.276(1)(c) — "give immigration advice" | No advice is given; Saathi shows factual extracted data only | No advice |
| "Clerical exemption" | The user is doing their own clerical work (hand-copying) | Saathi is purely a reference tool |
| User control | User has 100% control over every answer they write | Saathi has no involvement in the final form |

**The critical distinction:**

> **Autofill:** Saathi puts data IN the form field → Saathi has prepared the form → potentially s.276(1)(b)  
> **Answer Sheet:** Saathi shows data BESIDE the form field → user puts data IN the form field → Saathi has not prepared the form

The Answer Sheet is analogous to:
- A friend showing you their passport so you can copy the details
- A dictionary showing you a word so you can spell it correctly
- A calculator showing you the calculation so you can write the number yourself

None of these activities require MARA registration.

---

## PROS AND CONS VS. AUTOFILL APPROACH

| Dimension | Autofill Architecture | Answer Sheet (Fallback) |
|-----------|---------------------|------------------------|
| **User Experience** | Fast — form pre-filled | Slower — hand-copy required |
| **Legal Risk** | Medium — genuinely uncertain | **Zero — definitively legal** |
| **Accuracy** | OCR/AI may introduce errors | Human copies correctly from their own docs |
| **User Control** | Medium — system fills in | Maximum — user fills in |
| **Effort to Implement** | Higher — PDF write logic required | Lower — print-ready PDF |
| **Can Launch Now?** | No — needs lawyer opinion first | Yes — can launch immediately |
| **Regulatory Exposure** | Possible — s.276(1)(b) risk | None |
| **Privacy** | Higher — less human review | Same |
| **Completeness** | May miss fields user hasn't uploaded docs for | All fields shown as blank if no data |

---

## IMPLEMENTATION NOTES

### Output Format
- Print-ready **PDF** (generated server-side from structured JSON + template)
- Also available as **web view** for users who prefer to view on screen
- Bilingual: English field labels with Nepali translations below

### Data Flow
1. User uploads documents → Stage 3 structured JSON
2. Answer Sheet Renderer reads Stage 3 JSON
3. Maps each extracted value to the corresponding Form 80/1221 field reference
4. If a field has no extracted value, shows "(No data extracted — please complete by hand)"
5. Renders with clear visual distinction between reference value and blank answer column
6. Displays prominent notice: "This is your own work. Saathi is providing this as a reference guide only."

### Recommended UX
- Offer the Answer Sheet as the **default free option**
- Offer the Autofill as a **premium feature** (available only after written legal opinion confirms legality)
- User can switch between modes at any time before submission

---

## BILINGUAL HEADER NOTICE

**English:**
> This Answer Sheet was prepared from documents you uploaded. Saathi has extracted factual information from your documents and displayed it here as a reference. **You must complete this form yourself** — in your own handwriting. Saathi is not a registered migration agent and is not filling in this form for you. If you need help, consult a registered migration agent (MARN-registered).

**नेपाली (Nepali):**
> यो उत्तर पाना तपाईंले अपलोड गरेको कागजातहरूबाट तयार गरिएको हो। Saathi ले तपाईंको कागजातहरूबाट तथ्यगत जानकारी निकालेर यहाँ सन्दर्भको रूपमा देखाएको छ। **तपाईंले यो फारम आफैं भर्नु पर्छ** — आफ्नै हातले। Saathi एक दर्ता भएको प्रवासन एजेन्ट होइन र तपाईंको लागि यो फारम भर्दैन। मद्दत चाहिन्छ भने, दर्ता भएको प्रवासन एजेन्ट (MARN-दर्ता) सँग सम्पर्क गर्नुहोस्।

---

## RECOMMENDATION

**The Answer Sheet should be built first and launched immediately** — it is the unambiguously legal path. The autofill architecture can be developed in parallel and launched later, after a written migration lawyer opinion confirms its legality.

This gives Saathi a **legal, working product from day one** while the regulatory question for the autofill feature is being resolved.

---

*Prepared as part of Saathi product development (Dophora-Nepal)*  
*Date: July 4, 2026*  
*See also: `legal-memo.md` for full legal analysis*