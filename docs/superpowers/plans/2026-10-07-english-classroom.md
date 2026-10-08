# english-classroom Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build and (after explicit confirmation) publish `english-classroom`, a Claude Code plugin and skill that interviews a teacher about a class and produces differentiated, multi-student lesson plans.

**Architecture:** A plugin repo mirroring `ratherlegit/english-tutoring`. One skill (`SKILL.md`) holds the workflow. Two new reference files hold the class interview and differentiation rules. Four references and one script are copied from english-tutoring and extended with group notes. A stdlib `unittest` file (`tests/validate_skill.py`) is the "test suite": it pins structure, required rules, and the no-em-dash output rule, and is written before the content it checks.

**Tech Stack:** Markdown, JSON (Claude Code plugin manifest), Python 3 stdlib `unittest`, `python-docx` (already installed, used by the copied converter), `git`, `gh`.

**Spec:** `docs/superpowers/specs/2026-10-07-english-classroom-design.md`

## Global Constraints

- Repo root: `/Users/zacharyrosen/english-classroom` (all paths below are relative to it).
- Plugin and skill name: `english-classroom`. Initial version `1.0.0`. License MIT. Author `Zachary Rosen`.
- GitHub target: `ratherlegit/english-classroom`, public, created only after the user confirms (Task 8).
- Class records live in `~/eng-teaching-records/classes/<class-slug>.md` (lowercase, hyphens). One-on-one files in `~/eng-teaching-records/` are never read or modified.
- No em dash (U+2014) in narrative or body text in any file this plan creates, except heading separators. New content files in this plan use none at all. Vocab items use an en dash (`word – definition`).
- No eyebrow labels (small uppercase labels above headings) in any copy.
- Copied files: `cefr-levels.md`, `lesson-frameworks.md`, `activity-bank.md`, `teacher-activities.md`, `md_to_docx.py`, `LICENSE`. `md_to_docx.py` is copied byte-for-byte.
- Wide-spread threshold: more than two CEFR bands. Class size bands: small up to 6, medium 7 to 15, large 16 or more.
- Fresh vocab: one CEFR level above the class median, same list for everyone, Support tier adds scaffolds never a shorter list.
- Commits end with the trailer `Co-Authored-By: Claude Sonnet 5.5 <noreply@anthropic.com>`.

## Review Focus

Inputs the spec implies but no happy-path task exercises. Each has a pinned phrase test in the task that owns the file.

1. A class of one student (the teacher describes a single learner): skill must point to english-tutoring and offer to switch, not build a class plan. (Task 4, `SKILL.md`)
2. A new class whose name slugs to an existing class record: skill must say the record already exists and ask whether it is the same class or needs a different name, never overwrite. (Task 4)
3. Kids or teens class: topics and activities must be age-appropriate and exclusions are honored. (Tasks 2 and 4)
4. Every student at the same level (no spread) or every level unknown: tiers still apply, based on task scaffolding, and the skill asks can-do questions instead of guessing. (Tasks 2 and 4)
5. Environment with no shell or no connected folder: `.docx` export must fall back to pasteable markdown and never claim a file was created. (Task 4)

## File Structure

| File | Responsibility |
|---|---|
| `tests/validate_skill.py` | The test suite. Structure, required rules, no-em-dash, converter smoke test, doc checks. |
| `.claude-plugin/plugin.json`, `marketplace.json` | Plugin identity and marketplace listing. |
| `skills/english-classroom/SKILL.md` | The workflow: Steps 0 to 6 and output rules. |
| `skills/english-classroom/references/class-interview.md` | Interview questions, why they matter, light mode prompts, roster, mono vs mixed L1, L1 cheat sheet. |
| `skills/english-classroom/references/differentiation.md` | Tiers, grouping, wide spread, class size, age, early finishers. |
| `skills/english-classroom/references/{cefr-levels,lesson-frameworks,activity-bank,teacher-activities}.md` | Copied, then a `## Group classroom notes` section appended to three of them. |
| `skills/english-classroom/scripts/md_to_docx.py` | Copied converter. |
| `README.md`, `AGENTS.md` | User-facing docs and multi-harness instructions. |
| `LICENSE`, `.gitignore` | Standard. |

---

### Task 1: Test suite and scaffold

**Files:**
- Create: `tests/validate_skill.py`
- Create: `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`, `.gitignore`, `LICENSE`
- Create: `skills/english-classroom/scripts/md_to_docx.py` (copy)
- Create: `skills/english-classroom/references/{cefr-levels,lesson-frameworks,activity-bank,teacher-activities}.md` (copies)

**Interfaces:**
- Produces: `tests/validate_skill.py` with test classes `TestScaffold`, `TestExport`, `TestNewReferences`, `TestGroupNotes`, `TestSkillMd`, `TestDocs`. Helpers `read(path)`, `frontmatter(text)`, `non_heading_lines(text)`. Constants `ROOT`, `SKILL_DIR`, `SKILL_MD`, `REFS`. Later tasks run one class at a time: `python3 tests/validate_skill.py TestName -v`.

- [ ] **Step 1: Write the test suite**

Create `tests/validate_skill.py`:

