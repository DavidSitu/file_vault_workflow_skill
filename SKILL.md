---
name: file-vault-workflow
description: Use when running the school file vault workflow: initialize or inspect a local vault/database, import and preserve raw school files, create or read previews and inspection artifacts, sort/classify material, draft reviewed processed wiki pages, generate worksheets or answer keys from confirmed wiki knowledge, and verify manifest provenance across raw, preview, processed, and generated artifacts.
---

# File Vault Workflow

Use this skill for the local agent-led education-material vault workflow. The product flow is:

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
- Processed Markdown is canonical knowledge only after human confirmation.
- Generated worksheets and answer keys must cite active processed wiki page IDs.
- Do not generate practice directly from raw PDFs, images, DOCX files, or previews.
- Keep vault paths relative inside `index/manifest.json`.
- Do not hide provenance or confirmation steps.

## Runtime Vault Layout

The vault/database is a local filesystem plus manifest:

```text
<user-selected-vault>/
  raw/
  previews/
  processed/
  generated/
  index/
    manifest.json
```

Manifest minimum shape:

```json
{
  "schemaVersion": 1,
  "rawFiles": [],
  "previewPages": [],
  "processedPages": [],
  "generatedOutputs": []
}
```

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

## Workflow

### 1. Create Or Select Vault

Initialize an existing writable folder as the local database.

Expected result:

- `raw/`
- `previews/`
- `processed/`
- `generated/`
- `index/manifest.json`
- selected vault config updated

Validate with `status`.

### 2. Import Raw Material

Register each source file into `raw/<uploadId>/original.<ext>`.

Capture:

- upload ID
- original filename
- vault-relative raw path
- MIME type if available
- SHA-256 hash if implemented
- status
- preview page IDs

Never mutate the user's original source file or the stored raw copy.

### 3. Read Raw Through Inspection Artifacts

Prefer previews, extracted text, OCR output, or placeholder notes. Raw files are evidence; do not treat them as final knowledge.

For each imported file, inspect:

- original filename
- raw file ID
- preview page IDs
- page numbers
- preview status
- visible subject/level/topic clues
- language
- material type
- usable pages
- excluded pages
- uncertainty

If inspection is weak, mark the draft as uncertain and require user review.

### 4. Sort And Classify

Classify into reviewed metadata fields:

- `level`
- `subject`
- `topic`
- `materialType`
- `language`
- `sourceRawFileIds`
- `sourcePreviewPageIds`
- `sourceSummary`
- `extractedNotes`
- `scopeNotes`
- `questionPatterns`
- `answerKeyNotes`

Useful `materialType` values include:

- `scope`
- `worksheet`
- `answer_key`
- `exam`
- `notes`
- `mixed`

Do not silently invent missing metadata. If a field is inferred, say it is inferred.

### 5. Draft Processed Wiki

Draft Markdown as reviewed knowledge, but do not make it trusted until accepted.

A processed page should summarize:

- what the material covers
- key concepts
- source scope
- question patterns
- answer-key notes if present
- provenance links to raw and preview records

Processed wiki pages live under:

```text
processed/<level>/<subject>/<topic>/
```

A confirmed page should become `active`. A non-confirmed page should remain `needs_review`.

### 6. Confirm Wiki

Only write processed wiki when the user has reviewed or explicitly accepted the draft.

Review status mapping:

- `confirmed` -> active processed knowledge
- `needs_review` -> non-active draft-like processed record

Generation should default to active pages only.

### 7. Generate Practice

Before generating, retrieve active processed wiki pages matching the request.

The agent may draft:

- worksheet Markdown
- answer key Markdown
- question count
- mode
- question type
- source processed page IDs

Generated output must save under:

```text
generated/<level>/<subject>/<topic>/
```

Generated records must link to:

```json
"sourceProcessedPageIds": ["processed_..."]
```

Do not use `sourceRawFileIds` as generation sources.

### 8. Verify Provenance

After wiki creation or generation, inspect:

- `index/manifest.json`
- processed Markdown frontmatter
- generated Markdown frontmatter
- source IDs and paths
- active or needs-review status

A valid chain looks like:

```text
rawFiles[]
-> previewPages[]
-> processedPages[] active
-> generatedOutputs[]
```

## Draft JSON Shapes

### Agent Wiki Draft

```json
{
  "level": "P1",
  "subject": "Math",
  "topic": "coins",
  "materialType": "scope",
  "language": "zh",
  "sourceRawFileIds": ["raw_upload-001"],
  "sourcePreviewPageIds": ["preview_upload-001_p001"],
  "sourceSummary": "Short evidence-based summary.",
  "extractedNotes": "Observed notes from preview/raw inspection.",
  "scopeNotes": "What students are expected to learn.",
  "questionPatterns": ["Identify coin values", "Add small coin amounts"],
  "answerKeyNotes": "",
  "title": "P1 Math Coins Scope",
  "confidence": 0.72,
  "rationale": "Based on visible title and page content."
}
```

### Practice Draft

```json
{
  "level": "P1",
  "subject": "Math",
  "topic": "coins",
  "language": "zh",
  "mode": "simple_practice",
  "questionType": "short_answer",
  "questionCount": 10,
  "materialType": "worksheet",
  "sourceProcessedPageIds": ["processed_p1_math_coins_scope"],
  "worksheetMarkdown": "# Worksheet\n\n1. ...",
  "answerKeyMarkdown": "# Answer Key\n\n1. ..."
}
```

## Quality Bar

Before finishing any workflow task, check:

- vault initialized
- raw file registered
- raw original preserved
- previews or inspection notes exist where possible
- wiki draft includes provenance
- confirmed wiki page is active
- generated output uses processed wiki IDs
- manifest links are vault-relative
- no generated output depends directly on raw files

## Failure Handling

If raw inspection is poor, say so and keep the page in review.

If no active processed wiki source exists, do not generate practice. Create or confirm wiki knowledge first.

If the manifest has broken paths or missing IDs, stop and repair provenance before adding more generated output.

If the user asks for direct raw-to-worksheet generation, explain that it violates the architecture and offer the smallest correct alternative: create or confirm a processed wiki page first, then generate from that page.
