# Information Theory 2026 Fall — course homepage (lincolnkerry/infotheory-2026)

Public repo: the course homepage (GitHub Pages) and the problem sheets.
Everything private — solutions, rubrics, grader, gradebook — lives elsewhere.

## Layout
```
index.html          # homepage: schedule, HW badges (link to problems/hwset-N.pdf), submission guide, AI TA chat
problems/           # hwset-0.pdf … hwset-8.pdf — the canonical problem sheets
CLAUDE.md           # AI TA persona / policy
docs/               # design notes
```

## How the course pipeline is wired (2026)
- **Students**: one private repo each, `gist-infotheory-2026/it2026-<username>`,
  created from `gist-infotheory-2026/hw-template`. They push `submission/hwN.pdf`
  (or `.md`) by Wednesday 23:59 KST. A push-time workflow in the student repo
  fails the check with a note if the grader would not see the file name.
- **Problem sheets**: only here, under `problems/`. Student repos hold submissions
  only, so a corrected sheet never goes stale in 44 copies.
- **Grader** (private `lincolnkerry/it2026-solutions`): `grade_all.py` runs in
  GitHub Actions, grades each submission against the solution book, commits
  `feedback/hwN-feedback.md` to the student's repo, and pushes
  `gradebook/hwN.csv` + `INDEX.md` to private `lincolnkerry/it2026-gradebook`.
- **Trigger**: a Cloudflare Worker cron dispatches the grader Thursday 00:10 KST
  (15:10 UTC Wed) and verifies at 07:30 KST, opening an issue if grading did not run.
  GitHub's own `schedule` stays as a backup; a deadline-aware guard makes reruns idempotent.

## Invariants
- No secrets, solutions, or student data in this public repo.
- Grades and corrections go only to each student's private repo.