```python
"""Structure and rule checks for the english-classroom plugin. Stdlib only."""
import json
import re
import subprocess
import sys
import tempfile
import unittest
from pathlib import Path

ROOT = Path(__file__).resolve().parent.parent
SKILL_DIR = ROOT / "skills" / "english-classroom"
SKILL_MD = SKILL_DIR / "SKILL.md"
REFS = SKILL_DIR / "references"
EM_DASH = "\u2014"


def read(path):
    return Path(path).read_text(encoding="utf-8")


def frontmatter(text):
    match = re.match(r"---\n(.*?)\n---\n", text, re.S)
    if not match:
        return {}
    fields = {}
    for line in match.group(1).splitlines():
        if ":" in line:
            key, value = line.split(":", 1)
            fields[key.strip()] = value.strip()
    return fields


def non_heading_lines(text):
    body = re.sub(r"\A---\n.*?\n---\n", "", text, flags=re.S)
    return [ln for ln in body.splitlines() if not ln.lstrip().startswith("#")]


def em_dash_lines(path):
    return [ln for ln in non_heading_lines(read(path)) if EM_DASH in ln]


class TestScaffold(unittest.TestCase):
    def test_required_files_exist(self):
        required = [
            ".claude-plugin/plugin.json",
            ".claude-plugin/marketplace.json",
            ".gitignore",
            "LICENSE",
            "skills/english-classroom/scripts/md_to_docx.py",
            "skills/english-classroom/references/cefr-levels.md",
            "skills/english-classroom/references/lesson-frameworks.md",
            "skills/english-classroom/references/activity-bank.md",
            "skills/english-classroom/references/teacher-activities.md",
        ]
        for rel in required:
            self.assertTrue((ROOT / rel).is_file(), rel)

    def test_plugin_json(self):
        data = json.loads(read(ROOT / ".claude-plugin/plugin.json"))
        self.assertEqual(data["name"], "english-classroom")
        self.assertEqual(data["version"], "1.0.0")
        self.assertEqual(data["license"], "MIT")
        self.assertIn("ratherlegit/english-classroom", data["repository"])

    def test_marketplace_json_matches_plugin(self):
        plugin = json.loads(read(ROOT / ".claude-plugin/plugin.json"))
        market = json.loads(read(ROOT / ".claude-plugin/marketplace.json"))
        self.assertEqual(market["name"], "english-classroom")
        self.assertEqual(market["version"], plugin["version"])
        self.assertEqual(market["plugins"][0]["name"], "english-classroom")
        self.assertEqual(market["plugins"][0]["source"], "./")

    def test_gitignore(self):
        self.assertIn(".DS_Store", read(ROOT / ".gitignore"))

    def test_converter_identical_to_english_tutoring_copy(self):
        src = Path.home() / ".claude/skills/english-tutoring/skills/english-tutoring/scripts/md_to_docx.py"
        if not src.is_file():
            self.skipTest("english-tutoring not installed locally")
        self.assertEqual(
            (SKILL_DIR / "scripts/md_to_docx.py").read_bytes(), src.read_bytes()
        )


class TestExport(unittest.TestCase):
    def test_converter_builds_docx_with_heading_and_table(self):
        try:
            import docx
        except ImportError:
            self.skipTest("python-docx not installed")
        md = (
            "# Test Lesson\n\n## Warm Up Conversation\n\n- One\n- Two\n\n"
            "| Lesson # | Lesson Name | Grouping | Status |\n|---|---|---|---|\n"
            "| 1 | Intro | Mixed pairs | Not yet delivered |\n"
        )
        with tempfile.TemporaryDirectory() as tmp:
            src = Path(tmp) / "plan.md"
            out = Path(tmp) / "plan.docx"
            src.write_text(md, encoding="utf-8")
            subprocess.run(
                [sys.executable, str(SKILL_DIR / "scripts/md_to_docx.py"), str(src), str(out)],
                check=True,
            )
            self.assertTrue(out.is_file())
            document = docx.Document(str(out))
            self.assertEqual(len(document.tables), 1)
            texts = [p.text for p in document.paragraphs]
            self.assertIn("Test Lesson", texts)


class TestNewReferences(unittest.TestCase):
    def test_class_interview_content(self):
        text = read(REFS / "class-interview.md")
        for phrase in [
            "Number of students",
            "Backgrounds",
            "mono-lingual",
            "mixed",
            "Light mode",
            "Roster",
            "L1 interference cheat sheet",
            "age-appropriate",
            "can do",
        ]:
            self.assertIn(phrase, text, phrase)

    def test_differentiation_content(self):
        text = read(REFS / "differentiation.md")
        for phrase in [
            "Support",
            "Core",
            "Stretch",
            "two CEFR bands",
            "parallel",
            "Small",
            "Medium",
            "Large",
            "mixed-level pairs",
            "early finishers",
            "single level",
            "never a shorter",
        ]:
            self.assertIn(phrase, text, phrase)

    def test_no_em_dashes_in_new_references(self):
        for name in ["class-interview.md", "differentiation.md"]:
            self.assertEqual(em_dash_lines(REFS / name), [], name)


class TestGroupNotes(unittest.TestCase):
    def test_group_notes_appended(self):
        for name in ["teacher-activities.md", "activity-bank.md", "lesson-frameworks.md"]:
            self.assertIn("## Group classroom notes", read(REFS / name), name)

    def test_teacher_activities_group_rules(self):
        text = read(REFS / "teacher-activities.md")
        for phrase in ["Support", "Stretch", "rotate", "Press conference"]:
            self.assertIn(phrase, text, phrase)


class TestSkillMd(unittest.TestCase):
    def setUp(self):
        self.text = read(SKILL_MD)

    def test_frontmatter(self):
        fm = frontmatter(self.text)
        self.assertEqual(fm.get("name"), "english-classroom")
        desc = fm.get("description", "")
        self.assertIn("english-tutoring", desc)
        self.assertIn("class", desc)
        self.assertLess(len(desc), 1024)

    def test_steps_present_in_order(self):
        headings = [
            "## Step 0: Check for a returning class",
            "## Step 1: Interview the teacher about the class",
            "## Step 2: Design the class learning path",
            "## Step 3: Pick a framework",
            "## Step 4: Build the plan",
            "## Step 5: Differentiation and common pitfalls",
            "## Output format",
            "## Step 6: Export as a .docx (optional)",
        ]
        positions = [self.text.find(h) for h in headings]
        self.assertTrue(all(p >= 0 for p in positions), positions)
        self.assertEqual(positions, sorted(positions))

    def test_required_rules(self):
        for phrase in [
            "~/eng-teaching-records/classes/",
            "median",
            "Support",
            "Core",
            "Stretch",
            "Grouping",
            "two CEFR bands",
            "parallel",
            "never a shorter",
            "exit task",
        ]:
            self.assertIn(phrase, self.text, phrase)

    def test_review_focus_rules(self):
        for phrase in [
            "class of one",
            "already exists",
            "age-appropriate",
            "single level",
            "can do",
            "cannot generate a Word file",
        ]:
            self.assertIn(phrase, self.text, phrase)

    def test_referenced_files_exist(self):
        for rel in re.findall(r"`((?:references|scripts)/[\w./-]+)`", self.text):
            self.assertTrue((SKILL_DIR / rel).is_file(), rel)

    def test_no_em_dashes(self):
        self.assertEqual(em_dash_lines(SKILL_MD), [])


class TestDocs(unittest.TestCase):
    def test_readme(self):
        text = read(ROOT / "README.md")
        self.assertIn("ratherlegit/english-classroom", text)
        self.assertIn("english-tutoring", text)
        self.assertEqual(em_dash_lines(ROOT / "README.md"), [])

    def test_agents(self):
        text = read(ROOT / "AGENTS.md")
        self.assertIn("skills/english-classroom/SKILL.md", text)
        self.assertIn("~/eng-teaching-records/classes/", text)
        self.assertEqual(em_dash_lines(ROOT / "AGENTS.md"), [])


if __name__ == "__main__":
    unittest.main()
```

- [ ] **Step 2: Run to verify it fails**

Run: `cd ~/english-classroom && python3 tests/validate_skill.py TestScaffold -v`
Expected: FAIL (required files missing, `FileNotFoundError` for `plugin.json`).

- [ ] **Step 3: Create the scaffold and copy the shared files**

```bash
cd ~/english-classroom
SRC=~/.claude/skills/english-tutoring
mkdir -p .claude-plugin skills/english-classroom/references skills/english-classroom/scripts
cp "$SRC/LICENSE" LICENSE
cp "$SRC/skills/english-tutoring/scripts/md_to_docx.py" skills/english-classroom/scripts/md_to_docx.py
for f in cefr-levels lesson-frameworks activity-bank teacher-activities; do
  cp "$SRC/skills/english-tutoring/references/$f.md" "skills/english-classroom/references/$f.md"
done
printf 'class-records/\n.DS_Store\n' > .gitignore
```

Create `.claude-plugin/plugin.json`:

```json
{
  "name": "english-classroom",
  "description": "ESL/EFL lesson planning for group classes: class interview, differentiated multi-lesson learning paths, shared vocabulary, and per-class memory.",
  "version": "1.0.0",
  "author": { "name": "Zachary Rosen" },
  "homepage": "https://github.com/ratherlegit/english-classroom",
  "repository": "https://github.com/ratherlegit/english-classroom",
  "license": "MIT"
}
```

