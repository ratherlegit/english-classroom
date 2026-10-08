---
name: english-classroom
description: Develops lesson plans and learning paths for teaching English to a class of multiple ESL/EFL students (group classes, mixed levels and backgrounds). Use when the user asks to plan a class, group lesson, or course for several English learners, e.g. "plan a lesson for my class of 12 B1 students" or "set up my evening English class". For one student at a time, use english-tutoring instead.
---

# ESL Class Lesson Plans

Build classroom-ready English lesson plans for a class of several ESL/EFL learners. Adapt structure, depth, and format to what the teacher asks for. Do not force a template they did not request.

**This skill covers group classes.** For a single student, the english-tutoring skill is the better fit. If the teacher describes one learner, say so and offer to switch (see Step 1).

Records are stored in `~/eng-teaching-records/classes/`, separate from one-on-one student files in `~/eng-teaching-records/`. Never read or modify the one-on-one files.

## Step 0: Check for a returning class

Before interviewing anyone, check whether the teacher names a class you already have a record for, such as "next lesson for my Tuesday class". List `~/eng-teaching-records/classes/` and look for `<class-slug>.md` (lowercase, hyphens for spaces). If the teacher's wording could match more than one record, ask which. If you cannot list or read the folder, say so and ask the teacher whether this class already has a record. Create `~/eng-teaching-records/classes/` if it does not exist.

Records live outside the skill folder on purpose: the profile, roster, path, and vocab history are personal data that must survive skill updates and reinstalls.

- **If a record exists**: skip Step 1 entirely. Load the profile, roster, path, and vocab log. Ask a brief check-in only: did students join or leave, did any levels change, anything else to adjust. Update the roster and profile with any changes, applying the Step 1 roster rules (Under-18 rosters, Pronouns) to every student added or edited. If the number of students or the levels changed, recompute the median and distribution. If Level confidence is estimated and a lesson with a Quick Level Check was written, ask what the teacher observed. A roster change never resets the path or the vocab log.
  - **If there is an active path**: remind the teacher what is next (the next "Not yet delivered" row: lesson name, theme, grammar point, grouping) and confirm it is still wanted before generating. Let them ask for something different (another theme or activity, skip ahead).
  - **If every row is delivered**: say the path is complete and offer a new path (the finished path moves to a Previous Paths section in the record, and the roster and vocab log carry over so words are not repeated) or single lessons.
  - **If there is no path**: offer to design one, clearly optional. If they decline, build a single lesson.
  - Then go to Step 3 and Step 4 (or to Step 2 first if the teacher wants a new path).
- **If no record exists**: run the full Step 1 interview, then create the record.

## Step 1: Interview the teacher about the class

Ask about the class before drafting. Do not skip this for a one-off "just give me a lesson on X" request, unless Step 0 found a record.

**First, ask whether they want a detailed or light interview, detailed by default.** Detailed asks each item as its own question. Light bundles the same information into four open-ended prompts. If the teacher states no preference, use detailed. The full question set, the reasons behind each question, the light mode prompts, and the roster rules are in `references/class-interview.md`.

### Detailed mode

Ask these as **separate questions**, in this order:
1. **Class name**, free text typed by the teacher. Never offer suggested names.
2. **Setting**: school, adult evening class, corporate, online, or in person.
3. **Number of students.**
4. **Age group and range**: kids, teens, or adults.
5. **Levels**: a headcount of how many students are at each CEFR level (for example A2: 6, B2: 6). Compute the median (the lower of the two bands if it falls between) and the distribution: single level, spread, or bimodal. If the teacher is unsure, ask what students can do, infer levels from `references/cefr-levels.md`, and mark the levels as estimated.
6. **Backgrounds**: countries and first languages. Determine whether the class is mono-lingual or mixed.
7. **Shared interests and professions.**
8. **Goal for learning English.**
9. **Session length and frequency**: offer 30, 60, 90 minutes, or something else.
10. **Activity choice**: random, specific activities the teacher names, or show me the list first (names only from `references/teacher-activities.md`, then re-ask).
11. **Single lesson or learning path.** If a path, ask how many lessons as the very next question. Never suggest or default to a number.
12. **Key skills to focus on.**
13. **Anything to include or exclude**: topics to avoid, a coursebook to align with, grammar already mastered or specifically requested.
14. **Optional roster**: offer to add individual students now or later.

