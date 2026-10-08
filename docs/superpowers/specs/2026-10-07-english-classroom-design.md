# english-classroom: Design Spec

Date: 2026-10-07
Author: Zachary Rosen
Status: Approved; amended after final whole-branch review (see Amendments)

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
| Mixed-level approach | Anchor level plus tiers. Each lesson targets the class median CEFR level, with Support, Core, and Stretch versions of each activity. Parallel tracks only for very wide or bimodal spreads. |
| Under-18 rosters | Name is first name plus last initial, extended with more surname letters and then a number to resolve collisions. No surnames, birthdates, addresses, photos, diagnoses, or family details. Adult rosters unchanged. |

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
- Level headcount: students per CEFR level (for example A2: 6, B2: 6)
- Median level: from the headcount (the lower of the two if it falls between bands)
- Distribution: single level / spread / bimodal
- Level confidence: confirmed / estimated
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

## Previous Paths (optional)
### Completed YYYY-MM-DD
(the old path table)

## Vocabulary Log
### YYYY-MM-DD, Lesson N: <name>
- word
```

### Roster rules

- Pronouns are asked, never inferred from a name (they/them until told).
- **Under-18 classes (kids and teens):** the name is the first name plus the last initial ("Maria G."). Ask for the last initial only, never a full surname, and never ask for birthdates, addresses, or photos. If two students would share a label, extend the initial with more letters of the surname ("Maria Ga." and "Maria Gr."), and if that still collides, add a number ("Maria G. 2"). Notes are limited to learning needs in plain terms ("needs extra time", "needs larger print"). Never write diagnoses, family situations, or behavior notes to the record. If the teacher volunteers them, use them in the session only where they help. A class with any under-18 student follows this rule for the whole roster. Adult classes use whatever names the teacher gives.

## Workflow

### Step 0: Returning class check

If the teacher names a class that has a record, skip the interview. Load the profile, roster, path, and vocab log. Ask a brief check-in: students who joined or left, level changes, anything to adjust. A roster change never resets the path or the vocab log. If an active path exists, confirm the next "Not yet delivered" row before generating, and let the teacher request a different theme or activity. If every row is delivered, say the path is complete and offer a new path (the finished path moves to Previous Paths, and the roster and vocab log carry over so words are not repeated) or single lessons. If there is no path, offer to design one, clearly optional.

### Step 1: Class interview

One question at a time. Detailed mode (default) or light mode (a few bundled open-ended prompts), as in english-tutoring. Fields, in order:

1. Class name (free text)
2. Setting (school, adult evening, corporate, online, in person)
3. Number of students
4. Age group and range
5. Levels: how many students are at each CEFR level (a headcount). Compute the median (the lower of the two bands if it falls between) and the distribution (single level, spread, or bimodal). If the teacher is unsure, ask what students can do, infer levels from `cefr-levels.md`, and mark the levels as estimated
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

Interview rules (in `SKILL.md`):
- Class of one: if the number of students is 1, point to english-tutoring and offer to switch.
- Duplicate class name: if the slug matches an existing record, say it already exists and offer three options: the same class (load it), a new term for the same class (keep the roster and vocab log, move the old path to Previous Paths, start a new path), or a different class (choose another name). Never overwrite a record.
- Kids and teens: keep all content age-appropriate, honor exclusions, and follow the roster rules above.
- Bimodal: two clusters of at least a quarter of the class each, two or more CEFR bands apart, with the bands between them holding fewer students than either cluster. Flag it and offer parallel tracks even when the total spread is only two bands.

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

Order: Objective, Warm Up Conversation, Quick Level Check (Lesson 1 only, when levels are estimated), Vocab Review (Lesson 2 onward), Fresh Vocabulary, Grammar Focus, Activities, Wrap-up, Home Assignment, Materials needed, Time Summary, Teacher's Notes, Student Handout. No student-details or framework header at the top.

Changes from english-tutoring:

- **Warm Up Conversation:** pair or group prompts every student can answer, 2 to 3 fresh bulleted questions per lesson, not one-to-one rapport questions.
- **Quick Level Check (Lesson 1 only, when levels are estimated):** a short diagnostic in pairs (a few can-do prompts and one written line) placed after the warm-up, with a teacher's note on what to observe. After delivering Lesson 1, ask the teacher what they observed and update the headcount, median, distribution, and confidence in the record.
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
- **Wide or bimodal spread:** if the spread exceeds two CEFR bands (for example A2 to C1), or the class is bimodal (for example half at A2 and half at B2, where no single anchor level fits), tell the teacher plainly and offer a parallel-tracks option for that lesson (two or three content bands) in place of tiers. Default stays tiers unless the teacher chooses otherwise.
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
7. Bimodal class (6 students at A2, 6 at B2): the skill flags the class as bimodal, says no single anchor level fits, and offers parallel tracks.
8. Completed path: with every row delivered, a returning teacher is told the path is complete and offered a new path that keeps the roster and vocab log.
9. Under-18 roster with two students who would both be "Maria G.": the skill extends the label, never asks for a full surname, birthdate, address, or photo, and writes no diagnoses or family details to the record.
10. Estimated levels: Lesson 1 includes a Quick Level Check, and the skill asks for observations afterward and updates the record.
11. Duplicate class name: the skill offers the three options (same class, new term, different class).

## Publishing

Create `ratherlegit/english-classroom` as a public repo, push, and verify the plugin installs from the marketplace. This is outward-facing, so it happens only after the user explicitly confirms, after the skill has been built and tested locally.

## Open items

These are defaults chosen during spec writing. They are easy to change in review:

- Wide-spread threshold: more than two CEFR bands (for example A2 to C1) triggers the parallel-tracks offer.
- Class size bands: small up to 6, medium 7 to 15, large 16 or more.
- Records live in `~/eng-teaching-records/classes/` so they cannot collide with one-on-one student files.
- Roster interests are spread across lessons so no one student is the recurring example.
- Bimodal means two clusters of at least a quarter of the class each, two or more CEFR bands apart, with the bands between them holding fewer students than either cluster. It is a judgment aid, not a formula the skill must compute exactly.
- Under-18 rosters use first name plus last initial, extended with more surname letters and then a number to resolve collisions (user decision).
- Shared references are copied, not linked, so the two repos can drift over time. Accepted for independent installability.

Implementation plan comes next, after spec approval.

## Amendments after the final whole-branch review

These refine the sections above and are binding where they differ.

1. **Returning teachers:** the under-18 roster rule and the pronoun rule apply to every student added or edited in a Step 0 check-in, not only during the Step 1 interview. A changed headcount or level triggers recomputing the median and distribution.
2. **Under-18 sensitive details:** if the teacher volunteers a diagnosis, family situation, or behavior detail, the skill describes the need rather than the diagnosis in any plan, and never puts it in a handout or exported file. If a surname is given anyway, only the letters the label needs are stored.
3. **Parallel tracks:** a lesson with parallel tracks is the one exception to "one vocabulary list for the whole class": each track gets its own list pitched one level above that track's band, and every list goes into the Vocabulary Log. Grammar is then taught once per track at that track's band. The teacher's choice (tiers with an anchor band, or parallel tracks) is saved in a `Differentiation` line of the Profile and reused in later sessions.
4. **Previous Paths:** when a record already has a Learning Path and a new path is approved (a completed path, or a new term), the old table moves unchanged under `## Previous Paths` as `### Completed <date>` (or `### Ended <date>` if it still had undelivered rows). The roster, profile, and Vocabulary Log are kept.
5. **Quick Level Check:** applies to the first lesson written for the class while levels are estimated. Because the lesson is taught after the session ends, the skill asks for results in chat, never inside the plan or handout, and asks again at the next Step 0 check-in. Level confidence becomes confirmed once the teacher reports levels from what they saw.
6. **Vocab Review** runs whenever the class Vocabulary Log has entries, and is skipped when it is empty.
7. **Student handout** is offered by default regardless of the teacher's-notes setting.
8. **Never overwrite a record:** before writing a new record file, the skill confirms the path does not exist. If it cannot list or read the folder, it asks the teacher whether the class already has a record.
9. **Mixed classes:** every lesson includes at least one culture-sharing prompt or task.
10. **Path rows track reality:** if a lesson's theme, grammar point, activities, or grouping ends up different from the approved path row, the row is updated and the teacher is told.
11. **Exports** are never saved inside `~/eng-teaching-records/`. A `.docx` converter path relative to the repo root only works from the repo; in an installed plugin the script is run from the skill's own folder.
12. **Publishing hygiene:** published files contain no absolute local paths and no real student names or data. History is squash-merged before publishing so earlier drafts of the test log never ship.