Create `.claude-plugin/marketplace.json`:

```json
{
  "$schema": "https://anthropic.com/claude-code/marketplace.schema.json",
  "name": "english-classroom",
  "version": "1.0.0",
  "description": "ESL/EFL lesson planning for group classes: class interview, differentiated multi-lesson learning paths, shared vocabulary, and per-class memory.",
  "owner": {
    "name": "Zachary Rosen"
  },
  "plugins": [
    {
      "name": "english-classroom",
      "description": "ESL/EFL lesson planning for group classes: class interview, differentiated multi-lesson learning paths, shared vocabulary, and per-class memory.",
      "source": "./",
      "category": "education"
    }
  ]
}
```

- [ ] **Step 4: Run to verify it passes**

Run: `cd ~/english-classroom && python3 tests/validate_skill.py TestScaffold TestExport -v`
Expected: PASS for all tests (the converter test is skipped only if python-docx is missing; it is installed).

- [ ] **Step 5: Commit**

```bash
cd ~/english-classroom
git add -A
git commit -m "Add test suite, plugin manifests, and shared files copied from english-tutoring" -m "Co-Authored-By: Claude Sonnet 5.5 <noreply@anthropic.com>"
```

---

### Task 2: New reference files (class interview, differentiation)

**Files:**
- Create: `skills/english-classroom/references/class-interview.md`
- Create: `skills/english-classroom/references/differentiation.md`

**Interfaces:**
- Consumes: `tests/validate_skill.py::TestNewReferences` from Task 1.
- Produces: the section names `SKILL.md` links to: class-interview.md (question table, `## Light mode prompts`, `## Roster`, `## Mono-lingual vs mixed classes`, `## L1 interference cheat sheet`) and differentiation.md (`## Anchor level and tiers`, `## Grouping patterns`, `## Wide level spread`, `## Class size`, `## Age groups`, `## Early finishers and pitfalls`).

- [ ] **Step 1: Run the test to verify it fails**

Run: `cd ~/english-classroom && python3 tests/validate_skill.py TestNewReferences -v`
Expected: FAIL with `FileNotFoundError: .../class-interview.md`.

- [ ] **Step 2: Write `class-interview.md`**

Create `skills/english-classroom/references/class-interview.md`:

```markdown
# Class Interview

This table lists the information needed, asked as separate questions in **detailed mode**. In **light mode** the same fields are gathered through four bundled prompts (see below). The "why it matters" reasoning applies either way.

## Questions and why they matter

| Ask | Why it matters |
|---|---|
| **Class name** (asked first, free text) | Names the record file and makes the plan feel written for this class. Always typed by the teacher, never offered as a suggestion. |
| **Setting** | School, adult evening class, corporate, online, or in person. Decides which activities are practical (movement, whiteboards, breakout rooms) and the register of topics. |
| **Number of students** | Sets the grouping plan and monitoring approach (see `differentiation.md`, Class size). If the answer is 1, this is not a class: point to english-tutoring. |
| **Age group and range** | Kids, teens, or adults. Drives tone, topics, activity types, attention span, and what is age-appropriate. |
| **Levels** | The lowest and highest CEFR level and the median. If the teacher is unsure, ask what students can do (for example "can most of them hold a short conversation about their day? write an email?") and infer a level from `cefr-levels.md` instead of asking them to self-rate against labels they may not know. If every student is at a single level, record that. |
| **Backgrounds** | Countries and first languages. Decides mono-lingual vs mixed handling (below). Ask for places, not language families. |
| **Shared interests and professions** | The main personalization lever for a class. Themes and examples come from what the group has in common. |
| **Goal** | Exam prep, work, travel, immigration, general fluency. Steers topic register and skill priority for the whole class. |
| **Session length and frequency** | Offer 30, 60, 90 minutes or something else. Ask frequency separately if it matters for path planning. |
| **Activity choice** | Random, specific, or "show me the list first" (show names only from `teacher-activities.md`, then re-ask). |
| **Single lesson or path** | If a path, ask how many lessons as the very next question. Never suggest or default to a number. |
| **Key skills** | Which skills get the most weight across the path. |
| **Include or exclude** | Topics to avoid, a coursebook to align with, grammar already mastered or specifically requested. For kids and teens classes, also ask about topics the school or parents would not want. |
| **Roster** (last) | Offer to add individual students now or later (see Roster). |

Vocab constraint, vocab count, home assignment, teacher's notes, and handout are NOT asked in this round. They are asked right before a specific lesson is written.

## Light mode prompts

Ask these one at a time. Each answer can be a short paragraph covering several fields. Parse every answer into the same fields as detailed mode, and ask a one-off follow-up for anything missing instead of guessing.

1. "What is the class called, where does it meet (school, evening class, company, online), and how many students are in it?"
2. "How old are the students, and what are their levels? Give CEFR levels if you know them, or describe what they can and can't do."
3. "Where are the students from, what first languages do they speak, and what interests or jobs do they share?"
4. "What is the class goal, and how long and how often do you meet? Is this a single lesson or a path (and how many lessons)? Any activity preferences or things to exclude, or say 'you choose' for defaults."

Then offer the roster as a final question.

## Roster

The roster is optional and can be added at any time, including when a returning teacher says "add two new students". Each entry has: name, pronouns, level, first language, notes. Pronouns are asked, never inferred from a name; use they/them until told.

When a roster exists, use it to name students in grouping suggestions and to spread personal interests across lessons so no one student is the recurring example. When no roster exists, plans use generic tiers and shared interests only.

## Mono-lingual vs mixed classes

- **Mono-lingual** (everyone shares a first language, inferred from the location): apply the L1 interference cheat sheet below. Anticipate the most likely errors and put a short pre-emptive correction or drill in the teacher's notes. Treat the cheat sheet as a starting hypothesis, not a stereotype.
- **Mixed** (several first languages): activities stay English-only and lean on culture-sharing ("how does this work in your country?"). No vocab or activity may depend on translation. Teacher's notes give general error guidance. If the roster records first languages, notes may also flag a few high-frequency L1 errors for those named students.

## Age groups

- **Kids**: short activities, movement, games, concrete topics, simple instructions. Content must be age-appropriate and honor every exclusion.
- **Teens**: topics tied to school, social life, hobbies, and media. Content must be age-appropriate. Avoid topics a school would not want.
- **Adults**: work, travel, daily-life scenarios, and goal-driven content.

## L1 interference cheat sheet

Common error patterns by native language. Use to anticipate errors and build a quick pre-emptive correction or extra drill into the lesson, not to stereotype every learner. Since the interview asks for a place rather than a language name, map the place to its likely native language yourself (Barcelona to Spanish or Catalan, Osaka to Japanese, and so on) and apply this silently.

- **Spanish/Italian/Portuguese (Romance)**: false cognates (embarrassed/embarazada), gender agreement carried into English adjectives, "have" vs "be" for age and states ("I have 20 years"), present tense for future plans
- **Mandarin/Cantonese**: no articles in the L1, so omission or overuse; no verb tense marking, so tense errors especially with perfect aspect; topic-comment structure, so unusual word order in complex sentences
- **Japanese/Korean**: particles instead of word order, so word order errors; no articles; plural marking difficulty; politeness levels mapped onto English formality choices
- **Arabic**: no present tense "to be" in the L1, so omission of "is/are"; specific pronunciation targets (p/b, short vowel distinctions)
- **Russian/Polish/other Slavic**: no articles; no continuous aspect in the same way, so overuse of simple tenses; case system does not map to English prepositions, so preposition errors
- **Vietnamese/Thai**: no verb conjugation or tense marking, so tense and third-person -s omission; tone-based L1, so stress and intonation challenges
- **German/Dutch**: structurally close to English, so fewer structural errors, but subordinate clause word order and false cognates still appear

When the language is not listed, reason from typological features (does the L1 mark tense, articles, plurals? what is its base word order?) instead of skipping the analysis.
```