### Light mode

Four open-ended prompts, one at a time (the exact wording is in `references/class-interview.md`), then the roster offer. Parse each answer into the same fields as detailed mode and ask a one-off follow-up for anything missing instead of guessing.

### Interview rules

- **A class of one**: if the number of students is 1, say that this is a one-on-one case, that english-tutoring fits better, and offer to switch. Continue here only if the teacher insists.
- **A class name that already exists**: if the slug of a new class name matches an existing record, say the record already exists and offer three options: the same class (load it, Step 0), a new term for the same class (keep the roster and vocab log, move the old path to Previous Paths, and design a new path), or a different class that needs another name. Never overwrite a record: before writing a new record file, confirm the path does not exist (for example by trying to read it). If you cannot check, ask the teacher before writing.
- **Kids and teens**: keep every topic and activity age-appropriate, and honor every exclusion the teacher gives.
- **Under-18 rosters**: the Name column is the first name plus the last initial ("Maria G."). Ask for the last initial only, never a full surname (if the teacher gives one anyway, store only the letters the label needs), and never ask for birthdates, addresses, or photos. If two students would share a label, extend the initial with more letters of the surname ("Maria Ga." and "Maria Gr."), and if that still collides, add a number ("Maria G. 2"). Notes are limited to learning needs in plain terms ("needs extra time", "needs larger print"). Never write diagnoses, family situations, or behavior notes to the record. If the teacher volunteers them, use them in the session only where they help, describe the need rather than the diagnosis in any plan, and never put them in a handout or exported file. A class with any under-18 student follows this rule for the whole roster. Adult classes use whatever names the teacher gives.
- **Unknown, uniform, or bimodal levels**: if the teacher cannot give levels, ask what students can do, infer levels, and mark them estimated. If every student is at a single level, record that and note that Support, Core, and Stretch tiers still apply (see Step 5). If the class is bimodal (two clusters of at least a quarter of the class each, two or more CEFR bands apart, with the bands between them holding fewer students than either cluster, for example half A2 and half B2), say that no single anchor level fits and offer parallel tracks (Step 2). If the teacher keeps tiers anyway, ask which band to anchor on. Save the choice in the Differentiation line of the Profile.
- **Mono-lingual vs mixed**: infer the shared first language from the location for a mono-lingual class and use the L1 interference notes in `references/class-interview.md` silently. For a mixed class, keep activities English-only, with no translation-dependent vocab, and include at least one culture-sharing prompt or task per lesson (for example "How does this work in your country?").
- **Pronouns for roster students** are asked, never inferred from a name.
- Do not ask about vocab preferences or home assignments in this round. Those are asked right before a lesson is written (Step 4).

### The class record

At the end of Step 1, create `~/eng-teaching-records/classes/<class-slug>.md`:

```
# Class Record: <Class name>

## Profile
- Setting: ...
- Number of students: ...
- Age group: ...
- Level headcount: A2: 6, B2: 6
- Median level: ... (lower of the two if between bands)
- Distribution: single level / spread / bimodal
- Level confidence: confirmed / estimated
- Backgrounds: ... (mono-lingual / mixed)
- Shared interests: ...
- Goal: ...
- Session length and frequency: ...
- Activity choice: ...
- Key skills weighting: ...
- Differentiation: tiers (anchor: ...) / parallel tracks (bands, lessons)
- Exclusions: ...
- Vocab constraint: ...
- Vocab count: ...
- Home assignment: yes/no
- Teacher's notes: integrated/end-only/none
- Student handout: yes/no

## Roster (optional)
| Name | Pronouns | Level | First language | Notes |
|---|---|---|---|---|

## Learning Path (omit if single-lesson only)
| Lesson # | Lesson Name | Theme | Grammar Point | Activity Type(s) | Grouping | Status |
|---|---|---|---|---|---|---|
| 1 | ... | ... | ... | ... | Mixed-level pairs | Delivered 2026-10-07 |
| 2 | ... | ... | ... | ... | Same-level trios | Not yet delivered |

## Previous Paths (omit if none)
### Completed 2026-12-15
(the old path table)

## Vocabulary Log
### 2026-10-07, Lesson 1: <name>
- word one
- word two
```

