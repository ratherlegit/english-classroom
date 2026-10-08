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

If your class spans more than two CEFR bands (for example A2 to C1), or splits into two clusters (half at A2 and half at B2, where an average level would fit nobody), it tells you plainly and offers parallel tracks for the lessons where tiers would stretch too far. Not sure of your students' levels? It starts the first lesson with a short level check and updates the class record afterward.

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

Then type `/english-classroom` to get started.

### Claude Code

```
claude plugin marketplace add ratherlegit/english-classroom
claude plugin install english-classroom@english-classroom
```

Then ask in plain English, for example *"Plan a lesson for my class of 12 B1 teens."* To invoke it explicitly, the command is `/english-classroom:english-classroom`.

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