- [ ] **Step 3: Write `differentiation.md`**

Create `skills/english-classroom/references/differentiation.md`:

```markdown
# Differentiation for a Mixed Room

One lesson has to work for every student in it. The default approach is an anchor level plus three versions of each task.

## Anchor level and tiers

Target the class median CEFR level for grammar, vocab, and the Core task. Every activity then has three versions of the same task:

- **Support**: the Core task with scaffolds. Sentence frames, a word bank, fewer items, a model answer, or a partly completed example. The goal is the same as Core, with a bridge to reach it.
- **Core**: the baseline task, pitched at the median.
- **Stretch**: the Core task extended. An extra constraint, longer or more precise output, more abstract content, or an error-correction twist.

Rules:
- Same task, three entry points. Nobody sits out and the class stays together for the debrief.
- Fresh vocabulary is one list for everyone, pitched one level above the median. The Support tier adds scaffolds for the same words (sentence frames, example hints, a word bank), never a shorter or easier list, so the whole class learns the same vocabulary.
- Grammar is taught once at the median. The Support tier gets a short review box of the prerequisite structure. The Stretch tier gets a short extension (a further use or a contrast).
- If the class is at a single level, tiers still apply. Differentiate by scaffolding and extension, not by assuming the level is uniform in practice. If levels are unknown, ask what students can do instead of guessing.

## Grouping patterns

- **Mixed-level pairs**: a stronger and a weaker student. Good for dialogues, information gaps, and peer correction. Rotate partners across lessons.
- **Same-level trios or groups**: good for Stretch tasks and Support tasks, so each group works at its own pace.
- **Whole class**: warm-up conversation, vocab check, and wrap-up.
- Each activity states its suggested grouping (for example mixed-level pairs for a dialogue, same-level trios for a Stretch task). When a roster exists, name the pairs and groups. When it does not, describe them by tier ("pair a Support student with a Core student").
- Rotate who goes first and who reports back across lessons so the same students do not always lead.

## Wide level spread

If the spread is wider than two CEFR bands (for example A2 to C1), say so plainly and offer parallel tracks for that lesson: two or three content bands with their own vocabulary and grammar focus, sharing one theme so the class can reconvene for a joint wrap-up. Tiers remain the default unless the teacher chooses parallel tracks. Parallel tracks cost the teacher more management, so recommend them only when tiers would stretch too far.

## Class size

- **Small (up to 6)**: one or two groups or pairs only. The teacher can monitor everyone closely. Whole-class discussion works.
- **Medium (7 to 15)**: three to five groups or pairs. Monitor by circulating on a fixed route. Use mini-whiteboards or ticket checks for quick feedback.
- **Large (16 or more)**: pairs and fixed small groups with assigned roles (reporter, timekeeper, writer). Avoid activities where one student performs alone for the whole room. Use written exit tasks that can be collected.

## Age groups

- **Kids**: 5 to 10 minute activities, movement, games, concrete topics.
- **Teens**: topics from school, hobbies, and media. Content must be age-appropriate.
- **Adults**: goal-driven scenarios (work, travel, exams) and respect for time constraints.

## Early finishers and pitfalls

- Always give early finishers something useful: the Stretch version of the same task, or a bonus prompt. Name it in the teacher's note under the relevant activity.
- Avoid more than one new grammar point per lesson or more than 8 to 10 new vocab items.
- For exam-prep classes, tie each activity to the exam skill being tested.
- Do not over-label students. Tiers are about tasks, not about people. Students can choose or be moved between tiers per activity.
```

- [ ] **Step 4: Run to verify it passes**

Run: `cd ~/english-classroom && python3 tests/validate_skill.py TestNewReferences -v`
Expected: PASS (3 tests).

- [ ] **Step 5: Commit**

```bash
cd ~/english-classroom
git add skills/english-classroom/references/class-interview.md skills/english-classroom/references/differentiation.md
git commit -m "Add class-interview and differentiation references" -m "Co-Authored-By: Claude Sonnet 5.5 <noreply@anthropic.com>"
```

---

### Task 3: Group notes for the copied references

**Files:**
- Modify (append): `skills/english-classroom/references/teacher-activities.md`
- Modify (append): `skills/english-classroom/references/activity-bank.md`
- Modify (append): `skills/english-classroom/references/lesson-frameworks.md`

**Interfaces:**
- Consumes: `TestGroupNotes` from Task 1. Section name `## Group classroom notes` must match exactly.

- [ ] **Step 1: Run the test to verify it fails**

Run: `cd ~/english-classroom && python3 tests/validate_skill.py TestGroupNotes -v`
Expected: FAIL (`## Group classroom notes` not found).

- [ ] **Step 2: Append the three sections**

```bash
cd ~/english-classroom/skills/english-classroom/references
cat >> teacher-activities.md <<'EOF'

## Group classroom notes

The entries above were written for one student. In english-classroom, every activity runs with a class, so apply these adjustments. Write each activity as the Core version, then add Support and Stretch versions (see `differentiation.md`) and a suggested grouping.

- **Works in pairs, trios, or small groups as written**: Read a (generated) dialogue, Speaking Practice, Role Play (roles in pairs or trios, with an observer in trios), Ranking scenario (rank individually, then agree a group ranking), Impossible choice, Mini Debates (two teams), Inbox activity (reply, then compare with a partner), Gallery tour (groups circulate), Hometown Hero (interviewer and interviewee pairs, then swap), Respond to Subway takes (pass the take around the group).
- **Needs adapting**: Devil's Advocate (pairs: one states an opinion, the partner argues against, then swap), Press conference (running it with one student on stage at a time is slow for a class, so use small groups where one student answers and the others ask, rotating the role), Sentence upgrades (work individually, then compare in pairs).
- **Performance-style activities**: any activity where one student speaks alone in front of the whole room should appear at most once per lesson, and the teacher should rotate who goes first across lessons.
- **Describe a picture**: pairs describe a photo to each other. It still counts against the one external-material activity per lesson. The teacher can project one photo or give each pair a different one.
- **Fresh vocabulary**: one list for the whole class, with scaffolds in the Support tier. See `differentiation.md`.
EOF

cat >> activity-bank.md <<'EOF'

## Group classroom notes

Use these adjustments when an activity is run with a class instead of one student.

- Prefer formats that work in pairs and small groups. If an activity is a monologue or a one-to-one exchange, turn it into a pair task or a rotating group task.
- Give every activity a suggested grouping (mixed-level pairs, same-level trios, whole class) and a Support, Core, and Stretch version. See `differentiation.md`.
- Warm-ups work well as whole-class or pair activities. Choose ones every student can answer regardless of level.
- Limit activities where one student performs alone for the whole room, and rotate who goes first.
EOF

cat >> lesson-frameworks.md <<'EOF'

## Group classroom notes

The frameworks above apply to a class with these adjustments.

- **PPP**: Presentation is whole class. Practice starts individually, then students check answers in pairs. Production runs in pairs or groups, with Support, Core, and Stretch versions of each task.
- **TBL**: the task cycle runs in groups, then each group reports back. The language focus stage uses errors collected across groups, so the teacher should monitor several groups and note common mistakes.
- **ESA**: Engage is a whole-class or pair warm-up. Study is taught once at the median level. Activate runs in tiered groups.
- **Receptive skills**: pre-task prediction in pairs, during-task checks individually with a pair comparison, post-task discussion in groups.
- In every framework, plan for monitoring: decide where the teacher stands, which groups to visit first, and what the early finishers do.
EOF
```