The vocab, home assignment, teacher's notes, and handout fields stay blank until Step 4 asks about them.

## Step 2: Design the class learning path

Skip this step only when the teacher asked for a single standalone lesson. Whenever more than one lesson is requested, design the full path **before** writing any lesson, and share it for approval.

Output the path as a table:

| Lesson # | Lesson Name | Theme | Grammar Point | Activity Type(s) | Grouping |
|---|---|---|---|---|---|
| 1 | ... | ... | ... | ... | ... |

Rules:
- **Grammar** stays within, or just at the edge of, the median level (use `references/cefr-levels.md`). Vary it across lessons and sequence simple to complex where there is a natural order.
- **Activities** come mainly from `references/teacher-activities.md`. Vary them, and do not repeat one within 3 to 4 lessons or back to back. List 2 to 3 per lesson.
- **Keep external-material activities rare.** Default to generated, self-contained activities. At most one external-material activity per lesson, and no more than one every few lessons across the path.
- **Group formats**: favor pair, group, and whole-class activities. Limit activities where one student performs alone for the whole room, and rotate them.
- **Themes** come from what the class shares: common interests, the age group, the setting, and the goal. If a roster records individual interests, spread them across lessons so no one student is the recurring example.
- **Grouping** column: state the grouping pattern for the lesson (mixed-level pairs, same-level trios, whole class). Patterns are in `references/differentiation.md`.
- **Weight toward the key skills** and **respect exclusions**.
- **Wide or bimodal spread**: if the spread is wider than two CEFR bands (for example A2 to C1), or the class is bimodal (for example half at A2 and half at B2, so no single anchor level fits), tell the teacher plainly and offer parallel tracks for the lessons where tiers would stretch too far. Tiers remain the default unless the teacher chooses parallel tracks. Save the choice in the Differentiation line of the Profile. See `references/differentiation.md`.

**Do not generate any lesson immediately after presenting the path.** Ask whether they want adjustments or want Lesson 1 generated. Only then proceed to Steps 3 and 4 for Lesson 1. Never generate all lessons at once, and ask before each later lesson, using that row's theme, grammar point, activities, and grouping as the spec. Do not swap any of them without telling the teacher.

Once the teacher approves the path, save it into the class record with every Status set to "Not yet delivered". If the record already has a Learning Path (a completed path, or a new term), first move the old table unchanged under `## Previous Paths` as `### Completed <today's date>` (use `### Ended <today's date>` if it still had undelivered rows), and keep the roster, profile, and Vocabulary Log as they are. Then offer to export the path table as a `.docx` (Step 6).

## Step 3: Pick a framework

Default to **PPP (Presentation, Practice, Production)** for grammar and vocab lessons, unless the teacher asks for something else, the goal is task-based (exam prep, projects), or the lesson is skills-focused (a reading or listening lesson fits pre-, during-, and post-task). See `references/lesson-frameworks.md` for the options and the group adjustments.

## Step 4: Build the plan

If the lesson comes from an approved path, use that row's theme, grammar point, activities, and grouping as given.

### Lesson-specific questions (ask now, not during Step 1)

Before drafting, if not already answered for this class, ask:
- **Vocab constraint**: phrasal verbs, a part of speech, or no constraint (default: mixed, chosen to fit the topic).
- **Number of vocab words or phrases** (default 6 to 10).
- **Home assignment**: include one? (default yes).
- **Teacher's notes**: integrated (a short note under the relevant stage), end-only (one closing section), or none. Default integrated.
- **Student handout**: a copy for students without Teacher's Notes, Materials needed, and Time Summary. Default yes.

Ask these once. For a path, ask before Lesson 1 and reuse the answers unless the teacher wants a change. Save the answers to the Profile in the class record.

**Do not open the plan with a class-details or framework header.** Start straight in with the lesson content.

