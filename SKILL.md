---
name: file-vault-workflow
description: Use when running the school file vault workflow: initialize or inspect a local vault/database, import and preserve raw school files, create or read previews and inspection artifacts, sort/classify material, draft reviewed processed wiki pages, generate worksheets or answer keys from confirmed wiki knowledge, export generated Markdown to polished PDF when requested, and verify manifest provenance across raw, preview, processed, generated, and rendered artifacts.
---

# File Vault Workflow

Use this skill for the local agent-led education-material vault workflow:

```text
user intent
-> local vault/database
-> raw file import
-> preview/inspection artifacts
-> agent-drafted sorting and wiki content
-> human-confirmed processed wiki
-> generated practice output
```

This is not a generic file manager. The useful behavior is source-traceable school-material processing with explicit review gates.

## Hard Rules

- Raw files are evidence, not knowledge.
- Raw originals must remain untouched after import.
- Preview files are inspection artifacts.
- Agent output is a draft until reviewed.
- Processed Markdown is the human-readable wiki layer.
- Processed Markdown is canonical knowledge only after human confirmation.
- Generated worksheets and answer keys must cite active processed wiki page IDs.
- Generated Markdown is the canonical generated source; PDF is a deterministic presentation artifact.
- Do not generate practice directly from raw PDFs, images, DOCX files, or previews.
- Keep vault paths relative inside `index/manifest.json`.
- Do not hide provenance or confirmation steps.
- Do not encode raw-file processing state by moving raw originals between folders after import.
- Do not save or require HTML files for generated questions; HTML may only be a transient renderer intermediate.

## Runtime Vault Layout

The vault/database is a local filesystem plus manifest:

```text
<user-selected-vault>/
  raw/
  previews/
  processed/        # human-readable wiki Markdown
  generated/
  index/
    manifest.json
```

Manifest top-level shape:

```json
{
  "schemaVersion": 1,
  "rawFiles": [],
  "previewPages": [],
  "processedPages": [],
  "generatedOutputs": []
}
```

Raw processing state belongs in the manifest, not in the raw file path. If a filesystem view is useful for humans, create derived indexes or reports under `index/`, but do not move the raw original to represent state.

## Reference Files

Read only the references needed for the user's task:

- `references/data-model.md`: manifest record shapes, raw/preview/processed/generated status fields, frontmatter, and draft JSON payloads.
- `references/workflow-steps.md`: vault creation, import, inspection, classification, wiki confirmation, practice generation, and provenance verification.
- `references/runtime-code-organization.md`: arranging runtime code before implementation, keeping entrypoints thin, and separating reusable behavior.
- `references/rendering-and-export.md`: Markdown-to-PDF export rules, rendered artifact records, and the no-saved-HTML rule.

## Current CLI Surface

Use the project CLI/debug commands when available:

```bash
PYTHONPATH=src python3 -m school_file_vault init <vault>
PYTHONPATH=src python3 -m school_file_vault status --vault <vault>
PYTHONPATH=src python3 -m school_file_vault import <source-file> --vault <vault> --preview
PYTHONPATH=src python3 -m school_file_vault preview <raw-file-id> --vault <vault>
PYTHONPATH=src python3 -m school_file_vault wiki-create --vault <vault> ...
PYTHONPATH=src python3 -m school_file_vault wiki-list --vault <vault>
PYTHONPATH=src python3 -m school_file_vault agent-plan-wiki <raw-file-id> --vault <vault> --draft-json <path-or-stdin>
PYTHONPATH=src python3 -m school_file_vault agent-accept-wiki --vault <vault> --draft-json <path-or-stdin> --review-status confirmed
PYTHONPATH=src python3 -m school_file_vault agent-generate-practice "<request>" --vault <vault> --draft-json <path-or-stdin>
PYTHONPATH=src python3 -m school_file_vault generated-list --vault <vault>
```

Current preview support may be limited. If preview generation fails for unsupported formats, preserve the raw file and report the limitation instead of pretending OCR or preview content exists.

## Workflow Summary

Default path:

```text
create/select vault
-> import raw material
-> inspect previews or notes
-> classify metadata
-> draft human-readable processed wiki Markdown
-> confirm wiki
-> generate practice from active processed wiki
-> optionally render PDF from generated Markdown
-> verify provenance
```

For detailed step rules, read `references/workflow-steps.md`.

For PDF export, read `references/rendering-and-export.md`.

For implementation or refactor work in a runtime project, read `references/runtime-code-organization.md` before editing code. Do not create `ARCHITECTURE/`, `TODO.md`, or `LOG.md` files inside this skill package unless the user explicitly asks for WF project files here.

## Quality Bar

Before finishing any workflow task, check:

- vault initialized
- raw file registered
- raw original preserved
- previews or inspection notes exist where possible
- wiki draft includes provenance
- confirmed wiki page is active
- generated output uses processed wiki IDs
- generated PDF, if present, is rendered from generated Markdown
- manifest links are vault-relative
- raw file processing status matches active processed wiki links
- no generated output depends directly on raw files

## Failure Handling

If raw inspection is poor, say so and keep the page in review.

If no active processed wiki source exists, do not generate practice. Create or confirm wiki knowledge first.

If the manifest has broken paths or missing IDs, stop and repair provenance before adding more generated output.

If the user asks for direct raw-to-worksheet generation, explain that it violates the architecture and offer the smallest correct alternative: create or confirm a processed wiki page first, then generate from that page.
