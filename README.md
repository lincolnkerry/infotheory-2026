# Information Theory 2026 Fall — course repo (lincolnkerry/infotheory-2026)

Interactive course homepage + AI TA + GitHub Classroom homework pipeline.

## Repo layout
```
index.html                  # interactive homepage (GitHub Pages)
CLAUDE.md                   # AI TA persona & policy (used by ai-ta.yml)
.github/workflows/ai-ta.yml # Claude answers student questions in Issues/Discussions
hw-template/                # template repo content for each Classroom assignment
grader/                     # -> goes into a SEPARATE PRIVATE repo (see below)
docs/DESIGN-ko.md           # full architecture design (Korean)
```

## One-time setup (≈30 min)
1. **Organization**: create `gist-infotheory-2026` (GitHub Classroom requires an org;
   personal accounts cannot host classrooms). Free for education.
2. **Classroom**: classroom.github.com → New classroom → link the org →
   upload roster (student ID ↔ GitHub username ↔ email).
3. **This repo**: push to `lincolnkerry/infotheory-2026`,
   Settings → Pages → deploy from `main` → homepage at
   `https://lincolnkerry.github.io/infotheory-2026/`.
   Add repo secret `ANTHROPIC_API_KEY` (for the AI TA workflow).
   Enable Discussions; create a "Q&A" category; create label `question`.
4. **Private grader repo**: create `gist-infotheory-2026/grader` (PRIVATE).
   Copy `grader/` contents there + add:
   ```
   roster.csv                # github,student_id,name,email
   rubric/hwN-rubric.md      # per-problem points & criteria
   solutions/hwN-solution.pdf
   CURRENT_HW                # single number, bump weekly
   ```
   Secrets: `CLASSROOM_PAT` (fine-grained PAT, contents:rw on org repos),
   `ANTHROPIC_API_KEY`, `SMTP_HOST/PORT/USER/PASS` (e.g. Gmail + app password).
5. **Each assignment**: Classroom → New assignment `hwN` → template =
   `hw-template` repo (put `problems.pdf` in it) → private repos, deadline on →
   paste the invite link into `HW[]` in `index.html`.

## Weekly routine (fully automatic after setup)
- Students accept invite → push PDF/Markdown before Mon/Wed 23:59.
- Cron in the grader repo fires past midnight: Claude grades vs rubric →
  `feedback/hwN-feedback.md` committed to each student repo → email sent →
  `gradebook/hwN.csv` updated. Manual run/regrade: Actions → "Grade homework"
  → enter hw number (optionally specific usernames, or dry-run).

## Security invariants
- Rubrics, solutions, API keys, SMTP credentials, gradebook: **only in the
  private grader repo**. Student repos and this public repo never contain secrets.
- Grades/corrections go only to each student's private repo + their email.