- [ ] **Step 3: Run to verify it passes**

Run: `cd ~/english-classroom && python3 tests/validate_skill.py TestGroupNotes -v`
Expected: PASS (2 tests).

- [ ] **Step 4: Commit**

```bash
cd ~/english-classroom
git add skills/english-classroom/references
git commit -m "Append group classroom notes to shared references" -m "Co-Authored-By: Claude Sonnet 5.5 <noreply@anthropic.com>"
```

---

### Task 4: SKILL.md

**Files:**
- Create: `skills/english-classroom/SKILL.md`

**Interfaces:**
- Consumes: `references/class-interview.md`, `references/differentiation.md`, `references/cefr-levels.md`, `references/lesson-frameworks.md`, `references/activity-bank.md`, `references/teacher-activities.md`, `scripts/md_to_docx.py` (all exist after Tasks 1 to 3). `TestSkillMd` checks every backticked `references/...` and `scripts/...` path in the file exists.
- Produces: the class record format and step names that README and AGENTS.md reference.

- [ ] **Step 1: Run the test to verify it fails**

Run: `cd ~/english-classroom && python3 tests/validate_skill.py TestSkillMd -v`
Expected: FAIL (`FileNotFoundError: .../SKILL.md`).

- [ ] **Step 2: Write `SKILL.md`**

Create `skills/english-classroom/SKILL.md`:

````markdown
---
name: english-classroom
description: Develops lesson plans and learning paths for teaching English to a class of multiple ESL/EFL students (group classes, mixed levels and backgrounds). Use when the user asks to plan a class, group lesson, or course for several English learners, e.g. "plan a lesson for my class of 12 B1 students" or "set up my evening English class". For one student at a time, use english-tutoring instead.
---

# ESL Class Lesson Plans

Build classroom-ready English lesson plans for a class of several ESL/EFL learners. Adapt structure, depth, and format to what the teacher asks for. Do not force a template they did not request.

**This skill covers group classes.** For a single student, the english-tutoring skill is the better fit. If the teacher describes one learner, say so and offer to switch (see Step 1).

Records are stored in `~/eng-teaching-records/classes/`, separate from one-on-one student files in `~/eng-teaching-records/`. Never read or modify the one-on-one files.

## Step 0: Check for a returning class

Before interviewing anyone, check whether the teacher names a class you already have a record for, such as "next lesson for my Tuesday class". Look for `~/eng-teaching-records/classes/<class-slug>.md` (lowercase, hyphens for spaces). Create `~/eng-teaching-records/classes/` if it does not exist.

Records live outside the skill folder on purpose: the profile, roster, path, and vocab history are personal data that must survive skill updates and reinstalls.

- **If a record exists**: skip Step 1 entirely. Load the profile, roster, path, and vocab log. Ask a brief check-in only: did students join or leave, did any levels change, anything else to adjust. Update the roster and profile with any changes. A roster change never resets the path or the vocab log.
  - **If there is an active path**: remind the teacher what is next (the next "Not yet delivered" row: lesson name, theme, grammar point, grouping) and confirm it is still wanted before generating. Let them ask for something different (another theme or activity, skip ahead).
  - **If there is no path**: offer to design one, clearly optional. If they decline, build a single lesson.
  - Then go to Step 3 and Step 4 (or Step 2 first if a new path was just approved).
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
5. **Levels**: lowest CEFR, highest CEFR, and the median. If unsure, ask what students can do and infer a level from `references/cefr-levels.md`.
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
- **A class name that already exists**: if the slug of a new class name matches an existing record, say the record already exists and ask whether this is the same class (load it, Step 0) or a different class that needs another name. Never overwrite a record.
- **Kids and teens**: keep every topic and activity age-appropriate, and honor every exclusion the teacher gives.
- **Unknown or uniform levels**: if the teacher cannot give levels, ask what students can do and infer a level. If every student is at a single level, record that. Support, Core, and Stretch tiers still apply (see Step 5).
- **Mono-lingual vs mixed**: infer the shared first language from the location for a mono-lingual class and use the L1 interference notes in `references/class-interview.md` silently. For a mixed class, keep activities English-only and culture-sharing, with no translation-dependent vocab.
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
- Level spread: lowest ..., highest ..., median ...
- Backgrounds: ... (mono-lingual / mixed)
- Shared interests: ...
- Goal: ...
- Session length and frequency: ...
- Activity choice: ...
- Key skills weighting: ...
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
- **Wide spread**: if the spread is wider than two CEFR bands (for example A2 to C1), tell the teacher plainly and offer parallel tracks for the lessons where tiers would stretch too far. Tiers remain the default unless the teacher chooses parallel tracks. See `references/differentiation.md`.

**Do not generate any lesson immediately after presenting the path.** Ask whether they want adjustments or want Lesson 1 generated. Only then proceed to Steps 3 and 4 for Lesson 1. Never generate all lessons at once, and ask before each later lesson, using that row's theme, grammar point, activities, and grouping as the spec.

Once the teacher approves the path, save it into the class record with every Status set to "Not yet delivered". Then offer to export the path table as a `.docx` (Step 6).

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
- **Student handout**: only if teacher's notes are integrated or end-only. Default yes.

Ask these once. For a path, ask before Lesson 1 and reuse the answers unless the teacher wants a change. Save the answers to the Profile in the class record.

**Do not open the plan with a class-details or framework header.** Start straight in with the lesson content.

Unless the teacher specifies their own format, include in this order:
1. **Objective(s)**: a can-do statement ("Students will be able to...").
2. **Warm Up Conversation**
3. **Vocab Review** (from Lesson 2 onward, or any returning class)
4. **Fresh Vocabulary**
5. **Grammar Focus**
6. **Activities** (2 to 3, each with a suggested grouping and three versions)
7. **Wrap-up**
8. **Home Assignment** (unless the teacher opted out)
9. **Materials needed**
10. **Time Summary**
11. **Teacher's Notes** (if not turned off)
12. **Student Handout** (a second, notes-free copy, unless the teacher opted out)

### Time Summary

