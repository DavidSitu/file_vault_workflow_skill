# Rendering And Export

Use this when the user asks for polished, printable, rendered, or PDF output.

Generated Markdown is the canonical generated source. PDF is a deterministic presentation artifact.

Preferred flow:

```text
active processed wiki
-> generated worksheet.md / answer_key.md
-> deterministic Markdown-to-PDF renderer
-> worksheet.pdf / answer_key.pdf
```

Rules:

- Save generated Markdown before rendering PDF.
- Keep Markdown as the auditable source of truth.
- Treat PDF as a rendered artifact, not a separate knowledge source.
- HTML may be used as a transient renderer intermediate, but do not save or require HTML files for generated questions.
- Prefer a pinned Markdown-to-PDF path when available.
- Use fixed page size, margins, fonts, and CSS for stable output.
- Save PDFs beside their Markdown source under `generated/<level>/<subject>/<topic>/`.
- Record PDF paths in the generated output manifest record using vault-relative paths.
- If rendering fails, preserve the Markdown output and report the renderer limitation.
