# Product Requirements Document: Screen-Time Log & Habit Tracker

## 1. Vision

Help individuals and families become aware of their real screen-time habits, set gentle limits, and track progress over time – without guilt, shaming, or intrusive tracking. The tool should feel like a wellness journal for screen use, not surveillance.

## 2. Problem Definition

Most people underestimate or misjudge their screen time. They either:

- Don’t know how much time is spent on which device/app.
- Feel overwhelmed and guilty but lack a simple system to change habits.
- Use built-in screen reports (iOS/Android) but find them noisy, hard to interpret, or not aligned with their goals.

We need a simple, user-friendly tracker where users can log screen time by device/app and visualize trends and goals over time.

## 3. Core Behavioral Principles (Evidence-based)

- Self-monitoring increases behavior change: merely tracking increases awareness and reduces excessive use.
- Small, achievable goals work better than strict bans.
- Visual feedback supports habit change (graphs, streaks, gentle milestones).
- Non-judgmental framing reduces shame and drop-off.

## 4. Target Personas (Matrix)

1. **Adult Individual**
   - Age: 25–50
   - Goal: Reduce doom-scrolling, protect focus and sleep.

2. **Parent**
   - Age: 30–55
   - Goal: Track child’s screen exposure and gradually improve balance.

3. **Teen / Young Adult (via parent)**
   - Age: 12–20
   - Goal: Better balance between social media, school, and rest.

No direct child account storage; all usage is logged under a parent/adult profile with optional child labels (e.g. “Child A (8–10)” only).

## 5. User Journey (Step-by-step)

1. User lands on Screen-Time Tracker page.
2. Sees a short explanation and options:
   - Add today’s screen-time log.
   - View history (last 7 / 30 days).
   - Set or update a daily goal.
3. User logs screen time for current day:
   - Select date (default = today).
   - Add one or more entries:
     - Device (phone, tablet, computer, TV, other).
     - Category (social, work, learning, entertainment, gaming, other).
     - Duration in minutes.
     - Optional note.
4. User saves log:
   - Data stored locally (IndexedDB or localStorage).
   - UI updates day summary and weekly graph.
5. User can define a daily total screen-time goal and optionally per-category goals.
6. Dashboard shows:
   - Today vs goal.
   - Last 7/30 days trend.
   - Streak of days within goal.
7. User can edit or delete past entries.
8. Optional: User can export data as JSON/CSV (client-side only).

## 6. Functional Requirements

### FR1 – Daily Log Creation

- User can create a screen-time log for any date (default = today).
- A log consists of one or more entries.

### FR2 – Entry Structure

Each entry includes:

- `date`: ISO date string `yyyy-mm-dd`
- `deviceType`: enum  
  `"phone" | "tablet" | "computer" | "tv" | "console" | "other"`
- `category`: enum  
  `"social" | "work" | "school" | "learning" | "entertainment" | "gaming" | "other"`
- `minutes`: integer (1–1440)
- `profileLabel` (optional): string (e.g. `"Self"`, `"Child A (8–10)"`)
- `note` (optional): short string up to 200 chars

### FR3 – Validation

- `minutes` > 0 and ≤ 1440.
- Total minutes per day per profile should not block saving, but if > 16 hours, display a gentle “check for errors” hint.
- Date cannot be in the future by more than 7 days (to prevent accidental wrong-year inputs).

### FR4 – View Daily Summary

For each date, show:

- Total minutes.
- Breakdown per category.
- Goal comparison (if goal set).

### FR5 – Goals

- User can set:
  - `dailyTotalGoalMinutes` (integer between 30 and 1440).
  - Optional `categoryGoals: { [category]: minutes }`.
- Store goals per profile (`"Self"`, `"Child A"`, etc.).

### FR6 – Trend Visualization

- Show a 7-day and 30-day trend:
  - X-axis: date.
  - Y-axis: total minutes.
  - Indicate goal line (if any).

### FR7 – Streaks

Compute:

- `currentStreakDaysWithinGoal` – consecutive days from most recent date backward where total ≤ goal.
- `longestStreakDaysWithinGoal` – max streak historically.

### FR8 – Edit / Delete Entries

- User can edit any entry fields.
- User can delete an entry.
- Daily totals and streaks update accordingly.

### FR9 – Data Persistence

- Store all data locally (IndexedDB or `localStorage`).
- Namespace storage so it doesn’t collide with other CuroSee tools.
- Provide a “Reset all data” button with confirmation modal.

### FR10 – Optional Export

- Local-only export to JSON and/or CSV.
- No backend calls required.

## 7. Logic & Algorithms

### 7.1 Streak Calculation

Given:

- `logsByDate: { [date: string]: totalMinutesForThatDate }`
- `dailyGoalMinutes: number`

Algorithm (high-level):

1. Sort dates ascending.
2. For `currentStreak`:
   - Start from latest date and iterate backward.
   - If `totalMinutes <= dailyGoalMinutes`, increment streak; otherwise break.
3. For `longestStreak`:
   - Traverse all dates from oldest to newest, maintaining a streak counter and storing the maximum.

### 7.2 Trend Data Aggregation

