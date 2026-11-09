# Claude Code — Koemi working agreement

Read `KOEMI_PROJECT.md` for product goals, data caveats, milestones, and architecture.

This is a practical portfolio project by an undergraduate data science student. Help me build code I understand and can explain. Prefer straightforward pandas, SQL, Python, FastAPI, and simple React/TypeScript over elaborate frameworks or clever abstractions.

- Inspect files and data before assuming anything.
- Work on one bounded task at a time. Explain the intended edit before doing it.
- Use small readable functions, plain variable names, minimal comments, and no boilerplate-heavy docstrings.
- Don't add dependencies, layers, design patterns, or infrastructure without a real need.
- Check data grain, missing values, duplicates, and join cardinality explicitly.
- Start with baselines. Don't add LLMs until the deterministic analytics work.
- Run relevant checks/tests and report actual results. Never invent metrics or pretend code ran.
- Don't touch unrelated files or refactor the whole project without a clear reason.
- Keep each response concise: objective, changes, commands, verification, next step.
- After each meaningful feature, give a short explanation of why it works and its limitations so I can defend it in interviews.
- I am responsible for reviewing and understanding the code; do not invent authorship, benchmarks, or business impact.

**Current priority:** Phase 1, profile the real Olist CSV files before choosing schema or modeling decisions.
