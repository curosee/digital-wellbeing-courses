# Product Requirements Document: Family Screen-Time Scheduler / Digital Curfew Planner

## 1. Vision

Enable families to plan and respect screen-free times (e.g. meals, evenings, weekends) through a simple visual “family tech schedule”, promoting shared routines without feeling punitive.

## 2. Problem Definition

Parents struggle to enforce consistent tech rules:

- Rules are unclear (“sometimes we allow devices at dinner, sometimes we don’t”).
- Each family member has different patterns and expectations.
- Conflicts arise when rules are improvised in the moment.

We need a planner that:

- Makes rules explicit, visible, and flexible.
- Focuses on routine and agreement, not control or spying.

## 3. Core Behavioral Principles

- Pre-commitment reduces later conflict (“we already agreed on this schedule”).
- Visual cues (calendar, time blocks) help children understand limits.
- Family participation improves buy-in.
- Predictable routines reduce negotiation fatigue.

## 4. Target Personas

1. **Parent of 1–2 kids (6–14)**  
   Wants a simple, visual tool to schedule screen-free times.

2. **Parent in shared custody or blended family**  
   Needs clarity when children switch homes.

3. **Adult individual / couple**  
   Uses scheduler to protect evenings, focus blocks, or couple time.

## 5. User Journey

1. Parent opens the Scheduler tool.
2. A short explanation appears: “Create your weekly screen routine.”
3. Parent defines:
   - Weekly template (Mon–Sun).
   - Time blocks granularity (30 min or 1 h).
4. Parent adds profiles:
   - Example: “Parent”, “Child A (8–10)”, “Child B (12–14)”, “Whole Family”.
5. For each day of week, they mark blocks as:
   - Screen allowed.
   - Screen limited (e.g. only 1h).
   - Screen-free (“digital curfew”).
6. Tool shows a color-coded weekly grid.
7. Parent saves schedule locally and optionally prints/exports it.
8. Optional: generate a simple “Family Tech Charter” summary.

## 6. Functional Requirements

### FR1 – Profile Management

- User can define 1–6 profiles.
- A profile has:
  - `id` (uuid)
  - `name` (string)
  - `type` (enum: `"adult" | "child" | "family"`)

### FR2 – Weekly Schedule Structure

- Week = 7 days (Mon–Sun).
- Each day has time slots from configurable range (default 06:00–22:00).
- Slot size: 30 or 60 minutes (global choice per schedule).

### FR3 – Slot States

Each time slot has one of:

- `"screen_allowed"`
- `"screen_limited"`
- `"screen_free"`
- `"unspecified"` (default)

Optional: `note` (e.g. “Family movie night”).

### FR4 – Per-Profile Schedules

- Scheduler maintained per profile.
- One schedule per profile (for MVP).
- “Family View” overlays all profile schedules (read-only view).

### FR5 – Editing UX

- User can:
  - Click to set state of a single slot.
  - Click-and-drag to apply a state to multiple adjacent slots.
  - Clear slots for a day.
- Provide simple undo/redo for last action (optional nice-to-have, not required for MVP).

### FR6 – Presets

Provide built-in presets as starting points:

- “School days”:
  - Limited weekdays, more freedom on weekend.
- “Evening curfew”:
  - Screen-free after 20:30.
- “Meals screen-free”:
  - Meal-time blocks set to `screen_free`.

User can save current schedule as a named preset and re-apply later.

### FR7 – Export / Print

- Generate printable view of weekly schedule.
- Generate a plain-text summary:

  > “On weekdays, screens are allowed between 17:00 and 19:30. No screens during meals and after 20:30.”

All export is client-side (no backend).

### FR8 – Data Persistence

- Persist profiles, schedules, presets locally (IndexedDB or `localStorage`).
- Storage key: `curosee_family_scheduler_state_v1`.

### FR9 – Non-Blocking Behavior

- Tool does not control devices.
- It is purely for planning and communication.

## 7. Logic & Algorithms

### 7.1 Time Slot Generation

Given:

