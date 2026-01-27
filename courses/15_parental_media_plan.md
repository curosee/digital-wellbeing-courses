# Product Requirements Document: Parental Control & Media Plan Wizard

## 1. Vision
Help parents create a concrete, age-appropriate **Family Media Plan** — a written agreement defining when, where, and how screens are used. Not surveillance, not app blocking.

## 2. Problem
Parents need clarity:
- Conflicts occur due to inconsistent rules.
- Each child has different needs.
- Co-parents and caregivers apply rules differently.

We provide a wizard that transforms choices into a **clear text plan**.

## 3. Behavioral Principles
- Written commitments outperform verbal rules.
- Age-based guidelines increase fairness.
- Framing as collaboration reduces fights.

## 4. Personas
- Parents of kids 3–17.
- Shared custody households.
- Educators assisting families.

---

## 5. User Journey

1. Parent starts wizard.
2. Defines children profiles (labels + ages).
3. Steps:
   a. Devices & access rules  
   b. Weekday/weekend screen limits  
   c. Screen-free contexts (meals, bedrooms)  
   d. Content rules  
   e. Sleep/bedtime  
   f. Communication & consequences  
4. Plan generated in clear prose.
5. Parent can edit, export, print.

---

## 6. Functional Requirements

### FR1 – Family Setup
- `children[]`: 1–5 child profiles.
- `label`: e.g. `"Child A"`.
- `ageRange`: `"3-5" | "6-9" | "10-12" | "13-17"`.

### FR2 – Devices Rules
- List device types:
  `"smartphone" | "tablet" | "computer" | "tv" | "console" | "other"`
- For each:
  - Access: `"adult_only" | "children_with_permission" | "all"`
  - Location: free text (no PII, ex: “living room drawer”)

### FR3 – Time Limits
Per child age range:
- Weekday daily max (minutes).
- Weekend/holiday daily max.

### FR4 – Screen-Free Zones
Selectable defaults:
- during meals
- bedrooms
- during homework
- family car
- bedtime hour

### FR5 – Content Rules
- Allowed categories (booleans):
  educational, creative, messaging, video, gaming, social.
- Social media start age (optional).
- Rating limits: movie rating, game rating (e.g., PEGI, MPAA).

### FR6 – Sleep/Bedtime
- Screens before bed limit (minutes).
- Device storage location at night.
- Per-child curfew time `"HH:MM"`.

### FR7 – Communication Style
Short free text: how parents will talk about screens respectfully.

### FR8 – Consequence Approach
Short free text: e.g., dialogue + temporary limit.

### FR9 – Media Plan Output
Produce multi-section formatted text:

Sections:
- Introduction
- Devices
- Time limits
- Screen-free zones
- Content rules
- Sleep rules
- Communication & family norms

---

## 7. Logic

### Template-driven
Use string templates with conditional blocks.

Example snippet:
````

If Screen-free during meals:
"Devices are not used during meals so we can connect and rest."

````

No AI generation required.

User can edit generated text.

---

## 8. Data Model

```ts
export type ChildAgeRange =
  | "3-5"
  | "6-9"
  | "10-12"
  | "13-17";

export interface ChildProfile {
  id: string;
  label: string;
  ageRange: ChildAgeRange;
}

export type DeviceType =
  | "smartphone"
  | "tablet"
  | "computer"
  | "tv"
  | "console"
  | "other";

export type DeviceAccess =
  | "adult_only"
  | "children_with_permission"
  | "all";

export interface DeviceRule {
  deviceType: DeviceType;
  access: DeviceAccess;
  mainLocation: string;
}

export interface TimeRule {
  ageRange: ChildAgeRange;
  weekdayMinutes: number;
  weekendMinutes: number;
}

export interface MediaPlanWizardData {
  children: ChildProfile[];
  devices: DeviceRule[];
  timeRules: TimeRule[];
  screenFreeZones: string[];
  contentRules: {
    allowEducational: boolean;
    allowCreative: boolean;
    allowMessaging: boolean;
    allowVideo: boolean;
    allowGaming: boolean;
    allowSocialMedia: boolean;
    socialMediaStartingAge?: number;
    maxMovieRating?: string;
    maxGameRating?: string;
  };
  sleepRules: {
    allowScreensBeforeBedMinutes?: number;
    devicesInBedroomAtNight: boolean;
    offScreenTimesByChild: Record<string, string>;
  };
  communicationApproach: string;
  consequenceApproach: string;
}
````

Local storage key: `curosee_media_plan_v1`.

---

## 9. Local Service

```ts
export function generateMediaPlanText(
  data: MediaPlanWizardData
): string;

export function saveMediaPlan(
  data: MediaPlanWizardData,
  planText: string
): Promise<void>;

export function loadMediaPlan(): Promise<{
  data: MediaPlanWizardData;
  planText: string;
} | null>;
```

No backend required.

---

## 10. UI

Views:

* Wizard (stepper)
* Summary editor screen

Components:

* `<ChildProfilesForm/>`
* `<DevicesForm/>`
* `<TimeRulesForm/>`
* `<ZonesForm/>`
* `<ContentForm/>`
* `<SleepRulesForm/>`
* `<CommunicationForm/>`
* `<PlanEditor/>`

---

## 11. Privacy

* No real names.
* Avoid addresses or school names.
* Everything local to browser.

---

## 12. Roadmap

* v0.1: Wizard + text.
* v0.2: multiple saved plans.
* v1.0: export PDF or shareable image (client-side).