Unless the teacher specifies their own format, include in this order:
1. **Objective(s)**: a can-do statement ("Students will be able to...").
2. **Warm Up Conversation**
3. **Quick Level Check** (first lesson only, when levels are estimated)
4. **Vocab Review** (whenever the class Vocabulary Log has entries)
5. **Fresh Vocabulary**
6. **Grammar Focus**
7. **Activities** (2 to 3, each with a suggested grouping and three versions)
8. **Wrap-up**
9. **Home Assignment** (unless the teacher opted out)
10. **Materials needed**
11. **Time Summary**
12. **Teacher's Notes** (if not turned off)
13. **Student Handout** (a second copy without Teacher's Notes, Materials needed, and Time Summary, unless the teacher opted out)

### Time Summary

Timings are suggestions the teacher can flex. Do not put a time next to each heading. Collect all suggested timings in one Time Summary section near the end. Non-activity stages should sum close to the class length. The activity options can add up to more than what remains: that is intentional headroom, not an error.

### Warm Up Conversation

Placed right after the Objective. A short, genuine check-in the whole class can answer in pairs or groups: 2 to 3 open-ended questions about their lives, in the spirit of "Any highlights from your week?" or "What are you looking forward to this weekend?". Write fresh questions for every lesson and vary the angle. Format as a bulleted list, and carry that format into the Student Handout. These are not lesson-topic teasers and not one-to-one rapport questions: every student must be able to answer regardless of level.

### Quick Level Check

Only in the first lesson written for this class while the record says levels are estimated. Place it right after the Warm Up Conversation. A short diagnostic of about 5 minutes, done in pairs: 3 to 4 can-do prompts at ascending difficulty (for example "Describe your morning.", "Explain a problem at work or school and what you did about it.", "Give your opinion on a topic and defend it.") and one written line the teacher can collect. Add a Teacher's Note on what to observe (who stalls at which prompt, which structures break down). The lesson is taught after this session, so ask for the results in chat, never inside the plan or handout: invite the teacher to report back, and ask again at the next Step 0 check-in. Then update the headcount, median, distribution, and level confidence (confirmed once the teacher reports levels from what they saw, still estimated if they are unsure). The Teacher's Note says what to observe and never addresses the assistant.

### Vocab Review (recycling)

Before new vocabulary, recycle old words:
- Pull 2 to 4 items from the class Vocabulary Log. Prioritize items from lessons that are neither the most recent nor already reviewed several times, so review rotates through everything taught.
- Use a quick format: a sentence with the word blanked out, or "use this word in a sentence about..." tied to the class's shared interests. Run it in pairs.
- Skip this section when the Vocabulary Log is empty.

### Fresh Vocabulary

Every lesson gets a standalone new-vocabulary section:
- Use the count the teacher specified (default 6 to 10), tied to the lesson's theme and the class's shared interests. Examples must read as written for this class.
- **Pitch vocab one CEFR level above the class median.** This is a deliberate stretch (use `references/cefr-levels.md`).
- **One list for the whole class.** The Support tier adds scaffolds for the same words (sentence frames, example hints, a word bank), never a shorter or easier list. The one exception is a lesson where the teacher chose parallel tracks: each track gets its own list, pitched one level above that track's band, and every list goes into the Vocabulary Log (see `references/differentiation.md`).
- Apply the vocab constraint from the interview.
- Lead with collocations, not isolated definitions.
- **Never repeat a word already taught to this class.** Check the Vocabulary Log first and swap out any overlap.
- A mixed class gets no translation-dependent items.

**Format**, every item exactly:
```
word – definition
Example: example sentence
```
Use an en dash between word and definition. The example must connect to the lesson theme and the class's shared interests.

### Grammar Focus

The lesson's main teaching point, taught once at the median level (or at the anchor band the teacher chose; with parallel tracks, once per track at that track's band). The presentation always includes:
- **A clear, plain-language explanation of the rule**: what it means and when a speaker would use it.
- **A formula, if the structure has one, kept as short as possible** (for example *have/has* + past participle). Skip it for grammar that is not formulaic. Do not add parentheticals.
- **Several thematic examples** tied to the lesson topic and the class's interests.
- **Separate examples for each distinct use or form** if the grammar point has several. If the lesson is deliberately scoped to one use, say so briefly.
- A short **review box** of the prerequisite structure for the Support tier, and a short **extension** (a further use or a contrast) for the Stretch tier.

Keep it tight. Edge cases and register notes belong in a Teacher's Note, not in the student-facing explanation. Practice that follows builds on the examples already given.

### Activities

