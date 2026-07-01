# Dubai_Kurukku_sandhu — Project Instructions

This repo ships two **local Claude Code skills** in `.claude/skills/` so the whole team has the same workflow:

- **`/new-issue`** — create a structured GitHub issue (auto-detects bug/feature/chore/docs, writes acceptance criteria, adds it to the project board as Backlog).
- **`/start-task`** — pick up an assigned issue from Backlog, refine it, move it to Ready, create a feature branch, implement, commit, and raise a PR.

When a teammate types `/new-issue` or `/start-task`, invoke the matching skill in `.claude/skills/` before doing anything else.

## Project board
GitHub Project #9 → https://github.com/users/YUVARAJ-R-ai/projects/9
Repo: `YUVARAJ-R-ai/Dubai_Kurukku_sandhu`

## Key docs
- `README.md` — team onboarding + how to proceed
- `docs/research.md` — full research brief (features, tasks, tech, risks)
- `docs/architecture.md` — full pipeline diagram (Mermaid) — imagery → mask → graph → dashboard

## Branching
- `main` — protected, stable
- feature branches: `feat/<issue#>-short-name`, raised as PRs into `main` (or `dev` if created)
