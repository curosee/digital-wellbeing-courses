# agents.md — Agent Workflows for Course Development

## Overview

This document describes how AI agents and human contributors work together to develop, review, and maintain digital wellbeing course content in this repository.

---

## Agent Roles

### Content Creator Agent
**Purpose:** Generate new course content or expand existing outlines into full lessons.

**Workflow:**
1. Read existing courses to understand tone, structure, and schema
2. Research the topic using peer-reviewed sources and expert frameworks
3. Draft YML metadata following the established schema
4. Write MD content with full lessons, exercises, and challenges
5. Ensure gamification elements (badges, XP, challenges) are included
6. Submit changes for human review

**Constraints:**
- Follow the writing style guide in CLAUDE.md
- Never fabricate statistics — cite real research or omit
- Keep lessons 5–10 minutes of reading each
- Every lesson must have a practical exercise or reflection
- Every module must have a hands-on challenge

### Content Reviewer Agent
**Purpose:** Review course content for quality, accuracy, and tone.

**Review Checklist:**
- [ ] Tone is warm, non-judgmental, and empowering
- [ ] No shame-based or fear-based messaging
- [ ] Statistics and citations are accurate and current
- [ ] Exercises are practical and culturally inclusive
- [ ] YML schema is valid and complete
- [ ] ID conventions are followed correctly
- [ ] Gamification elements are present and consistent
- [ ] Content is translatable (short sentences, no idioms)
- [ ] Module 1 is marked as free without login
- [ ] Estimated times are realistic

### Research Agent
**Purpose:** Find and synthesize evidence to support course content.

**Workflow:**
1. Search for peer-reviewed studies, meta-analyses, and expert guidelines
2. Prioritize recent research (last 3–5 years)
3. Note key statistics, researcher names, and practical implications
4. Flag areas where evidence is mixed or limited
5. Store findings in `research/` directory

**Sources to prioritize:**
- Peer-reviewed journals (JAMA, Pediatrics, JMIR, Cyberpsychology)
- Health organization guidelines (WHO, AAP, AACAP)
- Research centers (Center for Humane Technology, Center for Digital Thriving at Harvard)
- Expert books (Lembke, Alter, Heitner, Weinstein, Haidt)

### Translation Readiness Agent
**Purpose:** Ensure content is ready for translation into multiple languages.

**Checks:**
- Sentences are short and grammatically simple
- No culture-specific idioms or references
- Consistent terminology throughout (glossary adherence)
- Lists and bullet points preferred over long paragraphs
- Headings are clear and descriptive

---

## Workflow: Creating a New Course

```
1. Research     → Gather evidence and expert frameworks
2. Outline      → Draft YML with modules, lessons, and gamification
3. Write        → Create full MD content for all lessons
4. Review       → Check tone, accuracy, structure, and inclusivity
5. Test         → Verify YML schema validity
6. PR           → Submit pull request for human review
7. Merge        → Human approves and merges to main
8. Sync         → GitHub Actions triggers platform sync
```

## Workflow: Updating Existing Content

```
1. Identify     → Flag outdated stats, broken tone, or missing content
2. Research     → Find current evidence
3. Edit         → Update specific sections (minimal changes preferred)
4. Review       → Verify accuracy and tone consistency
5. PR           → Submit with clear description of what changed and why
```

---

## Quality Standards

### Must Have
- Every claim backed by research or clearly marked as practical guidance
- Every lesson has a clear learning goal
- Every module has a practical challenge
- Tone review passes (no shame, no fear, no judgment)

### Should Have
- Links to further reading or expert resources
- Printable worksheet suggestions
- Gamma slide prompts for visual learners

### Nice to Have
- Video script suggestions
- Podcast discussion prompts
- Community discussion questions

---

## File Naming Conventions

- Course folders: `{number}_{slug}` (e.g., `01_why_is_my_kid_glued`)
- YML files: `{number}.yml` (e.g., `01.yml`)
- MD files: `{number}_{slug}.md` (e.g., `01_why_is_my_kid_glued.md`)
- Research files: `{topic}_{year}.md` in `research/` directory