- `startHour` (int, e.g. 6 = 06:00)
- `endHour` (int, e.g. 22 = 22:00)
- `slotMinutes` (30 or 60)

Generate slots for each day:

```ts
interface TimeSlot {
  start: string; // "HH:MM"
  end: string;   // "HH:MM"
}
````

### 7.2 Family Overlay Logic

For “Family View”, each visible slot state is derived from all profiles’ states at that time:

Priority ordering:

1. If any profile has `screen_free` → `screen_free`.
2. Else if any has `screen_limited` → `screen_limited`.
3. Else if any has `screen_allowed` → `screen_allowed`.
4. Else → `unspecified`.

## 8. Data Model

### 8.1 TypeScript Interfaces

```ts
export type ProfileType = "adult" | "child" | "family";

export interface FamilyProfile {
  id: string; // uuid
  name: string;
  type: ProfileType;
  createdAt: string;
  updatedAt: string;
}

export type SlotState =
  | "screen_allowed"
  | "screen_limited"
  | "screen_free"
  | "unspecified";

export interface DayScheduleSlot {
  startTime: string; // "HH:MM"
  endTime: string;   // "HH:MM"
  state: SlotState;
  note?: string;
}

export interface DaySchedule {
  dayOfWeek: number; // 0 = Monday ... 6 = Sunday
  slots: DayScheduleSlot[];
}

export interface WeeklySchedule {
  profileId: string;
  slotMinutes: 30 | 60;
  startHour: number;
  endHour: number;
  days: DaySchedule[];
  updatedAt: string;
}

export interface FamilySchedulerState {
  profiles: FamilyProfile[];
  schedules: WeeklySchedule[]; // one per profile
  // Optional: saved presets could be added later
}
```

## 9. Local Service Contracts

```ts
export function getSchedulerState(): Promise<FamilySchedulerState>;

export function upsertProfile(
  profile: FamilyProfile
): Promise<FamilySchedulerState>;

export function deleteProfile(
  profileId: string
): Promise<FamilySchedulerState>;

export function upsertSchedule(
  schedule: WeeklySchedule
): Promise<FamilySchedulerState>;

export function clearSchedulerData(): Promise<void>;
```

No external API required.

## 10. UI Specification

### 10.1 Views

1. **Scheduler Home**

   * List profiles.
   * Buttons: “Add profile”, “Edit schedule”.
   * “View family overlay”.

2. **Weekly Editor**

   * Weekly grid (columns = days, rows = time slots).
   * Color-coded states per slot.
   * Controls:

     * Profile selector.
     * Slot size select (30/60).
     * Time range (start/end).
     * Preset selector.
   * Legend for states.

3. **Family Overlay View**

   * Combined view showing the “most restrictive” state per slot using overlay logic.

4. **Charter Summary View**

   * Read-only text summary of rules.
   * Button to print.

### 10.2 Components

* `<ProfileList />`
* `<ProfileForm />`
* `<WeeklyGrid />`
* `<SlotLegend />`
* `<PresetSelector />`
* `<FamilyOverlayView />`
* `<CharterSummary />`

## 11. Technical Stack

* Vite + React + TypeScript + TailwindCSS.
* State via React Context or Zustand.

## 12. Integration Requirements

* Route: `/tools/family-scheduler`.
* No dependency on other CuroSee tools; can be implemented completely standalone.
* Optional future integration: link from Media Plan Wizard summary (non-technical link only).

## 13. Privacy, Legal & Safety

* No real names required; UI should suggest labels like “Child A”.
* No PII, no backend sync for MVP.
* This is a planning/communication tool, not device control or monitoring.

## 14. KPIs (Future Analytics)

* Number of users defining at least one weekly schedule.
* Number of times per month the schedule page is visited.

## 15. Risks & Mitigations

* **Risk:** Tool is mistaken for device control software.
  Mitigation: clearly state in copy that this is for planning only.

## 16. Release Roadmap

* **v0.1:** Single weekly schedule per profile, local-only.
* **v0.2:** Presets, Family overlay view, Charter summary.
* **v1.0:** Optional online sync and shareable PDF/image export.
