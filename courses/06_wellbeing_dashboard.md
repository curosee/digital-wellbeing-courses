# Product Requirements Document: Screen Impact vs Well-being Dashboard

## 1. Vision

Provide a **reflection dashboard** that helps users see patterns between their screen-time habits and how they feel (mood, energy, focus, sleep quality), so they can run their own “experiments” and adjust gently over time.

The tool must work:

- **Standalone** (user manually enters total screen time per day), and
- **Optionally** read totals from the Screen-Time Tracker if present.

## 2. Problem Definition

Common issues:

- Users log screen time or see OS stats, but don’t link it clearly to well-being.
- Mood or sleep are felt subjectively, not tracked with data.
- Without a simple visual, users miss simple patterns (“on heavy-screen days my sleep quality is worse”).

We need a **non-clinical**, simple visual that correlates:

- Total daily screen time  
with  
- Self-rated mood, energy, focus, sleep.

No language of diagnosis or causation, only patterns.

## 3. Behavioral Principles

- Self-tracking + reflection > generic advice.
- Seeing trends over a 1–4 week period encourages trying small changes.
- Non-judgmental language prevents shame and drop-off.

## 4. Personas

1. **Adult individual**
2. **Parent tracking approximate child use and mood**
3. **Student/young adult**

No child accounts; all data recorded by an adult profile.

---

## 5. User Journey

1. User opens the Well-being Dashboard.
2. If insufficient data:
   - See a prompt: “Log how you felt today and your approximate screen time.”
3. User can:
   - Log today’s well-being and screen time.
   - View the dashboard for the last 7, 14, or 30 days.
4. Dashboard shows:
   - For each day: total screen minutes, mood, energy, focus, sleep quality.
   - Simple visual chart (screen time vs mood).
   - Summary patterns: “On days with higher screen time than your chosen goal, your average mood rating was X vs Y on lower-screen days.”
5. User can filter date range and optionally apply a daily goal threshold.

---

## 6. Functional Requirements

### FR1 – Daily Well-being Log

For each date, user can enter:

- `date`: default = today, editable.
- `moodRating`: 1–5
- `energyRating`: 1–5
- `focusRating`: 1–5
- `sleepQualityRating`: 1–5
- `totalScreenMinutes`: integer ≥ 0  
  - If Screen-Time Tracker is installed in the app, this field can be **auto-suggested** from its daily sum, but user can always override.
- Optional `note` (≤ 280 chars).

### FR2 – Validation

- Ratings must be integers 1–5.
- `totalScreenMinutes` ≥ 0 and ≤ 1440.
- Date cannot be more than 30 days in future; otherwise warn & refuse.

### FR3 – Edit & View Logs

- User can view logs in a list or calendar view.
- User can edit or delete a daily log.

### FR4 – Date Range Selection

- Predefined ranges:
  - Last 7 days
  - Last 14 days
  - Last 30 days
- User can change range; charts and summaries recompute.

### FR5 – Optional Daily Goal

- User can enter a reference `goalMinutes` locally in this tool (independent of Screen-Time Tracker).
- For each day, mark whether `totalScreenMinutes` is:
  - `withinGoal` (<= goal)
  - `aboveGoal` (> goal)

### FR6 – Dashboard Metrics

For the selected range:

- Average screen time per day.
- Average mood, energy, focus, sleep.
- Conditional averages:
  - For days above goal: averages of mood/energy/focus/sleep.
  - For days within goal: same metrics.
- Differences summarized in plain text.

### FR7 – Visualizations

At minimum:

- Combined chart:
  - X axis: date
  - Y axis (left): total screen time
  - Y axis (right or normalized): mood or sleep rating
- Optionally allow user to toggle between mood/energy/focus/sleep overlays.

### FR8 – No Hard Dependencies

