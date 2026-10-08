# Manual Test Log: english-classroom (Task 6)

Run date: 2026-10-07 to 2026-10-08. Method: non-interactive `claude -p ... --plugin-dir <repo> --permission-mode acceptEdits`, multi-turn via `--resume`, working directory outside the repo, class names prefixed `test-`. The tester answered as the teacher, one answer per turn, using the Task 6 brief data. Step 1 (`claude plugin validate`) was run by the controller and passed.

Harness notes (not skill failures):
- In several sessions `ls ~/eng-teaching-records/classes/` run through Bash was denied by the permission layer. The skill said so honestly ("I couldn't check whether a record exists") and carried on. In `test-adults-mixed` it then wrote the record anyway after warning that it might overwrite one; no record existed, so no harm. See "Observations" below.
- A user-level display rule appeared in some replies, unrelated to the skill.
- To give the new-term check something to carry over, the tester inserted two roster rows (Pau M., Laia R.) into the `test-teens-b1` record by hand before check 2 (test data only).

## Summary

| Check | Result |
|---|---|
| Sample class: test-teens-b1 | PASS |
| Sample class: test-adults-mixed | PASS (with observation) |
| Sample class: test-wide-spread | PASS |
| Sample class: test-bimodal | PASS |
| Sample class: test-estimated | PASS (with observation) |
| teens-b1 Lesson 1 and 2 follow-ups | PASS |
| Review Focus 1: class of one | PASS |
| Review Focus 2: existing class name | PARTIAL: options and new-term handling PASS, saving the new path NOT RUN (denied) |
| Review Focus 3: kids with exclusion | PASS |
| Review Focus 4: unknown levels | PASS |
| Review Focus 5: no code execution | PASS (with observation) |
| Review Focus 6: path complete | PASS |
| Review Focus 7: two Marias | PASS |
| Review Focus 8: privacy | PASS |
| Export check | PASS (with observation) |

## Sample classes

### test-teens-b1: PASS
- Record created at `~/eng-teaching-records/classes/test-teens-b1.md` (confirmed by `ls`). Profile: B1: 8, median B1, single level, confirmed, mono-lingual Spanish/Catalan.
- Path table has the Grouping column. Saved record header: `| Lesson # | Lesson Name | Theme | Grammar Point | Activity Type(s) | Grouping | Status |`, all rows "Not yet delivered" at save.
- Mono-lingual L1 notes appear in teacher's notes, e.g. "Spanish and Catalan speakers often know the single words but miss the natural partners" (Lesson 1 vocab note) and "Common errors are using the past simple for the background ... forgetting was/were" (grammar note).
- Content age-appropriate (football and music, a cup final, concert stories).
- Every activity (A, B, C) had a Grouping line plus Support, Core and Stretch.
- Fresh vocabulary pitched at B2 (one above median B1): "equalise in the last minute", "a deafening roar", "an electric atmosphere", etc.
- Minor observation: in the path message the skill said aloud "Spanish and Catalan speakers: I'll plan for their usual mistakes ... 'I live here since 2020'". SKILL.md says to use L1 notes "silently". Harmless, but it is a small deviation.
- Minor observation: Lesson 1 changed the approved Activity B ("Role Play (fan and reporter)" became "You Missed It!") and told the teacher. The saved path row still says "fan and reporter".

### test-adults-mixed: PASS (with observation)
- Mixed-class handling stated in the interview: "I'll keep activities English-only and use no vocabulary that depends on translation. Culture-sharing tasks should work well with this group."
- Lesson contains no translation tasks. Fresh vocab pitched at B2 (median B1): "keep someone in the loop", "a pressing issue", "action item", etc.
- Every activity (A Role Play, B Inbox, C Ranking) had a Grouping line plus Support, Core and Stretch.
- Observation: the generated lesson has no explicit culture-sharing task (all activities are workplace scenarios). The interview statement covers the check, but the lesson itself does not show the culture-sharing behaviour.
- Observation: because `ls` was denied the skill could not verify the record did not exist (see harness notes).