Timings are suggestions the teacher can flex. Do not put a time next to each heading. Collect all suggested timings in one Time Summary section near the end. Non-activity stages should sum close to the class length. The activity options can add up to more than what remains: that is intentional headroom, not an error.

### Warm Up Conversation

Placed right after the Objective. A short, genuine check-in the whole class can answer in pairs or groups: 2 to 3 open-ended questions about their lives, in the spirit of "Any highlights from your week?" or "What are you looking forward to this weekend?". Write fresh questions for every lesson and vary the angle. Format as a bulleted list, and carry that format into the Student Handout. These are not lesson-topic teasers and not one-to-one rapport questions: every student must be able to answer regardless of level.

### Vocab Review (recycling)

Before new vocabulary, recycle old words:
- Pull 2 to 4 items from the class Vocabulary Log. Prioritize items from lessons that are neither the most recent nor already reviewed several times, so review rotates through everything taught.
- Use a quick format: a sentence with the word blanked out, or "use this word in a sentence about..." tied to the class's shared interests. Run it in pairs.
- Skip this section for a class's first lesson.

### Fresh Vocabulary

Every lesson gets a standalone new-vocabulary section:
- Use the count the teacher specified (default 6 to 10), tied to the lesson's theme and the class's shared interests. Examples must read as written for this class.
- **Pitch vocab one CEFR level above the class median.** This is a deliberate stretch (use `references/cefr-levels.md`).
- **One list for the whole class.** The Support tier adds scaffolds for the same words (sentence frames, example hints, a word bank), never a shorter or easier list.
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

The lesson's main teaching point, taught once at the median level. The presentation always includes:
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

If the teacher opted in, produce a second copy after the main plan. It is the same document, same wording, same formatting, minus exactly three things: all Teacher's Notes, Materials needed, and Time Summary. Label it with just the lesson name plus "Student Handout". Do not add any caption describing what was removed. If exporting to `.docx`, export the plan and the handout as separate files.

### Updating the class record

After finalizing a lesson:
- Set its path row Status to "Delivered <date>".
- **Before** drafting Fresh Vocabulary, read the Vocabulary Log. **After** finalizing the list, append the new items under a new dated heading.
- If the teacher says the class already knows certain words, log those too.

## Step 5: Differentiation and common pitfalls

The full rules are in `references/differentiation.md`. In short:
- **Anchor and tiers**: target the median. Every activity has Support, Core, and Stretch versions of the same task. Fresh vocab is one list for everyone, never a shorter list.
- **Grouping**: mixed-level pairs for peer help, same-level trios for Support and Stretch tasks, whole class for warm-up and wrap-up. Rotate partners and who goes first.
- **Wide spread**: more than two CEFR bands triggers an offer of parallel tracks. Tiers stay the default.
- **Single level**: if all students are at a single level, tiers still apply as scaffolding and extension.
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
2. **If there is a working connected location**, ask where to save (unless the teacher already said), reuse that answer for the session, and write the file there.
3. **If there is no connected location, or you cannot confirm one, say so in plain language** and share the file directly in the chat instead. Do not fail silently.
4. **After creating the file, confirm it exists and is reachable by the teacher** before saying it is done. A script exiting without error is not proof.

