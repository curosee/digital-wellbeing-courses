# Product Requirements Document: Digital Detox Challenge Generator

## 1. Vision

Offer structured, realistic digital detox challenges (e.g. “no social media between 19:00–22:00 for 7 days”) that users can start quickly and track easily, encouraging experimentation rather than rigid abstinence.

## 2. Problem Definition

Users want breaks from screens but:

- Overestimate difficulty of full “detox”.
- Lack structure or accountability.
- Forget challenges after 1–2 days.

We help them design **short, focused challenges** with tracking and gentle feedback.

## 3. Behavioral Principles

- Time-limited challenges feel approachable.
- Tracking completion builds motivation (streaks).
- Partial success still counts; avoid perfectionism.

## 4. Personas

- Adults wanting to reduce evening doom-scrolling.
- Parents doing family weekend detox.
- Students aiming for focused study periods.

---

## 5. User Journey

1. User opens Detox Challenges page.
2. Sees explanation and a button: “Create a new challenge”.
3. Wizard asks:
   - Challenge type.
   - Start date.
   - Duration days.
   - Time block(s) for detox (optional).
4. Tool generates challenge summary.
5. For each day, user can mark:
   - Completed / Partial / Skipped.
6. At any time, user views:
   - Progress bar.
   - Completion rate.
   - Streak stats.

---

## 6. Functional Requirements

### FR1 – Challenge Types

Predefined types:

- `"phone_free_block"` – e.g. no phone 19:00–22:00.
- `"social_media_free"` – no social media all day or certain times.
- `"screen_free_meals"` – no devices during meals.
- `"custom"` – user-defined.

### FR2 – Challenge Config

Each challenge has:

- `title` (auto-generated & editable).
- `type`: enum above.
- `startDate`: string `yyyy-mm-dd`.
- `durationDays`: integer 1–30.
- `timeBlocks?`: record of label → `"HH:MM-HH:MM"` (e.g., `"evening": "19:00-22:00"`).
- `description`: a full sentence/paragraph summary.

### FR3 – Challenge Day State

For each day:

- `date`: string `yyyy-mm-dd`.
- `status`: `"not_started" | "completed" | "partial" | "skipped"`.
- `note?`: optional.

### FR4 – Multiple Challenges

- User can have multiple challenges.
- Display “active” vs “completed” vs “upcoming”.

### FR5 – Challenge Summary Metrics

Per challenge:

- Completion rate: percentage of days with `status === "completed"`.
- Partial days count.
- Longest streak of consecutive completed days.

### FR6 – Persistence

All data stored locally. No backend.

---

## 7. Logic & Algorithms

### 7.1 Generating Days

When challenge is created:

- Build an array of `durationDays` starting at `startDate`.
- Initialize each day with `status: "not_started"`.

### 7.2 Updating Status

When user updates a given date’s status:

- Update corresponding `DetoxChallengeDay`.
- Recalculate metrics lazily or on demand.

### 7.3 Metrics

Pseudocode:

```ts
function computeChallengeStats(challenge: DetoxChallenge) {
  const completedDays = challenge.days.filter(d => d.status === "completed").length;
  const partialDays = challenge.days.filter(d => d.status === "partial").length;
  const totalDays = challenge.days.length;

  const completionRate = totalDays > 0
    ? Math.round((completedDays / totalDays) * 100)
    : 0;

  let longestStreak = 0;
  let currentStreak = 0;
  for (const day of challenge.days) {
    if (day.status === "completed") {
      currentStreak++;
      longestStreak = Math.max(longestStreak, currentStreak);
    } else {
      currentStreak = 0;
    }
  }

  return { completionRate, partialDays, longestStreak };
}
````

---

## 8. Data Model

```ts
export type ChallengeType =
  | "phone_free_block"
  | "social_media_free"
  | "screen_free_meals"
  | "custom";

export type ChallengeDayStatus =
  | "not_started"
  | "completed"
  | "partial"
  | "skipped";

export interface DetoxChallengeDay {
  date: string; // yyyy-mm-dd
  status: ChallengeDayStatus;
  note?: string;
}

export interface DetoxChallengeConfig {
  id: string; // uuid
  title: string;
  type: ChallengeType;
  startDate: string; // yyyy-mm-dd
  durationDays: number; // 1–30
  timeBlocks?: Record<string, string>; // label -> "HH:MM-HH:MM"
  description: string;
  createdAt: string;
}

export interface DetoxChallenge extends DetoxChallengeConfig {
  days: DetoxChallengeDay[];
}

export interface DetoxState {
  challenges: DetoxChallenge[];
}
```

Storage key: `curosee_detox_state_v1`.

---

## 9. Local Service Contracts

```ts
export function getDetoxState(): Promise<DetoxState>;

export function createChallenge(
  input: Omit<
    DetoxChallengeConfig,
    "id" | "createdAt"
  >
): Promise<DetoxState>;

export function updateChallengeDayStatus(
  challengeId: string,
  date: string,
  status: ChallengeDayStatus,
  note?: string
): Promise<DetoxState>;

export function deleteChallenge(
  challengeId: string
): Promise<DetoxState>;
```

Internally, IDs are generated (UUID) and state stored in local storage.

---

## 10. UI Specification

### Views

1. **Challenge List**

   * Cards for each challenge showing:

     * Title, type, dates.
     * Completion rate and current streak badge.
   * CTA: “New challenge”.

2. **New Challenge Wizard**

   * Step 1: Choose type.
   * Step 2: Start date + duration.
   * Step 3: Time blocks (for applicable types).
   * Step 4: Auto-generated summary (editable).

3. **Challenge Detail**

   * Challenge description.
   * List or small calendar showing day statuses.
   * Controls for today’s status:

     * Completed / Partial / Skipped.
   * Stats section (completion rate, longest streak).

### Components

* `<DetoxChallengeList />`
* `<DetoxChallengeCard />`
* `<DetoxChallengeWizard />`
* `<DetoxChallengeDetail />`
* `<DetoxDayStatusSelector />`

---

## 11. Technical Stack

* Vite + React + TypeScript + TailwindCSS.
* Data persisted locally.

---

## 12. Privacy & Safety

* No PII.
* Copy must normalize imperfection (“Partial progress is still progress.”).
* No shaming language.

---

## 13. Roadmap

* v0.1: Single challenge creation, manual status updates.
* v0.2: Multiple challenges + metrics.
* v1.0: Optional export summary (e.g. share how it felt, not public ranking).
