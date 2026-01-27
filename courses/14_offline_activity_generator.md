# Product Requirements Document: Offline Activity Generator

## 1. Vision
Provide fun, offline alternatives to screens by generating 3–6 simple activities tailored to time, energy, location, and participants.

Goal = eliminate “we don’t know what to do instead”.

## 2. Problem
Families and individuals often default to devices because they lack ideas for offline activities that feel realistic in their current moment.

This tool removes decision fatigue and encourages low-prep, real-world engagement.

## 3. Behavioral Principles
- Reduce choice overload → curated 3–6 suggestions.
- Match activities to **context** (energy, time, indoor/outdoor).
- Engage kids with achievable tasks they enjoy.

## 4. Personas
- Family with children (4–14).
- Adults unplugging at night or weekend.
- Couples seeking shared evenings.

## 5. User Journey
1. User opens tool.
2. Quick form: time, energy, participants, setting.
3. Click “Generate activities”.
4. See 3–6 activity cards.
5. Save favorites.
6. Optional: mark “done”.

No backend.

---

## 6. Functional Requirements

### FR1 – Input Form
- `who`: `"self" | "family" | "kids_only"`
- `kidsAgeRanges?`: array `"3-5" | "6-9" | "10-12" | "13-17"`
- `timeAvailableMinutes`: 15 | 30 | 60 | 120
- `energyLevel`: `"tired" | "normal" | "energetic"`
- `setting`: `"home_indoors" | "outdoor" | "mixed"`

### FR2 – Local Activity Catalog
Static JSON or TS constant.

Each activity:
- `title`
- `description`
- `minMinutes`, `maxMinutes`
- `suitableFor`: Who array
- `energyLevelTags`
- `settingTags`
- `prepLevel`: `"no_prep" | "light_prep" | "needs_materials"`

### FR3 – Matching Logic
Filter list using:

- who match
- time bounds
- energy overlap
- setting overlap

Random sample 3–6 from result.

### FR4 – Favorites
Local storage: ID list.

### FR5 – Optional Completed Today
Local mark; resets daily optional.

---

## 7. Algorithm

Simplest approach:
````

matchActivities(criteria):
filter by suitableFor
filter by time
filter by setting
sort by energy similarity
randomize
return top 3–6

````

Energy similarity = intersection count between criteria.energyLevel and activity.energyLevelTags.

---

## 8. Data Model

```ts
export type WhoInvolved = "self" | "family" | "kids_only";

export type KidsAgeRange =
  | "3-5"
  | "6-9"
  | "10-12"
  | "13-17";

export type EnergyLevel = "tired" | "normal" | "energetic";

export type SettingType = "home_indoors" | "outdoor" | "mixed";

export type PrepLevel =
  | "no_prep"
  | "light_prep"
  | "needs_materials";

export interface Activity {
  id: string;
  title: string;
  description: string;
  minMinutes: number;
  maxMinutes: number;
  suitableFor: WhoInvolved[];
  energyLevelTags: EnergyLevel[];
  settingTags: SettingType[];
  prepLevel: PrepLevel;
}

export interface ActivityCriteria {
  who: WhoInvolved;
  kidsAgeRanges?: KidsAgeRange[];
  timeAvailableMinutes: 15 | 30 | 60 | 120;
  energyLevel: EnergyLevel;
  setting: SettingType;
}

export interface OfflineActivityState {
  favorites: string[]; // ids
  completedToday: string[]; // ids
}
````

Local storage key: `curosee_offline_activity_v1`.

---

## 9. Local Service

```ts
generateActivities(criteria: ActivityCriteria): Activity[];
loadFavorites(): Promise<string[]>;
saveFavorites(ids: string[]): Promise<void>;
markCompleted(id: string): Promise<void>;
```

---

## 10. UI Specification

Views:

* Generator page.
* Result cards grid.
* Favorites page.

Components:

* `<ActivityCriteriaForm/>`
* `<ActivityCard/>`
* `<ActivityResultGrid/>`
* `<FavoritesView/>`

---

## 11. Privacy

No PII.
No exports required.
Static content only.

---

## 12. Roadmap

* v0.1: static catalog, favorites.
* v0.2: profiles, completed log.
* v1.0: dynamic catalog, shareable sets.