**If the environment has a dedicated Outputs or Files panel** (for example Claude's Cowork-style task view): a file the assistant writes directly shows up there, but a file produced by running a script does not. After running the converter, always send or attach each generated file explicitly. In the same message, name every file exactly and tell the teacher where to find it ("click **Outputs** in the panel on the right").

**If the environment has no way to run code or a shell** (a plain chat assistant with no tools): say plainly that this environment cannot generate a Word file, and do not claim to have created one. Output the content as clean, well-structured markdown (proper `#` and `##` headings, bold, bullet and numbered lists) and tell the teacher to copy and paste it into Word or Google Docs. If a code interpreter tool is available, try the conversion instead.

**Default, on any harness with shell access**: write the plan to a markdown file, then run the converter:

```bash
python3 skills/english-classroom/scripts/md_to_docx.py <plan>.md <output>.docx
```

If `python-docx` is missing, run `pip install python-docx` and retry. The script handles `#`, `##`, `###` headings, bold and italic inline markup, lists, `> ` blockquote lines (rendered indented, gray, italic, used for Teacher's Notes), and markdown tables (real Word tables with a bold header row, used for the path). Write the source markdown accordingly:
- Lesson title as `#`, each stage as `##`.
- Bold the vocab word in each `word – definition` line and italicize the example sentence.
- Prefix each Teacher's Note line with `> `.
- For a path, write the exact markdown table already used to present it.

Name files `<class-slug>-lesson-<n>.docx`, `<class-slug>-lesson-<n>-handout.docx`, and `<class-slug>-learning-path.docx`.

**After creating any file, tell the teacher exactly where it landed**: the full path in a connected folder, or a plain statement that it is attached in the chat. This applies to every file this skill writes: exports, handouts, and record files.
````

- [ ] **Step 3: Run to verify it passes**

Run: `cd ~/english-classroom && python3 tests/validate_skill.py TestSkillMd -v`
Expected: PASS. If `test_review_focus_rules` fails on "cannot generate a Word file", the sentence in Step 6 must read exactly "this environment cannot generate a Word file". If `test_no_em_dashes` fails, replace each reported character with a colon, comma, or period.

- [ ] **Step 4: Commit**

```bash
cd ~/english-classroom
git add skills/english-classroom/SKILL.md
git commit -m "Add english-classroom SKILL.md" -m "Co-Authored-By: Claude Sonnet 5.5 <noreply@anthropic.com>"
```

---

### Task 5: README and AGENTS.md

**Files:**
- Create: `README.md`
- Create: `AGENTS.md`

**Interfaces:**
- Consumes: `TestDocs`; names from `SKILL.md` (Steps, `~/eng-teaching-records/classes/`).

- [ ] **Step 1: Run the test to verify it fails**

Run: `cd ~/english-classroom && python3 tests/validate_skill.py TestDocs -v`
Expected: FAIL (`FileNotFoundError: README.md`).

- [ ] **Step 2: Write `README.md`**

````markdown
# english-classroom

**A planning assistant for teachers of English classes.** Tell it about your class once: how many students, their ages, levels, and backgrounds. It interviews you, drafts lessons (or a whole multi-lesson course) that work for everyone in the room at once, and remembers the class for next time.

Think of it as a fast, well-organized first draft, not a replacement for your judgment. Every activity, question, and note is meant to be reviewed, edited, or swapped.

For one student at a time, use [english-tutoring](https://github.com/ratherlegit/english-tutoring) instead.

It works inside Claude (Claude.ai, the Claude apps, or Claude Code), ChatGPT, and a few other AI assistants. See [Setting it up](#setting-it-up).

## Why teachers use this

- **It plans for the whole room.** Every activity comes in three versions of the same task: Support, Core, and Stretch. Stronger and weaker students work on the same thing at their own level, and the class stays together.
- **It starts from your class, not a template.** The class interview covers size, age, levels, and backgrounds, and everything after is built from that record.
- **It handles mixed and single-language classes differently.** A class that shares a first language gets targeted error prep. A mixed class gets English-only, culture-sharing activities with no translation needed.
- **It suggests how to group students.** Mixed-level pairs for peer help, same-level groups for support and stretch tasks, whole class for warm-ups and wrap-ups.
- **It plans ahead.** Ask for a 10-lesson course and it lays out the whole path before writing a single lesson, so you can adjust it first.
- **It never forgets what you have taught.** Every lesson checks the class vocabulary log, so words are never repeated by accident and older words come back for review.
- **You get a real, editable document.** Export any plan as a `.docx`, plus a clean student handout with your private notes removed.
- **It picks up where you left off.** Say "next lesson for [class]" and it already knows the class, the path, and everything taught so far.

## How a session works

1. **You tell it about your class.** Name, setting, number of students, ages, levels, backgrounds, interests, goal, and session length. Answer a handful of quick questions or a few open-ended prompts.
2. **Optionally, add a roster.** Names, pronouns, levels, and first languages for individual students, now or later. Plans use it to suggest pairs and spread personal interests across lessons.
3. **If you want more than one lesson, it plans the path first.** You see a table of every lesson's theme, grammar point, activities, and grouping before anything is written in full.
4. **It drafts the lesson.** Objective, warm-up, vocabulary, grammar focus, tiered activities, an exit task, and optional homework.
5. **You can ask for a clean student handout and a Word export.**
6. **It remembers the class.** The profile, roster, path, and vocabulary log are saved in `~/eng-teaching-records/classes/`.

## Wide level spreads

If your class spans more than two CEFR bands (for example A2 to C1), it tells you plainly and offers parallel tracks for the lessons where tiers would stretch too far.

## Setting it up

### Claude.ai or the Claude Desktop app

**On claude.ai (web):**
1. Click your account name in the bottom left, then **Settings**.
2. Open the **Plugins** tab.
3. Click **Add**, then **Add a marketplace**.
4. Enter `ratherlegit/english-classroom` and confirm. This adds the marketplace. It does not install the plugin yet.
5. Still on the **Plugins** tab, search for `english-classroom` and click **Add** on the result to install it.

**On the Claude Desktop app:**
1. Click **Customize** in the left sidebar, then open the **Plugins** tab.
2. Click **Add**, then **Add marketplace**.
3. Choose **Add from a repository**, enter `ratherlegit/english-classroom`, and confirm.
4. Search for `english-classroom` and click **Add** to install it.

Then type /english-classroom to get started.

### Claude Code

```
claude plugin marketplace add ratherlegit/english-classroom
claude plugin install english-classroom@english-classroom
```

### ChatGPT

Use ChatGPT's **Projects** feature:
1. Open [`skills/english-classroom/SKILL.md`](skills/english-classroom/SKILL.md), skip the short block between the two `---` lines at the top, and copy everything below it.
2. Download this repository as a ZIP from the green **Code** button and unzip it. The folder `skills/english-classroom/references/` contains six files.
3. Create a new Project, paste the copied text into its instructions, and upload the six reference files.
4. Ask for lessons as normal, for example: *"Plan a B1 lesson on the present perfect for my class of 12 teens."*

## Exporting to Word

Every lesson plan, handout, and path table can be exported as a `.docx`. This needs an environment that can run code. Without one, you get clean markdown to paste into Word or Google Docs.

## License

MIT
````

- [ ] **Step 3: Write `AGENTS.md`**

````markdown
# english-classroom

An ESL/EFL lesson-planning agent for group classes: interviews a teacher about a class, designs a differentiated multi-lesson learning path, and generates classroom-ready lesson plans with shared vocabulary and per-class memory. This file follows the [AGENTS.md](https://agents.md) convention, read automatically by OpenAI Codex, Cursor, Aider, Windsurf, Gemini CLI, RooCode/Cline, and several other coding-agent harnesses.

**Follow the instructions in `skills/english-classroom/SKILL.md`** for the full workflow. It references two supporting locations:

- `skills/english-classroom/references/` for the class interview, differentiation rules, CEFR calibration, lesson frameworks, and the activity banks
- `~/eng-teaching-records/classes/` where class profiles, rosters, learning paths, and vocab logs are read and written (create it if it does not exist). It lives outside the repo on purpose, so it survives updates. One-on-one student files in `~/eng-teaching-records/` belong to the english-tutoring skill and must not be touched.

For a single student, use the english-tutoring skill instead.

**Step 6, export as a .docx**, works on any harness with shell access: it runs `skills/english-classroom/scripts/md_to_docx.py` (requires `pip install python-docx`). In a chat harness where file access depends on a connected folder, verify a working delivery path first and do not assume a script-written file reaches the teacher (see Step 6 in `SKILL.md`). On a plain chat assistant with no code execution, say plainly that a Word file cannot be generated and give clean markdown to paste into Word or Google Docs.

**Platform notes:**
- **Codex, Aider, Cursor, Windsurf, RooCode/Cline, Gemini CLI, and other AGENTS.md-reading tools**: no setup needed. Check out this repo and follow `skills/english-classroom/SKILL.md`.
- **Claude Code**: install as a plugin (see `README.md`).
- **Anything else** (a custom system prompt, a custom GPT, a bespoke agent framework): paste in the body of `skills/english-classroom/SKILL.md` (everything below its YAML frontmatter) along with the contents of `skills/english-classroom/references/`. The `.docx` export needs code execution. Without it, plans are still fully usable as markdown.
````

- [ ] **Step 4: Run the whole suite**

Run: `cd ~/english-classroom && python3 tests/validate_skill.py -v`
Expected: all tests PASS (no failures, no errors).

- [ ] **Step 5: Commit**

```bash
cd ~/english-classroom
git add README.md AGENTS.md
git commit -m "Add README and AGENTS.md" -m "Co-Authored-By: Claude Sonnet 5.5 <noreply@anthropic.com>"
```

---

### Task 6: Plugin validation and sample-class exercise

**Files:**
- Create: `tests/manual-test-log.md` (results of this task and Task 7)

**Interfaces:**
- Consumes: the finished plugin. Writes only test records named `test-*` under `~/eng-teaching-records/classes/`, deleted at the end of this task.

- [ ] **Step 1: Validate the plugin manifest**

Run: `claude plugin validate ~/english-classroom`
Expected: validation passes. If the subcommand is not available in this Claude Code version, run `claude plugin --help`, record in the log that `validate` is unavailable, and continue.

- [ ] **Step 2: Exercise three sample classes**

Start a fresh session with the plugin loaded: `claude --plugin-dir ~/english-classroom` (or non-interactively `claude -p "<prompt>" --plugin-dir ~/english-classroom`). If the flag is not recognised, use the local-plugin option shown by `claude --help`. In that session, give these answers and record pass or fail for each check in `tests/manual-test-log.md`. Use class names beginning with `test-` so the records are easy to find and delete.

| Class | Inputs | Checks |
|---|---|---|
| `test-teens-b1` | 8 students, ages 13 to 15, B1 median (B1 to B1), all from Barcelona (Spanish/Catalan), interests football and music, 60 min weekly, path of 3 lessons, no exclusions | Record created under `classes/`. Path table has a Grouping column. Mono-lingual L1 notes appear in teacher's notes. Content is age-appropriate. |
| `test-adults-mixed` | 14 students, adults, A2 to B2, mixed countries (Brazil, Japan, Poland, Egypt), work-related interests, 90 min, single lesson | Mixed-class handling: English-only, culture-sharing, no translation. Vocab one level above median. Every activity has Support, Core, Stretch and a grouping. |
| `test-wide-spread` | 10 students, adults, A2 to C1, mixed, 60 min, path of 2 lessons | The skill states the spread is wider than two CEFR bands and offers parallel tracks. |

Then, in the `test-teens-b1` session, request Lesson 1 and Lesson 2 and check:
- Lesson 2 has a Vocab Review drawn from Lesson 1's log
- no word repeats across lessons
- the Wrap-up is an exit task checkable across the room
- the handout contains no Teacher's Notes, Materials needed, or Time Summary
- no em dash appears in body text (`grep` the exported markdown for the character U+2014)

- [ ] **Step 3: Exercise the Review Focus inputs**

In fresh sessions, record pass or fail:
1. Describe "one student, Maria, B1" as a class of 1: the skill must point to english-tutoring and offer to switch.
2. Create a class named `Test Teens B1` again: the skill must say the record already exists and ask whether it is the same class.
3. Request a kids class (ages 7 to 9) with exclusions "no scary topics": content must be age-appropriate and honor the exclusion.
4. A class where the teacher says "I don't know their levels": the skill must ask what students can do.
5. Ask for a `.docx` in an environment without code execution (a session started with tools disabled, or state the assumption): the skill must say it cannot generate a Word file and give pasteable markdown.

- [ ] **Step 4: Export check**

In the `test-teens-b1` session, accept the `.docx` export offer for Lesson 1 and its handout. Confirm both files exist on disk and open: `ls -l <path from the skill's message>` then `python3 -c "import docx,sys; docx.Document(sys.argv[1])" <file>` for each.

- [ ] **Step 5: Fix anything that failed**

For each failed check, edit the owning file (`SKILL.md` or a reference), add a phrase test in `tests/validate_skill.py` that pins the fix, and rerun `python3 tests/validate_skill.py -v` until it passes. Re-run the failed manual check.

- [ ] **Step 6: Clean up test records and commit the log**

List what will be deleted first, then delete only the `test-*` files:

```bash
ls ~/eng-teaching-records/classes/ | grep '^test-'
rm ~/eng-teaching-records/classes/test-*.md
ls ~/eng-teaching-records/classes/
cd ~/english-classroom
git add tests/manual-test-log.md tests/validate_skill.py skills
git commit -m "Record manual test results and fix issues found" -m "Co-Authored-By: Claude Sonnet 5.5 <noreply@anthropic.com>"
```

Expected: only `test-*` files removed. Any exported test `.docx` files outside the repo are reported to the user and left for them to delete.

---

### Task 7: Routing and english-tutoring regression

**Files:**
- Modify: `tests/manual-test-log.md` (append results)
- Verify: `~/.claude/skills/english-tutoring/skills/english-tutoring/SKILL.md` (already edited, uncommitted)

**Interfaces:**
- Consumes: both skills installed or loaded in the same session (`claude --plugin-dir ~/english-classroom` alongside the installed english-tutoring).

- [ ] **Step 1: Confirm the english-tutoring edit is present**

Run: `grep -c "use english-classroom instead" ~/.claude/skills/english-tutoring/skills/english-tutoring/SKILL.md && grep -c "handled by the separate \`english-classroom\` skill" ~/.claude/skills/english-tutoring/skills/english-tutoring/SKILL.md`
Expected: `1` and `1`.

- [ ] **Step 2: Routing prompts**

In a fresh session with both skills available, send each prompt and note which skill loads:

| Prompt | Expected skill |
|---|---|
| "Plan a B1 speaking class" | english-classroom (a class) or a clarifying question; never silently english-tutoring |
| "Lesson on conditionals for my 10 students" | english-classroom |
| "Next lesson for Maria" (Maria has a one-on-one record) | english-tutoring |
| "Write a 45-minute ESL lesson on phrasal verbs for my tutee" | english-tutoring |

If a prompt routes wrongly, adjust the `description` of the skill that mis-fired (keep it under 1024 characters, keep the "use X instead" sentence), add a phrase test for the change in `tests/validate_skill.py`, and re-run.

- [ ] **Step 3: english-tutoring regression**

In the same session, ask english-tutoring for "next lesson for Maria" (or any existing one-on-one record) and confirm it loads the record without asking for the full interview. Confirm `ls ~/eng-teaching-records/` still shows the one-on-one files unchanged and no new files appeared outside `classes/`.

- [ ] **Step 4: Record and commit**

Append the routing table results and the regression result to `tests/manual-test-log.md`.

```bash
cd ~/english-classroom
git add tests/manual-test-log.md
git commit -m "Record routing and regression results" -m "Co-Authored-By: Claude Sonnet 5.5 <noreply@anthropic.com>"
```

---

### Task 8: Publish (requires explicit user confirmation)

**Files:** none changed locally. Creates the GitHub repo and pushes two repos.

**Interfaces:**
- Consumes: a clean `main` in `~/english-classroom` with `python3 tests/validate_skill.py -v` passing, and an uncommitted description edit in `~/.claude/skills/english-tutoring`.

- [ ] **Step 1: Pre-publish checks (read-only)**

```bash
cd ~/english-classroom
python3 tests/validate_skill.py -v
git status --short
git log --oneline
gh auth status
gh repo view ratherlegit/english-classroom 2>&1 | head -2
cd ~/.claude/skills/english-tutoring && git status --short && git diff --stat
```

Expected: tests pass, working tree clean, `gh` authenticated as `ratherlegit`, `gh repo view` reports the repo does not exist, and the english-tutoring diff shows only `SKILL.md`.

- [ ] **Step 2: STOP and ask the user to confirm**

Do not continue until the user explicitly says yes to both, in these words or equivalent: "Create the public repo `ratherlegit/english-classroom` and push it" and "Commit and push the english-tutoring description change". If they answer yes to one, do only that one. Show them the commit log and the english-tutoring diff first.

- [ ] **Step 3: Create and push the new repo (after confirmation)**

```bash
cd ~/english-classroom
git branch -M main
gh repo create ratherlegit/english-classroom --public --source . --remote origin --push --description "ESL/EFL lesson planning for group classes: class interview, differentiated learning paths, shared vocabulary, per-class memory."
```

Expected: repo URL printed, `git status` shows `up to date with 'origin/main'`.

- [ ] **Step 4: Commit and push the english-tutoring change (after confirmation)**

```bash
cd ~/.claude/skills/english-tutoring
git add skills/english-tutoring/SKILL.md
git commit -m "Point group-class requests to english-classroom" -m "Co-Authored-By: Claude Sonnet 5.5 <noreply@anthropic.com>"
git push origin main
```

If `push` is rejected because the remote moved, run `git pull --rebase origin main`, re-check `git status`, and push again. Do not force push.

- [ ] **Step 5: Verify installation from GitHub**

```bash
claude plugin marketplace add ratherlegit/english-classroom
claude plugin install english-classroom@english-classroom
claude plugin list | grep english-classroom
```

Expected: the plugin appears in the list. Tell the user the repo URL and the install commands, and note that the new skill is now installed on this machine in addition to the local copy.
