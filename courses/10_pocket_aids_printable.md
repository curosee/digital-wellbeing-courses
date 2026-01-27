# PRD: Pocket Aids – Printable, Portable Content Sets

## 1. Product Overview

Pocket aids are **small, printable card sets** with:

- Short sentences.
- Simple prompts.
- Quotes and micro-practices.

They help users in the *exact moments* where screen habits are hardest to manage (waiting, commuting, stress peaks).

They are:

- A **standalone free product** (not tied to courses).
- Organized by persona:
  - Parents.
  - Self.
  - Supporters.
- Available as:
  - downloadable PDFs (A4 sheets with cut lines),
  - optional “one-pager” versions (foldable).

## 2. Personas and Sets

For each persona, offer at least **5 themed sets**.

### 2.1 Parents – 5 sets

1. **Set P1 – Calm Limit Setter**
   - Focus: sentences to use when taking screens away or saying no.
   - Tone: firm and kind.
2. **Set P2 – Guilt and Self-Compassion**
   - Focus: reduce parental guilt, support emotionally.
3. **Set P3 – Conflict De-escalation**
   - Focus: in-the-moment scripts when child is angry.
4. **Set P4 – Role Model Reminder**
   - Focus: parent’s own screen habits and example.
5. **Set P5 – Connection First**
   - Focus: small ideas to reconnect offline with child.

### 2.2 Self – 5 sets

1. **Set S1 – Urge Surfing**
   - Short steps to ride an urge without acting.
2. **Set S2 – Gentle Self-Talk**
   - Self-kindness when slipping.
3. **Set S3 – Micro-Alternatives**
   - Tiny things to do instead of scrolling.
4. **Set S4 – Focus Boosters**
   - Prompts before starting work or study.
5. **Set S5 – Sleep Protectors**
   - Prompts before bedtime.

### 2.3 Supporters – 5 sets

1. **Set H1 – Conversation Starters**
   - Sentences to open a talk.
2. **Set H2 – Boundary Lines**
   - Simple ways to state limits.
3. **Set H3 – De-Escalation**
   - In-the-moment calming lines.
4. **Set H4 – Self-Protection**
   - Reminders about supporter’s needs.
5. **Set H5 – Help-Seeking**
   - Prompts for deciding when to seek external help.

## 3. Content Format

### 3.1 Card Specification

- Size: A7 or similar (around 74 x 105 mm) when cut from A4.
- Each card:
  - Title (1–3 words).
  - 2–4 short lines of text (max ~120 characters total).
- Language: simple, direct, no idioms.

Example card (P1):

> **Title:** After “time’s up”  
> Text:  
> “I see you want to continue.  
> We agreed on this limit.  
> Let’s choose what to do next together.”

### 3.2 PDF Layout

Best approach for printing and portability:

- **A4 portrait** with **8 cards per page**:
  - 2 columns x 4 rows.
  - Thin cut marks between cards.
- Front side:
  - Titles + main texts.
- Optional back side:
  - For quotes or checklists.
- Use:
  - High-contrast,
  - large font,
  - no heavy backgrounds.

Recommended pipeline:

1. Write content in Markdown with a clear structure:

   ```md
   ## Card P1-01
   Title: After “time’s up”
   Persona: parent
   Set: P1 Calm Limit Setter
   Side: front
   Lines:
   - I see you want to continue.
   - We agreed on this limit.
   - Let’s choose what to do next together.
````

2. Generate HTML + CSS with a template that:

   * maps each card to a `div` sized for A7.
   * places them in a CSS grid on an A4 sheet.
3. Use headless browser or PDF generator (e.g. Puppeteer) to create PDFs.

## 4. Access Rules

* Pocket aids page lists all sets.
* Anonymous users:

  * Can view card content on screen.
  * Can download PDFs freely.
* Registered users:

  * Same rights; optionally:

    * can mark favorite sets,
    * can track which sets they use often.

No gating needed for MVP.

## 5. Data Model

```ts
export type Persona = "parent" | "self" | "supporter";

export interface PocketSet {
  id: string;        // "P1", "S3", etc.
  persona: Persona;
  code: string;      // "P1", "S2" etc.
  title: string;     // "Calm Limit Setter"
  description: string;
  cards: PocketCard[];
  printablePdfUrl?: string;
}

export interface PocketCard {
  id: string;       // "P1-01"
  title: string;    // "After 'time’s up'"
  lines: string[];  // up to 4 short lines
}
```

Storage key for favorites (client-side):
`curosee_pocket_favorites_v1`.

## 6. Example Sets with Sample Content

### 6.1 Parents – Set P1 Calm Limit Setter (sample 3 cards)

**Card P1-01**
Title: After “time’s up”
Lines:

* I see you want to continue.
* We agreed on this limit.
* Let us choose what to do next together.

**Card P1-02**
Title: Not a punishment
Lines:

* This is not a punishment.
* It is a way to protect your sleep and your brain.
* That is my job as a parent.

**Card P1-03**
Title: Not forever
Lines:

* I am not saying “never”.
* I am saying “not now”.
* We can talk about more time later.

(Extend to 8–12 cards per set in implementation.)

### 6.2 Self – Set S1 Urge Surfing (sample 3 cards)

**Card S1-01**
Title: Pause 30 seconds
Lines:

* I notice this urge to check my phone.
* I will wait 30 seconds.
* I just observe my body and thoughts.

**Card S1-02**
Title: Name the feeling
Lines:

* What am I feeling now?
* Bored, stressed, lonely, tired, or something else?
* I say the word quietly to myself.

**Card S1-03**
Title: Decide on purpose
Lines:

* If I use my phone now, it is a choice.
* I choose what I do with my attention.
* I can also choose not to.

### 6.3 Supporter – Set H2 Boundary Lines (sample 3 cards)

**Card H2-01**
Title: My limit
Lines:

* I care about you.
* I also need rest and respect.
* I cannot accept this behavior.

**Card H2-02**
Title: Clear request
Lines:

* I need us to talk without shouting.
* If voices rise, I will pause the talk.
* We can continue later when we are calmer.

**Card H2-03**
Title: Protecting myself
Lines:

* Your feelings matter.
* My feelings matter too.
* I am allowed to protect my time and energy.

## 7. UI and UX

* Pocket aids landing page:

  * filter by persona,
  * preview of sets (title, 2 sample cards).
* Set detail page:

  * all cards listed.
  * “Download print sheet” button.
* Simple icons:

  * parent, person, two people.

## 8. Non-Functional

* PDFs must be:

  * small enough (< 2 MB) for mobile download.
  * printer-friendly (black text on white background).