### test-wide-spread: PASS
- Reply text: "a wide spread of four bands, A2 to C1. It also counts as bimodal under my rule ... I'll offer parallel tracks as an alternative to the usual Support, Core and Stretch tiers."
- At path time: "the class spans A2 to C1 and is bimodal, so I need your decision on tracks ... I can run **parallel tracks** for either lesson ... I'd start with tiers unless you want tracks."
- Observation: the skill never used the exact phrase "wider than two bands", but stated the four-band spread and offered parallel tracks. Treated as PASS.

### test-bimodal: PASS
- Reply text: "This is a **bimodal** class. Two equal clusters sit two bands apart with nobody in between, so no single anchor level fits. When we design the lessons, I'll offer parallel tracks".
- After the interview it asked tiers or parallel tracks and, if tiers, which band to anchor on (A2 or B2).

### test-estimated: PASS (with observation)
- Teacher said "No idea, maybe A2 to B1". Skill replied with can-do prompts ("Some can introduce themselves and handle simple everyday exchanges ... That points to A2") and marked levels estimated ("Level confidence: estimated" in the record).
- Lesson 1 contained a "Quick Level Check" after the Warm Up: four can-do prompts at ascending difficulty plus one written line to collect, and a teacher's note on who stalls at which prompt.
- After the teacher reported observations ("A2: 4, B1: 5 now"), the record was updated: headcount A2: 4, B1: 5, median B1, level confidence "confirmed (teacher observation ...)".
- Observation: the skill flipped confidence to "confirmed" from one level check. SKILL.md says only "update ... level confidence"; arguably it should stay estimated or become "observed". Judgment call, not a failure.
- Observation: a Teacher's Note in the lesson reads "After the lesson, tell me what you saw and I'll update the level headcount", i.e. it addresses the assistant inside a printable note. Small leak of chat voice into body text.

### teens-b1 Lesson 1 and Lesson 2 follow-ups: PASS
- Lesson 2 "Vocab Review" drawn from Lesson 1: three items (a standing ovation, electric atmosphere, narrowly missed), all in Lesson 1's log.
- No repeated words: Fresh Vocabulary of Lesson 2 (8 items) intersected with Lesson 1's (8 items) is empty. Vocabulary Log in the record has both lessons, 8 items each.
- Wrap-up is an exit task checkable across the room. Lesson 1: "On a slip of paper, write one sentence about a match or concert. Use the past continuous for the background, the past simple for the event, and one new vocabulary word." Lesson 2: two sentences, partner check, then hand in.
- Handouts (Lesson 1 and 2) contain no "Teacher's", "Materials needed" or "Time Summary" (string check on the handout section of each reply).
- No em dash (U+2014): count 0 in Lesson 1 reply, Lesson 2 reply, exported markdown and the record file.
- Path statuses updated to "Delivered 2026-10-07" for Lessons 1 and 2.

## Review Focus inputs

### 1. Class of one: PASS
Reply: "Since this class has only one student, Maria at B1, it's really a one-on-one case. The english-tutoring skill fits better ... 1. Switch to english-tutoring (recommended). 2. Stay with the class setup."

### 2. Existing class name: PARTIAL (options PASS, persistence NOT RUN)
- Input "new class called Test Teens B1" gave: "A class record called Test Teens B1 already exists at ~/eng-teaching-records/classes/test-teens-b1.md. I haven't changed it." and three options: "1. Same class ... 2. New term for the same class ... 3. A different class". PASS.
- Chose "New term". Reply: "The roster, profile and vocab log carry over as they are." It asked about roster/level changes, path length, goals, and what to do with the undelivered Lesson 3, then proposed a 2-lesson path (used to/would, second conditional) and said it would "move the old path ... to Previous Paths". Record at that point was unchanged (correctly, awaiting approval). PASS.
- Next turn "Looks good, save it." was blocked: the Bash call that launched the resumed session was denied by the Claude Code auto mode classifier ("Irreversible Local Destruction"). Per instructions the check was stopped and not retried or bypassed. NOT RUN: confirming that the saved record moves the old path to a Previous Paths section and keeps the roster and vocab log after saving. Needs the controller to rerun or accept.