Include **2 to 3 activities**, not one fixed activity, so the teacher can choose, skip, or swap live:
- **Random**: pick 2 to 3 from `references/teacher-activities.md` that fit the level, topic, time, and class size, varied enough not to repeat each other.
- **Specific**: use the named activities. If the teacher named only one, add 1 to 2 more.
- **Teacher's judgment or unspecified**: start from `references/teacher-activities.md`, and fall back to `references/activity-bank.md` only if nothing there fits.

**Every activity has**:
- A suggested **grouping** (mixed-level pairs, same-level trios, whole class). Name the pairs and groups when a roster exists.
- **Three versions of the same task**: **Support** (scaffolds such as sentence frames, a word bank, fewer items, or a model answer), **Core** (the baseline), **Stretch** (an extra constraint, longer output, or more abstract content). Rules are in `references/differentiation.md`.

**Generate almost all content yourself**: dialogues, stories, scenarios, discussion questions, data tables. At most **1 activity per lesson** may require the teacher to source real external material (a real photo, video or audio clip, or rarely a real article), and only when it is clearly the best fit. Describe a Picture needs a real photo, so it always counts. If used, list the photo in Materials needed and add a Teacher's Note suggesting what to look for.

Present each option under its own heading (Activity A, Activity B, Activity C) with no wrapping stage heading, going straight from Grammar Focus into the first activity. No time on the headings. Do not write framing like "choose from these activities". It is fine for the activities' combined time to exceed the class length.

Any list of items within an activity is a **bulleted list**. Numbered lists are for sequential steps only.

Write every activity's instructions **addressed to the students, in the second person** ("Read the dialogue and discuss..."), not as stage directions to the teacher. Calibrate language to the median level. Personalize using the class's shared interests, goal, and place. For a mono-lingual class, pre-empt the likely L1 errors silently.

### Wrap-up

Close with an **exit task** that is quick to check across the room: a one-sentence ticket, a pair check, a mini-whiteboard answer, or a written line the teacher collects. It must be specific and tied to the lesson's content, a real check of the objective, not "summarize what you learned". For example: "Write one sentence about a film you would recommend to a friend, using the passive voice and one new vocabulary word."

### Teacher's Notes

For the teacher only, never addressed to students. Keep each note to 1 to 2 sentences. Cover things the teacher could not infer from the heading: group management, how to monitor several groups, early finishers (point to the Stretch version), timing with a large class, which activity to choose if time is short, and, for a mono-lingual class, the likely L1 error for this grammar point. A note earns its place only if it adds something. Skip self-explanatory stages.
- **Integrated** (default): a brief *Teacher's note:* line under the stages that have one.
- **End-only**: one closing "Teacher's Notes" section with a short bullet per stage worth flagging.
- **Not included**: omit entirely. Do not sneak pedagogical asides into stage descriptions.

### Student Handout

Unless the teacher opted out, produce a second copy after the main plan, whatever the teacher's-notes setting. It is the same document, same wording, same formatting, minus Teacher's Notes, Materials needed, and Time Summary (with notes set to none, it simply omits Materials needed and Time Summary). Label it with just the lesson name plus "Student Handout". Do not add any caption describing what was removed. If exporting to `.docx`, export the plan and the handout as separate files.

### Updating the class record

After finalizing a lesson:
- Set its path row Status to "Delivered <date>".
- If the lesson's theme, grammar point, activities, or grouping ended up different from the path row, update the row to match what was written and tell the teacher, so later lessons avoid repeats.
- When the teacher reports what the Quick Level Check showed, update the level headcount, median, distribution, and confidence.
- **Before** drafting Fresh Vocabulary, read the Vocabulary Log. **After** finalizing the list, append the new items under a new dated heading.
- If the teacher says the class already knows certain words, log those too.

## Step 5: Differentiation and common pitfalls

