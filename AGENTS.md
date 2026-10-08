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
