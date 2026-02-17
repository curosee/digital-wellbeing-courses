# CLAUDE.md — Project Guide for AI Assistants

## Project Overview

This repository contains open-source digital wellbeing courses served through the CuroSee platform. Courses are written in YAML (metadata) and Markdown (content) and synced to the website via GitHub Actions.

## Repository Structure

```
digital-wellbeing-courses/
├── README.md                    # Project overview and course catalog
├── CONTRIBUTING.md              # Contribution guidelines
├── CLAUDE.md                    # This file — AI assistant guide
├── agents.md                    # Agent workflow documentation
├── .github/
│   └── workflows/
│       └── trigger-sync.yml     # Auto-sync to CuroSee on push to main/master
├── courses/
│   ├── 01_why_is_my_kid_glued/          # Parent L1
│   ├── 02_screen_smart_families/         # Parent L2
│   ├── 03_raising_humans_digital_world/  # Parent L3
│   ├── 04_reclaim_your_attention/        # Self L1
│   ├── 05_rewire_your_day/              # Self L2
│   ├── 06_the_attention_architect/       # Self L3
│   ├── 07_your_phone_not_your_boss/      # Youth L1
│   ├── 08_level_up_irl/                 # Youth L2
│   ├── 09_be_the_change/               # Youth L3
│   ├── 10_someone_i_love_lost_in_screens/ # Supporter L1
│   ├── 11_words_that_open_doors/         # Supporter L2
│   └── 12_the_wise_ally/                # Supporter L3
└── research/                            # Evidence base documents
```

## Four Pathways

| Pathway | Persona | Levels | Course IDs |
|---------|---------|--------|------------|
| Parent | `parent` | 3 | p1, p2, p3 |
| Self | `self` | 3 | s1, s2, s3 |
| Youth | `youth` | 3 | y1, y2, y3 |
| Supporter | `supporter` | 3 | h1, h2, h3 |

## Course File Format

Each course has two files:

### YML File (metadata)
Required fields: `id`, `slug`, `title`, `subtitle`, `persona`, `level`, `estimated_duration_hours`, `goals`, `intended_outcomes`, `gamification`, `modules`, `resources`, `gamma_prompt`

### MD File (content)
Full lesson text with exercises, challenges, and resources. Uses Markdown with clear heading hierarchy.

## ID Conventions

- Course: `{pathway_prefix}{level}` → `p1`, `s2`, `y3`, `h1`
- Module: `{course_id}-m{index}` → `p1-m1`, `s2-m3`
- Lesson: `{course_id}-m{module}-l{lesson}` → `p1-m1-l1`
- Challenge: `{course_id}-m{module}-ch1` → `p1-m1-ch1`
- Badge: `{course_id}-m{module}` with unique `code`

## Writing Style

- **Tone**: Warm, direct, personal. Like a wise friend, not a lecturer.
- **Sentences**: Short. Translatable. No idioms.
- **Judgment**: Zero. Screens meet real needs. Imbalance is the problem.
- **Evidence**: Cite research but keep it human.
- **Voice**: "You" and "your child/friend/partner" consistently.
- **Emojis**: Never in course content.
- **Cultural sensitivity**: Avoid culture-specific references. Acknowledge diverse family structures.

## Key Principles

1. Never shame screen use — understand the needs it meets
2. Evidence-informed but accessible — no jargon walls
3. Practical over theoretical — every lesson has an action
4. Progressive autonomy — move from rules to self-regulation
5. Balance, not elimination — screens are part of modern life
6. Self-compassion — setbacks are normal and expected

## When Editing Courses

- Maintain the existing YML schema exactly
- Keep module 1 of each course `is_free_without_login: true`
- Each module needs a challenge with badge reward
- XP rules: lesson_read: 5, exercise_completed: 10, challenge_completed: 15
- Estimated times should be realistic (30–50 minutes per module)
- Always include a `gamma_prompt` section for slide generation

## Deployment

Pushing to `main` or `master` triggers a GitHub Actions workflow that notifies the CuroSee platform to sync course content.