For current 7-day view:

- Generate an array of last 7 dates.
- For each date, compute sum of `minutes` for all entries on that date for the selected profile.

Same for 30-day view.

### 7.3 Goal Comparison

Per date:

- `difference = totalMinutes - dailyGoalMinutes`.
- Flag status: `"below"`, `"at"`, or `"above"` depending on sign of `difference`.

## 8. Data Model

### 8.1 TypeScript Interfaces

```ts
export type DeviceType =
  | "phone"
  | "tablet"
  | "computer"
  | "tv"
  | "console"
  | "other";

export type ScreenCategory =
  | "social"
  | "work"
  | "school"
  | "learning"
  | "entertainment"
  | "gaming"
  | "other";

export interface ScreenTimeEntry {
  id: string; // uuid
  date: string; // ISO date yyyy-mm-dd
  deviceType: DeviceType;
  category: ScreenCategory;
  minutes: number;
  profileLabel?: string; // e.g. "Self" or "Child A (8-10)"
  note?: string;
  createdAt: string; // ISO datetime
  updatedAt: string; // ISO datetime
}

export interface ScreenTimeGoals {
  profileLabel: string; // "Self" etc.
  dailyTotalGoalMinutes?: number;
  categoryGoals?: Partial<Record<ScreenCategory, number>>;
  updatedAt: string;
}

export interface ScreenTimeState {
  entries: ScreenTimeEntry[];
  goals: ScreenTimeGoals[];
}
````

### 8.2 Storage Strategy

* Use IndexedDB via a small wrapper (recommended), or fallback to `localStorage`.
* Suggested key: `curosee_screen_time_state_v1`.
* Store entire `ScreenTimeState` JSON.

## 9. “API” Contracts (Local Service Layer)

No backend for v0.1.
Expose a local service that *looks* like an API so you can later swap in a real backend without changing the rest of the app.

```ts
// screenTimeService.ts
export function getState(): Promise<ScreenTimeState>;

export function upsertEntry(entry: ScreenTimeEntry): Promise<ScreenTimeState>;

export function deleteEntry(id: string): Promise<ScreenTimeState>;

export function setGoals(goals: ScreenTimeGoals): Promise<ScreenTimeState>;

export function clearAllData(): Promise<void>;
```

If you later add a backend (Supabase/Node), you can keep this interface and change the internals only.

## 10. UI Specification

### 10.1 Pages / Views

1. **Main Screen-Time Dashboard**

   * Today’s total vs goal.
   * Quick add form (device, category, minutes).
   * 7-day mini chart.
   * Streak summary.

2. **History View**

   * Calendar or list of days.
   * Each day: total minutes, color-coded vs goal.
   * Click to expand details.

3. **Goals Settings**

   * Form for total daily goal and per-category goals.
   * Separate per profile label.

4. **Settings**

   * Profile labels management (“Self”, “Child A”).
   * Reset data.
   * Export data.

### 10.2 Components (React + TS)

* `<ScreenTimeDashboard />`
* `<DaySummaryCard />`
* `<ScreenTimeEntryForm />`
* `<ScreenTimeHistoryList />`
* `<GoalsForm />`
* `<StreakSummary />`
* `<TrendChart />` (use a lightweight chart lib or simple SVG)
* `<ConfirmResetModal />`

### 10.3 UX / Tone

* Neutral, soft colors.
* Avoid alarming red except for obvious data entry mistakes.
* Use supportive language, e.g. “Today you spent 3h20 on screens. How do you feel about this?” rather than “Too much!”.

## 11. Technical Stack

* Frontend: Vite + React + TypeScript + TailwindCSS.
* State Management: React hooks + Context or lightweight store (Zustand) – your choice.
* Data Persistence: IndexedDB wrapper or `localStorage`.
* Charts: simple chart library or small custom SVG components.
* Testing (optional): Vitest or Jest.

## 12. Integration Requirements

* Implement as a standalone route: `/tools/screen-time-tracker`.
* Other tools may optionally *read* aggregated daily totals from this module in the future, but this PRD does **not** require any other tool to exist.

## 13. Privacy, Legal & Safety

* No account required.
* No PII collected; profile labels should be free text but UI must suggest “avoid using full names”.
* All data stays on user’s device unless exported manually.
* No external analytics required for v0.1.

## 14. KPIs & Outcome Metrics (for future analytics)

* Number of days with logged entries per user.
* Number of days users stay within goal.
* Retention: number of distinct weeks with usage.

## 15. Risks & Mitigations

* **Risk:** User feels judged and abandons tool.
  Mitigation: supportive copy, no “failure” wording, no “red alert” UI for normal use.

* **Risk:** Data loss on device reset.
  Mitigation: export feature; later add optional account sync.

## 16. Release Roadmap

* **v0.1 (MVP)**

  * Local log, simple dashboard, daily goal, 7-day trend.

* **v0.2**

  * 30-day view, streaks, per-category goals.

* **v0.3**

  * Export (JSON/CSV), better charts, profile labels UX.

* **v1.0**

  * Optional sync with parental account backend.
  * Optional integration with Well-being Dashboard (screen-time as input).
