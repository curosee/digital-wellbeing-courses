# Product Requirements Document: Sleep & Screen Exposure Risk Calculator

## 1. Vision

Help users understand how their screen habits near bedtime may affect their sleep by giving a personalized, science-informed risk score and actionable recommendations — without medical claims or judgment.

The tool should feel like a **wellness reflection assistant**, not a diagnostic engine.

## 2. Problem Definition

Users frequently:
- Use screens shortly before sleep.
- Have trouble falling asleep.
- Do not understand the causal mechanisms or intensity of impact.
- Receive vague advice (“screens are bad at night”) which lacks personalization.

We provide a **simple, transparent calculator** that:
- Measures risk factors based on habit.
- Shows what changes have biggest impact.
- Suggests small adjustments that are easy to try.

## 3. Core Behavioral Principles (evidence-based)

- **Awareness + personalization ➜ motivation**  
  Personalized scoring prompts more action than general warnings.
- **Small shifts outperform bans**  
  Moving last screen time 30–60 min earlier is more sustainable than “no screens after 6pm”.
- **Non-judgmental framing**  
  Users are more likely to adopt routines if they don’t feel scolded or ashamed.

## 4. Target Personas

1. **Adults with mild sleep issues**  
   Want to experiment with better routines.
2. **Parents assessing children**  
   Want age-appropriate guidance without fear messaging.
3. **Teens/young adults**  
   Want to improve focus and rest but dislike restrictive rules.

## 5. User Journey

1. User opens the calculator.
2. Simple explanation appears: “How your screen habits affect your sleep.”
3. User fills inputs:
   - Usual bedtime.
   - Time of last screen use.
   - Daily usage duration before bed.
   - Type of use.
   - Brightness / night mode.
   - Age category (self / child).
4. User submits.
5. Tool returns:
   - Score (0–100).
   - Categorization: low / moderate / high.
   - 3–6 targeted suggestions.
6. User can adjust habits and re-run.

No registration or persistence required in v0.1.

---

## 6. Functional Requirements

### FR1 – Input Form

Required fields:
- `profileType`: `"self" | "child"`
- `typicalBedtime`: time `"HH:MM"`
- `lastScreenUseTime`: time `"HH:MM"`
- `screenDurationBeforeBedMinutes`: number 0–240
- `screenContentTypes`: array  
  `"social_media" | "video_streaming" | "gaming" | "work_study" | "reading" | "other"`
- `usesBlueLightFilter`: boolean
- `deviceBrightnessLevel`: `"low" | "medium" | "high"`
- `sleepQualitySelfRating`: number 1–5

If `profileType === "child"`:
- `ageRangeChild`: `"3-5" | "6-9" | "10-12" | "13-17"`

### FR2 – Validation
- `screenDurationBeforeBedMinutes <= 240`
- `sleepQualitySelfRating` must be integer 1–5.
- For child profile: `ageRangeChild` required.
- Do not block on impossible input, only warn gently.

### FR3 – Risk Scoring
Produce numeric score 0–100 using multi-factor additive model (defined below).

### FR4 – Result Output
Return structured result:
- `riskScore` 0–100
- `category`: `"low" | "moderate" | "high"`
- `explanations[]`: around 1–3 short paragraphs
- `suggestions[]`: 3–6 targeted suggestions

### FR5 – Optional Persistence
User may save last assessment locally for comparison.

---

## 7. Logic & Algorithms

### 7.1 Time Gap
Convert times to minutes-from-midnight.
```

gapMinutes = bedtimeMinutes - lastScreenUseMinutes
if gapMinutes < 0: gapMinutes = 0

````

Risk points:
- gap >= 120 ➜ +0
- 60–119 ➜ +10
- 30–59 ➜ +20
- 0–29 ➜ +30

### 7.2 Duration
- <= 30 ➜ +5
- 31–60 ➜ +10
- 61–120 ➜ +20
- >120 ➜ +25 max

### 7.3 Content Factors (per selection)
- gaming ➜ +15
- social_media ➜ +10
- work_study ➜ +8
- video_streaming ➜ +8
- reading ➜ +2
- other ➜ +5  
Max content factor = +25

### 7.4 Brightness/Filter
- usesBlueLightFilter = false ➜ +10
- brightness medium ➜ +5
- brightness high ➜ +10

### 7.5 Age Sensitivity Multiplier
If profile = child:
- 3–5 ➜ ×1.2
- 6–9 ➜ ×1.1
- 10–12 ➜ ×1.05
- 13–17 ➜ ×1.0

Clamp final score 0–100.

### 7.6 Categories
- <=30 ➜ `"low"`
- 31–60 ➜ `"moderate"`
- >=61 ➜ `"high"`

---

## 8. Data Model

```ts
export type AssessmentProfileType = "self" | "child";

export type AgeRangeChild =
  | "3-5"
  | "6-9"
  | "10-12"
  | "13-17";

export type ScreenContentType =
  | "social_media"
  | "video_streaming"
  | "gaming"
  | "work_study"
  | "reading"
  | "other";

export type BrightnessLevel = "low" | "medium" | "high";

export interface SleepRiskAssessmentInput {
  profileType: AssessmentProfileType;
  ageRangeChild?: AgeRangeChild;
  typicalBedtime: string; // HH:MM
  lastScreenUseTime: string; // HH:MM
  screenDurationBeforeBedMinutes: number;
  screenContentTypes: ScreenContentType[];
  usesBlueLightFilter: boolean;
  deviceBrightnessLevel: BrightnessLevel;
  sleepQualitySelfRating: number;
  createdAt: string; // ISO datetime
}

export type RiskCategory = "low" | "moderate" | "high";

export interface SleepRiskAssessmentResult {
  riskScore: number;
  category: RiskCategory;
  explanations: string[];
  suggestions: string[];
}
````

Storage key (if implemented):
`curosee_sleep_risk_v1`

---

## 9. Local Service Contracts

(No backend required)

```ts
export function calculateSleepRisk(
  input: SleepRiskAssessmentInput
): SleepRiskAssessmentResult;

export function saveLastSleepRisk(
  input: SleepRiskAssessmentInput,
  result: SleepRiskAssessmentResult
): Promise<void>;

export function loadLastSleepRisk(): Promise<{
  input: SleepRiskAssessmentInput;
  result: SleepRiskAssessmentResult;
} | null>;
```

---

## 10. UI Specification

### Views

1. **Intro**

   * Welcoming, simple explanation.
2. **Form**

   * Horizontal steps or single page.
3. **Results**

   * Score (large number).
   * Category badge (neutral colors).
   * Suggestion tiles.

### Components

* `<SleepRiskForm />`
* `<SleepRiskResults />`
* `<SuggestionList />`

No dependencies on other tools.

---

## 11. Privacy & Safety

* No PII.
* Only educational and reflective.
* Must clearly state:
  “This is not medical advice. Consult a healthcare professional for sleep disorders.”

---

## 12. KPIs (future optional)

* Completion rate.
* Re-assessment count per user.

---

## 13. Release Roadmap

* v0.1: Calculator, no persistence.
* v0.2: Save last result locally.
* v1.0: Multi-assessment history.
