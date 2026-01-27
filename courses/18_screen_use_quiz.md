# Product Requirements Document: Screen-Use Risk & Benefit Self-Assessment Quiz

## 1. Vision

Provide a short quiz that helps users reflect on how screens affect different areas of their life (sleep, mood, focus, relationships, positive benefits), and receive a balanced profile plus suggestions and next steps.

The quiz is **not diagnostic**, only reflective and educational.

## 2. Problem Definition

Users are bombarded with extreme narratives (“screens are evil” vs “screens are fine”), and:

- Don’t know which risks apply to them personally.
- Underestimate some aspects (e.g., sleep) and overestimate others.
- Lack a structured way to reflect.

We provide a concise, well-structured quiz that outputs:

- Risk indicators (per dimension).
- Benefit indicators (per dimension).
- Personalized suggestions and links to relevant tools.

## 3. Behavioral Principles

- Reflection + personalized feedback → motivation.
- Balanced framing: risk and benefit.
- No moral judgment; emphasize experimentation.

## 4. Personas

- Adults reflecting on own use.
- Parents reflecting on a child’s screen habits.
- Teens doing it alongside a parent.

---

## 5. User Journey

1. User opens quiz.
2. Selects “For myself” or “About my child”.
3. Answers 10–15 Likert-scale questions (1–5 from “never” to “very often”).
4. Quiz computes:
   - Risk score per dimension (0–100).
   - Benefit score per dimension (0–100).
   - Overall risk level.
5. Results page:
   - Dimension-by-dimension summary.
   - 3–6 suggestions for next steps.
   - Links to relevant CuroSee tools (navigation only, no technical dependency).

---

## 6. Functional Requirements

### FR1 – Audience Choice

Inputs:

- `assessmentFor`: `"self" | "child"`
- If `"child"`, choose age range: `"3-5" | "6-9" | "10-12" | "13-17"`.

This may slightly change wording but does not affect scoring logic in v0.1.

### FR2 – Question Set

Static set of ~12 questions.

Each question has:

- `id`: string.
- `text`: string.
- `dimension`: `"sleep" | "mood" | "focus" | "relationships" | "positive_use"`.
- `direction`: `"risk" | "benefit"`.

Answer options: 1–5 numeric:

1. Never / almost never  
2. Rarely  
3. Sometimes  
4. Often  
5. Very often  

Meaning:
- For risk questions: higher → more risk.
- For benefit questions: higher → more positive effects.

### FR3 – Input Capture

- Store answers in a map `questionId -> 1–5`.
- All questions mandatory for scoring.

### FR4 – Scoring

For each dimension, compute:

- `riskScore`: 0–100
- `benefitScore`: 0–100

Rules:

- Sum all answers for risk questions in that dimension → `riskValue`.
- Maximum possible risk sum = `(5 * numberOfRiskQuestionsForThatDimension)`.
- `riskScore = (riskValue / riskMax) * 100`.

Similarly for benefit.

### FR5 – Dimension Categories

Risk categories:

- 0–33: `"low"`
- 34–66: `"watch"`
- 67–100: `"high"`

### FR6 – Overall Risk Level

Based on number/weight of high-risk dimensions:

- If at least 2 dimensions have `riskCategory === "high"` → `"high"`.
- Else if any dimension `riskCategory === "high"` or >2 `"watch"` → `"moderate"`.
- Else → `"low"`.

### FR7 – Suggestions & Next Steps

Based on highest-risk dimensions, suggest:

- Sleep issues high:
  - Suggest Sleep Risk Calculator + gentle advice.
- Mood/focus:
  - Suggest Screen-Time Tracker + Well-being Dashboard.
- Relationships:
  - Suggest Family Scheduler + Media Plan Wizard.
- Low risk but low benefit:
  - Suggest exploring positive uses (learning, creativity).

These are **UI links only** (no technical dependency).

---

## 7. Data Model

```ts
export type QuizDimension =
  | "sleep"
  | "mood"
  | "focus"
  | "relationships"
  | "positive_use";

export type QuestionDirection = "risk" | "benefit";

export interface QuizQuestion {
  id: string;
  text: string;
  dimension: QuizDimension;
  direction: QuestionDirection;
}

export interface QuizAnswerMap {
  [questionId: string]: number; // 1–5
}

export type RiskCategory = "low" | "watch" | "high";

export interface DimensionScore {
  dimension: QuizDimension;
  riskScore: number; // 0–100
  benefitScore: number; // 0–100
  riskCategory: RiskCategory;
}

export type OverallRiskLevel = "low" | "moderate" | "high";

export interface QuizResult {
  scores: DimensionScore[];
  overallRiskLevel: OverallRiskLevel;
  generatedSummary: string;
  suggestions: string[];
}
````

---

## 8. Local Service Contracts

```ts
export function getQuizQuestions(): QuizQuestion[];

export function scoreQuiz(
  answers: QuizAnswerMap,
  questions: QuizQuestion[]
): QuizResult;
```

Implementation:

* `getQuizQuestions` returns static array hard-coded in TS file.
* `scoreQuiz` follows the scoring rules above.

Optional: persist last result in local storage.

---

## 9. UI Specification

### Views

1. **Intro View**

   * Explanation about quiz purpose.
   * Radio: self vs child.

2. **Quiz View**

   * Simple, one question per row.
   * Progress bar (question X of N).
   * Button: Next / Back or single-page with auto-scroll.

3. **Results View**

   * Overall risk badge.
   * Cards per dimension:

     * Risk category, risk and benefit scores.
     * Short text snippet.
   * List of suggestions with links to tools pages.

### Components

* `<QuizIntro />`
* `<QuizForm />`
* `<QuizQuestionCard />`
* `<QuizProgressBar />`
* `<QuizResults />`

---

## 10. Technical Stack

* Vite + React + TypeScript + TailwindCSS.
* No backend or AI calls required for v0.1.

---

## 11. Privacy & Safety

* No PII captured.
* Quiz is not clinical; include disclaimer.

Suggested disclaimer:

> “This quiz is for reflection only and is not a medical or psychological assessment. If you are worried about your mental health or sleep, please consult a professional.”

---

## 12. Roadmap

* v0.1: Static question set, single language (English).
* v0.2: Language toggle, slight wording difference for child vs self.
* v1.0: Optional adaptive journey (e.g., follow-up questions depending on high-risk outputs).