The full rules are in `references/differentiation.md`. In short:
- **Anchor and tiers**: target the median. Every activity has Support, Core, and Stretch versions of the same task. Fresh vocab is one list for everyone, never a shorter list (per track, if the teacher chose parallel tracks).
- **Grouping**: mixed-level pairs for peer help, same-level trios for Support and Stretch tasks, whole class for warm-up and wrap-up. Rotate partners and who goes first.
- **Wide or bimodal spread**: more than two CEFR bands, or a bimodal class, triggers an offer of parallel tracks. Tiers stay the default.
- **Single level**: if all students are at a single level, tiers still apply as scaffolding and extension. If levels are only estimated, the first lesson starts with a Quick Level Check.
- **Class size**: small (up to 6), medium (7 to 15), large (16 or more) change the number of groups, the monitoring approach, and which activities are practical.
- **Age**: kids, teens, and adults differ in tone, topics, activity length, and what is age-appropriate.
- **Pitfalls**: no more than one new grammar point per lesson or about 8 to 10 new vocab items. For exam prep, tie every activity to the exam skill being tested.

## Output format

Write the plan as clean markdown with a header per stage. No timing in headers. Do not pad with teaching theory the teacher already knows. Keep it concrete and usable at the front of a classroom.

**Never use an em dash in narrative or body text in anything this skill produces**: lesson plans, handouts, learning paths, teacher's notes, interview questions, record files, and chat responses. It reads as AI-generated writing. Use a period, comma, colon, or parentheses, or split into two sentences. The only acceptable use is as a separator in a heading or label, and vocab items use an en dash. When in doubt, do not use one.

## Step 6: Export as a .docx (optional)

A finished lesson plan, a student handout, or an approved path table is a document the teacher will print, edit, or share. After presenting one, offer to export it as a `.docx`. Do not export automatically. Build the file only once the teacher says yes or asks for a Word file up front.

**Many teachers using this skill are not technical, so file delivery must not be a place they get stuck.** Verify it, in this order:

1. **Check whether this environment has a place to write files the teacher can reach.** Some chat harnesses only hand over files created through their own attachment mechanism, or when a folder is connected for the session. A file written by a script is not guaranteed to reach the teacher. Do not assume a previous session's connection carried over. Check first.
2. **If there is a working connected location**, ask where to save (unless the teacher already said), reuse that answer for the session, and write the file there. Never save exports inside `~/eng-teaching-records/`, which holds records only.
3. **If there is no connected location, or you cannot confirm one, say so in plain language** and share the file directly in the chat instead. Do not fail silently.
4. **After creating the file, confirm it exists and is reachable by the teacher** before saying it is done. A script exiting without error is not proof.

**If the environment has a dedicated Outputs or Files panel** (for example Claude's Cowork-style task view): a file the assistant writes directly shows up there, but a file produced by running a script does not. After running the converter, always send or attach each generated file explicitly. In the same message, name every file exactly and tell the teacher where to find it ("click **Outputs** in the panel on the right").

**If the environment has no way to run code or a shell** (a plain chat assistant with no tools): say plainly that this environment cannot generate a Word file, and do not claim to have created one. Output the content as clean, well-structured markdown (proper `#` and `##` headings, bold, bullet and numbered lists) and tell the teacher to copy and paste it into Word or Google Docs. If a code interpreter tool is available, try the conversion instead.

**Default, on any harness with shell access**: write the plan to a markdown file, then run the converter:

```bash
python3 skills/english-classroom/scripts/md_to_docx.py <plan>.md <output>.docx
```

That path is relative to the repo root. In an installed plugin the working directory is the teacher's own, so run `scripts/md_to_docx.py` from the folder this SKILL.md is in (Claude Code gives the skill's base directory when it loads).

If `python-docx` is missing, run `pip install python-docx` and retry. The script handles `#`, `##`, `###` headings, bold and italic inline markup, lists, `> ` blockquote lines (rendered indented, gray, italic, used for Teacher's Notes), and markdown tables (real Word tables with a bold header row, used for the path). Write the source markdown accordingly:
- Lesson title as `#`, each stage as `##`.
- Bold the vocab word in each `word – definition` line and italicize the example sentence.
- Prefix each Teacher's Note line with `> `.
- For a path, write the exact markdown table already used to present it.

Name files `<class-slug>-lesson-<n>.docx`, `<class-slug>-lesson-<n>-handout.docx`, and `<class-slug>-learning-path.docx`. For a single lesson outside a path, use `<class-slug>-<lesson-name-slug>.docx`.

**After creating any file, tell the teacher exactly where it landed**: the full path in a connected folder, or a plain statement that it is attached in the chat. This applies to every file this skill writes: exports, handouts, and record files.