- Tool must run completely standalone (with its own `totalScreenMinutes` field).
- Optional integration: if a Screen-Time Tracker module exists in the same app, it can **expose a function** to retrieve daily totals, which this tool can import. But that is optional and not required for this PRD.

---

## 7. Logic & Algorithms

### 7.1 Aggregation

For a given date range `[fromDate, toDate]`, build:

```ts
interface DailyWellbeingWithScreen {
  date: string;
  totalScreenMinutes?: number;
  goalMinutes?: number;
  withinGoal?: boolean;
  moodRating?: number;
  energyRating?: number;
  focusRating?: number;
  sleepQualityRating?: number;
}
````

* Use locally stored logs.
* If `goalMinutes` is set, compute `withinGoal`.

### 7.2 Conditional Averages

Group days into:

* `withinGoalDays` = days where `withinGoal === true`.
* `aboveGoalDays` = days where `withinGoal === false` and `totalScreenMinutes` is not undefined.

For each group, compute average mood/energy/focus/sleep.

Example summary text:

> “On days with screen time above your goal, your average sleep quality rating was 2.8. On days within your goal, it was 3.6.”

No statistical tests; keep it descriptive.

---

## 8. Data Model

```ts
export interface WellbeingDailyLog {
  date: string; // "yyyy-mm-dd"
  moodRating?: number; // 1–5
  energyRating?: number; // 1–5
  focusRating?: number; // 1–5
  sleepQualityRating?: number; // 1–5
  totalScreenMinutes?: number; // >= 0
  note?: string;
  updatedAt: string; // ISO datetime
}

export interface WellbeingState {
  logs: WellbeingDailyLog[];
  goalMinutes?: number;
}
```

Storage key: `curosee_wellbeing_state_v1`.

---

## 9. Local Service Contracts

```ts
export function getWellbeingState(): Promise<WellbeingState>;

export function upsertWellbeingLog(
  log: WellbeingDailyLog
): Promise<WellbeingState>;

export function deleteWellbeingLog(
  date: string
): Promise<WellbeingState>;

export function setWellbeingGoalMinutes(
  goalMinutes?: number
): Promise<WellbeingState>;

export interface DailyWellbeingWithScreen {
  date: string;
  totalScreenMinutes?: number;
  goalMinutes?: number;
  withinGoal?: boolean;
  moodRating?: number;
  energyRating?: number;
  focusRating?: number;
  sleepQualityRating?: number;
}

export function getDashboardData(
  fromDate: string,
  toDate: string
): Promise<DailyWellbeingWithScreen[]>;
```

Internal implementation uses only local storage / IndexedDB.
Optional integration with Screen-Time Tracker can be done by updating the `totalScreenMinutes` field from an external service, but that is outside this PRD.

---

## 10. UI Specification

### Views

1. **Daily Well-being Entry**

   * Date picker, rating sliders, `totalScreenMinutes` input, note.
   * Primary CTA: “Save today’s log”.

2. **Dashboard**

   * Date range selector.
   * Chart section.
   * Summary text.
   * Quick stats (cards): average screen time, average mood, etc.

### Components

* `<WellbeingLogForm />`
* `<WellbeingDashboard />`
* `<WellbeingChart />`
* `<WellbeingSummary />`
* `<GoalMinutesForm />`

---

## 11. Technical Stack

* Vite + React + TypeScript + TailwindCSS.
* Storage via small wrapper around IndexedDB or `localStorage`.

---

## 12. Privacy & Safety

* Data is sensitive (mood/sleep). All local.
* No PII, no backend sync in v0.1.
* UI must clearly avoid diagnostic language (“depression”, “disorder”, etc.).

---

## 13. KPIs (future)

* Number of days with well-being logs.
* Number of days with both screen and well-being data.
* Number of visits to dashboard per user.

---

## 14. Roadmap

* v0.1: Manual log + basic dashboard.
* v0.2: Goal logic, within/above goal summary.
* v1.0: Optional integration with Screen-Time Tracker and CSV export.