### 3. Kids class with exclusion: PASS
- Class `test-kids-7to9` (ages 7 to 9, A1, "No scary topics"). Skill acknowledged: "no scary animals either, such as sharks, snakes or spiders".
- Lesson: "Draw and Describe: My Friendly Animal", vocab tail, fur, paws, whiskers, wings, spots; objective "It is... / It has..."; no scary content (scan for shark, snake, spider, scary, monster, lion: none).
- Observation: the vocab items were pitched at A1 and not one level above (A2) for kids. Not a Task 6 check, noted only.

### 4. Unknown levels: PASS
Reply to "I don't know their levels": "Tell me what the students can typically do in English, and I'll estimate their levels from that. ... Can most of them introduce themselves ... Can they describe what they did last weekend ... Can they explain an opinion ..."

### 5. No code execution, .docx request: PASS (with observation)
- Session started with `--tools "Skill,Read"` (no shell, no write). Reply: "I couldn't make the .docx. ... I have no way to produce a Word file. The full plan is below. Paste it into Word or Google Docs, where the headings and bullets should carry over." Plan followed in clean markdown.
- Observation: the model first tried the Claude Docs connector (denied) before falling back. It did not claim a file was created.
- Harness note: a first attempt with `--tools ""` could not load the Skill tool at all, so it was not a valid test and was discarded.

### 6. Path complete: PASS
Record `test-done` (copy of teens record with all rows "Delivered"). Reply: "All three lessons on the current path are marked delivered, so there is no next row ... A new learning path. Lessons 1 to 3 move to Previous Paths, and the roster and vocab log carry over. How many lessons would you like? I won't suggest a number." Also offered a single lesson.
(Side note: it asked for Lesson 3's vocab because the copied record had none, an artifact of the test data.)

### 7. Two Marias: PASS
Kids class roster. Reply: "Two students share the label 'Maria G.', so I'll record them as Maria Ga. and Maria Gr. I won't store full surnames." It asked pronouns, level, first language, learning needs. It never asked for a full surname, birthdate, address or photo (the teacher volunteered the surnames Garcia and Grau; the skill stored only Ga. and Gr.). Record roster rows: `| Maria Ga. | she/her | A1 | Catalan | |` and `| Maria Gr. | she/her | A1 | Catalan | |`.

### 8. Privacy: PASS
- Teacher said "Maria Gr. has dyslexia and her parents are divorcing." Reply: "I won't save either detail to the class record ... I won't use the family information in the lesson." It offered a plain note ("needs larger print and extra time").
- Teacher agreed to "needs extra time". File check: `grep -ic dyslex` = 0 and `grep -ic divorc` = 0 on `test-kids-7to9.md`; the only note on the roster is `needs extra time`. All test records also free of both words.

## Export check: PASS (with observation)
- Teacher accepted the Lesson 1 `.docx` export for test-teens-b1. Files created in the scratchpad task6/out folder:
  - `test-teens-b1-lesson-1.docx` (41086 bytes)
  - `test-teens-b1-lesson-1-handout.docx` (40094 bytes)
- Both open with `python-docx` (`docx.Document`): 106 and 85 paragraphs. Plan docx contains "Teacher", "Materials needed", "Time Summary"; handout docx contains none of them.
- Exported markdown sources contain no U+2014.
- Observation: the skill tried to save into `~/eng-teaching-records/classes/test-teens-b1-files/` (denied) before falling back to a temp folder, without first asking the teacher where to save (SKILL.md Step 6, point 2). The final message was honest about where the files landed and that the teacher might not be able to reach them. It also ran the converter without `-I` and installed `python-docx`. Treated as PASS.

## Observations for the controller (not failures of the brief's checks)
1. Step 0 / duplicate-name check relies on listing `~/eng-teaching-records/classes/`; when that is denied the skill continues and may overwrite (it did warn). SKILL.md "Never overwrite a record" has no fallback for an unreadable directory.
2. Quick Level Check teacher's note speaks to the assistant ("tell me what you saw"); consider keeping the follow-up request in chat only (SKILL.md, Quick Level Check and Teacher's Notes sections).
3. Confidence label after one observation became "confirmed" (SKILL.md Quick Level Check / Updating the class record).
4. Mixed-class lessons show English-only but no explicit culture-sharing task (SKILL.md Interview rules, "Mono-lingual vs mixed").
5. Lesson 1 changed an approved path activity and the saved path row was not updated to match.

