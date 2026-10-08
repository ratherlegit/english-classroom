# english-classroom: Design Spec

Date: 2026-10-07
Author: Zachary Rosen
Status: Revised after review, awaiting final approval

## Purpose

A new, standalone skill, `english-classroom`, for a teacher running an ESL/EFL class of several students. It is a sibling to the existing `english-tutoring` skill (one-on-one), which explicitly lists group classes as out of scope and says they belong in a separate skill. This skill is that separate skill.

Success looks like this: a teacher answers an interview about the class once, gets a class profile record, and from then on can ask for "the next lesson for [class]" and receive a plan that works for the whole room at once, with built-in differentiation for the level spread and the mix of backgrounds.

## Non-goals

- Changing `english-tutoring` behavior. The only edit is to its description and its out-of-scope paragraph (see "Routing between the two skills").
- Fully per-student plans (that is english-tutoring run N times).
- Parallel lesson tracks for every class. Offered only when the level spread is very wide (see Differentiation).
- A GitHub Pages site. The deliverable is a repo, not a web page.

## Decisions made with the user

| Topic | Decision |
|---|---|
| GitHub target | New public repo `ratherlegit/english-classroom`, packaged as a Claude Code plugin, mirroring `english-tutoring` (README, AGENTS.md, LICENSE, marketplace.json). Created only after the user confirms. |
| Roster depth | Class-level profile with an optional roster. Aggregate info is required. Per-student entries (name, pronouns, level, first language, notes) are optional and can be added later. |
| Mixed-level approach | Anchor level plus tiers. Each lesson targets the class median CEFR level, with Support, Core, and Stretch versions of each activity. Parallel tracks only for very wide spreads. |

## Routing between the two skills

The original `english-tutoring` description matched group phrasing such as "plan a B1 speaking class". It has been narrowed to one-on-one tutoring ("my student", "my tutee") and now ends with "For a class of several students (group classes, mixed levels), use english-classroom instead." Its out-of-scope paragraph now points group requests to `english-classroom`. `english-classroom` has the mirror statement in its description. Test both with ambiguous prompts (see Testing plan).

## Repo layout

```
.claude-plugin/plugin.json
.claude-plugin/marketplace.json
skills/english-classroom/
  SKILL.md
  references/
    class-interview.md
    differentiation.md
    cefr-levels.md
    lesson-frameworks.md
    activity-bank.md
    teacher-activities.md
  scripts/md_to_docx.py
README.md
AGENTS.md
LICENSE
.gitignore
docs/superpowers/specs/   (this spec)
```

`cefr-levels.md`, `lesson-frameworks.md`, `activity-bank.md`, `teacher-activities.md`, and `md_to_docx.py` are copied from `english-tutoring` so the repo installs on its own, then adapted for groups where needed (`teacher-activities.md` and `activity-bank.md` get a group-suitability note per activity and a rotation rule for performance-style activities). `md_to_docx.py` is copied unchanged. Trade-off: the two repos can drift. Accepted for independent installability.

## Records

Stored outside the skill folder so they survive updates, as in english-tutoring:

- Class profile: `~/eng-teaching-records/classes/<class-slug>.md` (lowercase, hyphens)
- Optional roster lives inside the same class file.

The existing `~/eng-teaching-records/<name-slug>.md` files for one-on-one students are never touched, and a class slug cannot collide with them because classes live in the `classes/` subfolder.

### Class record format

```
# Class Record: <Class name>

## Profile
- Setting: school / adult evening / corporate / online / in person
- Number of students: ...
- Age group: kids / teens / adults, with range
- Level spread: lowest CEFR, highest CEFR, median
- Backgrounds: countries and first languages, mono-lingual or mixed
- Shared interests: ...
- Goal: ...
- Session length and frequency: ...
- Activity choice: ...
- Key skills weighting: ...
- Exclusions: ...
- Vocab constraint / count / home assignment / teacher's notes / handout: (asked at first lesson)

## Roster (optional)
| Name | Pronouns | Level | First language | Notes |
|---|---|---|---|---|

## Learning Path (omit if single-lesson only)
| Lesson # | Lesson Name | Theme | Grammar Point | Activity Type(s) | Grouping | Status |
|---|---|---|---|---|---|---|

## Vocabulary Log
### YYYY-MM-DD, Lesson N: <name>
- word
```

