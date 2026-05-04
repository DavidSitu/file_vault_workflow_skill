# Runtime Code Organization

Use this when applying the file vault workflow to the actual runtime codebase, especially before adding new vault behavior or refactoring a messy implementation.

Do not create `ARCHITECTURE/`, `TODO.md`, or `LOG.md` files inside this skill package unless the user explicitly asks for WF project files here.

## Rules

- Inspect the current code first; runtime code is the source of truth for actual behavior.
- Keep `main`, CLI, server, or UI entrypoints thin: parse input, call workflow functions, format output.
- Put reusable behavior in named subsystem modules or functions, not inside the entrypoint.
- Separate behavior areas when the codebase is large enough: vault setup, raw import, preview creation, manifest persistence, processed wiki writing, practice generation, and PDF export.
- Define public entrypoints for each behavior area and avoid importing another subsystem's internal helpers directly.
- Refactor one boundary at a time and preserve behavior before adding new features.
- Run the smallest useful verification after moving or separating code.

## Suggested Behavior Areas

- Vault setup: creates folder layout and manifest.
- Raw import: stores immutable originals and raw manifest records.
- Preview creation: creates inspection artifacts and page records.
- Manifest persistence: loads, validates, and saves `index/manifest.json`.
- Processed wiki: writes human-readable reviewed Markdown and processed-page records.
- Practice generation: writes worksheet/answer-key Markdown and generated-output records.
- Export: renders generated Markdown into optional PDF artifacts.