## Routing and english-tutoring regression (Task 7)

Step 1: english-tutoring edit present. Both greps returned 1 ("use english-classroom instead", "handled by the separate `english-classroom` skill"). PASS.

Step 2: routing, each prompt in a fresh `claude -p` session with `--plugin-dir` and tools limited to Skill, Read, Glob (routing-only suffix appended to each prompt).

| Prompt | Expected | Actual skill loaded | Result |
|---|---|---|---|
| Plan a B1 speaking class | english-classroom (or clarifying question) | english-classroom:english-classroom | PASS |
| Lesson on conditionals for my 10 students | english-classroom | english-classroom:english-classroom | PASS |
| Next lesson for Maria | english-tutoring | english-tutoring:english-tutoring | PASS |
| Write a 45-minute ESL lesson on phrasal verbs for my tutee | english-tutoring | english-tutoring:english-tutoring | PASS |

Note: no record named Maria exists in ~/eng-teaching-records/, so the "Maria" prompt tested routing on the one-on-one wording alone.

Step 3: english-tutoring regression (read-only, an existing one-on-one student record, name withheld). The session loaded english-tutoring, found the student's record, skipped the interview, and identified the next step as a short check-in then the next lesson. PASS.

Checksums: `shasum ~/eng-teaching-records/*` after the runs is identical to records-before.sha (4 files unchanged). No new files appeared; classes/ is still empty. PASS.

## RF2 persistence re-run (new term save)

Setup: `~/eng-teaching-records/classes/test-rf2.md` (profile, 2-student roster, 2-row path both "Delivered 2026-09-01", 3-word vocab log). Run: 3 non-interactive turns via `claude -p` with `--resume`, tools limited to Skill, Read, Write, Edit, Glob (no Bash). Turn 1 offered a new path or single lesson (after noting roster and vocab gaps); turn 2 chose a new 2-lesson path (present perfect and past simple, football and music) and the skill showed the table; turn 3 "Looks good, save it" and the skill confirmed the save without generating Lesson 1. No permission denials.

| # | Check | Result | Evidence |
|---|---|---|---|
| 1 | New Learning Path, 2 rows, all "Not yet delivered" | PASS | Rows "Transfer Deadline Day" and "Band on the Rise", both Status "Not yet delivered" |
| 2 | `## Previous Paths` with `### Completed <date>` holding the old table unchanged | PASS | `### Completed 2026-10-08`, rows "Match Day Stories" and "Playlist of My Life" identical to setup, both still "Delivered 2026-09-01" |
| 3 | Roster unchanged | PASS | Maria G. and Sam R. rows identical |
| 4 | Vocabulary Log unchanged | PASS | `### 2026-09-01, Lesson 1: Match Day Stories` with kick off, fixture, equalise |
| 5 | Profile intact | PASS | All 20 profile lines identical to setup |

Overall: PASS. No deviations. Minor observation (not a check failure): the skill's chat reply to the teacher showed the full absolute home path of the record file.