## Workflow

### Step 0: Returning class check

If the teacher names a class that has a record, skip the interview. Load the profile, roster, path, and vocab log. Ask a brief check-in: students who joined or left, level changes, anything to adjust. If an active path exists, confirm the next "Not yet delivered" row before generating, and let the teacher request a different theme or activity. If there is no path, offer to design one, clearly optional.

### Step 1: Class interview

One question at a time. Detailed mode (default) or light mode (a few bundled open-ended prompts), as in english-tutoring. Fields, in order:

1. Class name (free text)
2. Setting (school, adult evening, corporate, online, in person)
3. Number of students
4. Age group and range
5. Levels: CEFR spread and median. If unsure, ask what students can do and infer from `cefr-levels.md`
6. Backgrounds: countries and first languages. Determine whether the class is mono-lingual or mixed
7. Shared interests and professions
8. Goal for learning English
9. Session length and frequency
10. Activity choice (random, specific, or show the list first)
11. Single lesson or path; if path, how many lessons (no suggested number)
12. Key skills to focus on
13. Anything to include or exclude
14. Optional roster: offer to add individual students now or later

Light mode bundles these into four prompts and parses answers into the same fields. Lesson-level questions (vocab constraint, vocab count, home assignment, teacher's notes, handout) are not asked until a lesson is about to be written.

Background handling rules (in `class-interview.md`):
- Mono-lingual class: infer the shared L1 from the location and apply the L1-interference cheat sheet to anticipate common errors, surfaced in teacher's notes.
- Mixed class: English-only, culture-sharing activities, no translation-dependent vocab. Teacher's notes give general error guidance only. If the roster records first languages, notes may also flag a few high-frequency L1-specific errors for the named students.

### Step 2: Class learning path

Skipped for a single standalone lesson. Otherwise design the path before any lesson and get approval. Table columns: Lesson #, Lesson Name, Theme, Grammar Point, Activity Type(s), Grouping. Rules:

- Grammar within or at the edge of the median level, varied, simple to complex where natural
- Activities varied and spaced (no repeat within 3 to 4 lessons, never back to back)
- External-material activities rare
- Themes come from shared interests, age group, setting, and goal. Roster interests, if present, are spread across lessons so no one student is the recurring example
- Favor pair, group, and whole-class formats. Limit and rotate activities that need one student to perform alone
- Weight toward key skills, respect exclusions
- Do not generate any lesson until the teacher approves the path and asks for Lesson 1. Never generate all lessons at once
- Save the approved path to the class record with all rows "Not yet delivered", then offer a `.docx` export of the table

### Step 3: Framework

Default PPP for grammar and vocab lessons. Task-based or skills-based frameworks per `lesson-frameworks.md`, adapted for group work.

### Step 4: Lesson plan

Order: Objective, Warm Up Conversation, Vocab Review (Lesson 2 onward), Fresh Vocabulary, Grammar Focus, Activities, Wrap-up, Home Assignment, Materials needed, Time Summary, Teacher's Notes, Student Handout. No student-details or framework header at the top.

Changes from english-tutoring:

- **Warm Up Conversation:** pair or group prompts every student can answer, 2 to 3 fresh bulleted questions per lesson, not one-to-one rapport questions.
- **Fresh Vocabulary:** one CEFR level above the median, count per teacher (default 6 to 10), collocations first, `word – definition` plus `Example:` format, examples tied to shared themes, never repeat words in the class vocab log. Every student gets the same list. The Support tier adds scaffolds for the same words (sentence frames, example hints, a word bank), never a shorter or easier list, so the class learns the same vocabulary.
- **Grammar Focus:** same teaching requirements as english-tutoring (plain-language rule, short formula, several thematic examples, separate examples per distinct use). A quick review box for the Support tier.
- **Activities:** 2 to 3 per lesson, each with a suggested grouping and three versions of the same task (Support, Core, Stretch). Instructions addressed to the students. Item lists are bulleted. Content is generated by default, at most one external-material activity per lesson.
- **Wrap-up:** an exit task that is quick to check across the room (one-sentence ticket, pair check, mini-whiteboard), tied to the objective, not a one-to-one conversation.
- **Teacher's notes (integrated by default, or end-only, or none):** only where they add something. Include group management, monitoring several groups, early finishers, timing with a larger class, and mono-lingual L1 error flags.
- **Student handout:** same as the plan minus Teacher's Notes, Materials needed, and Time Summary. Titled "<Lesson Name> - Student Handout" with no trace of what was removed.
- **Time Summary:** one section; activity options may exceed class length by design.
- After generating, flip the path row to "Delivered <date>" and append the vocabulary to the log.

### Step 5: Differentiation

Defined in `differentiation.md`:

- **Anchor and tiers:** target the median. Support scaffolds (sentence frames, word banks, fewer items, a model answer). Core is the baseline task. Stretch extends (extra constraint, longer output, more abstract content, an error-correction twist).
- **Grouping patterns:** mixed-level pairs for peer help, same-level trios or groups for stretch and support tasks, whole-class for warm-up and wrap-up, with a rotation suggestion.
- **Wide spread:** if the spread exceeds two CEFR bands (for example A2 to C1), tell the teacher plainly and offer a parallel-tracks option for that lesson (two or three content bands) in place of tiers. Default stays tiers unless the teacher chooses otherwise.
- **Class size notes:** small (up to 6), medium (7 to 15), large (16 or more) adjust the number of groups, the monitoring approach, and which activities are practical.
- **Age adaptations:** kids, teens, adults differ in tone, topic, activity types, and attention span.
- Avoid more than one new grammar point per lesson or more than 8 to 10 vocab items. Exam-prep lessons tie each activity to the exam skill.

### Step 6: Export

Optional `.docx` export of a lesson, handout, or path table, using the copied `md_to_docx.py`, with the same environment-aware delivery rules as english-tutoring (verify a reachable save location, send or attach after running the script, name the files and where to find them, plain-markdown fallback when no code execution). File names: `<class-slug>-lesson-<n>.docx`, `<class-slug>-lesson-<n>-handout.docx`, `<class-slug>-learning-path.docx`.

## Output rules

- Never use an em dash in narrative or body text in anything the skill produces. Allowed only as a separator in headings and labels where english-tutoring allows it, and vocab items use an en dash.
- Plans are clean markdown with a header per stage, no timing in headers.
- After creating any file, tell the teacher exactly where it landed.

## Frontmatter

```
name: english-classroom
description: Develops lesson plans and learning paths for teaching English to a class of multiple ESL/EFL students (group classes, mixed levels and backgrounds). Use when the user asks to plan a class, group lesson, or course for several English learners, e.g. "plan a lesson for my class of 12 B1 students" or "set up my evening English class". For one student at a time, use english-tutoring instead.
```

## Testing plan

No automated test suite exists for skills. Verification is by exercise:

1. Run the interview for three sample classes: a small mono-lingual B1 teen class, a medium mixed-L1 adult class, and a wide-spread class (A2 to C1) to confirm the parallel-tracks offer triggers.
2. Confirm the class record is written to `~/eng-teaching-records/classes/` and read back correctly in a fresh session (returning class check-in, no re-interview).
3. Generate a path, approve it, generate Lesson 1 and Lesson 2. Check: Lesson 2 has a Vocab Review drawn from the log, no repeated vocab, every activity has Support, Core, and Stretch versions and a grouping, wrap-up is room-checkable, no em dashes, handout contains no teacher-only sections.
4. Export one lesson and one handout to `.docx` and confirm the files exist and open.
5. Confirm `english-tutoring` still works for one-on-one requests and its records are untouched.
6. Routing check with ambiguous prompts ("plan a B1 speaking class", "lesson on conditionals for my 10 students", "next lesson for Maria"). Group phrasing must route to `english-classroom`, one-student phrasing to `english-tutoring`.

## Publishing

Create `ratherlegit/english-classroom` as a public repo, push, and verify the plugin installs from the marketplace. This is outward-facing, so it happens only after the user explicitly confirms, after the skill has been built and tested locally.

## Open items

These are defaults chosen during spec writing. They are easy to change in review:

- Wide-spread threshold: more than two CEFR bands (for example A2 to C1) triggers the parallel-tracks offer.
- Class size bands: small up to 6, medium 7 to 15, large 16 or more.
- Records live in `~/eng-teaching-records/classes/` so they cannot collide with one-on-one student files.
- Roster interests are spread across lessons so no one student is the recurring example.
- Shared references are copied, not linked, so the two repos can drift over time. Accepted for independent installability.

Implementation plan comes next, after spec approval.
