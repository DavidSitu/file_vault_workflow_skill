# Data Model

The manifest is the relationship index across artifact layers. It stores IDs, vault-relative paths, states, and source links. It should not duplicate full wiki or worksheet content.

## Manifest

Top-level shape:

```json
{
  "schemaVersion": 1,
  "rawFiles": [],
  "previewPages": [],
  "processedPages": [],
  "generatedOutputs": []
}
```

Distinguish two raw-file state fields:

- `status`: storage/preview lifecycle, such as `pending`, `stored`, `previewing`, `previewed`, `needs_review`, `failed`, or `ignored`
- `processingStatus`: whether the raw file has contributed to processed wiki knowledge: `unprocessed`, `in_review`, or `processed`

Minimum `rawFiles[]` record:

```json
{
  "id": "raw_upload-001",
  "uploadId": "upload-001",
  "originalName": "coin-scope.pdf",
  "path": "raw/upload-001/original.pdf",
  "mimeType": "application/pdf",
  "sha256": "optional-sha256-hex",
  "status": "stored",
  "processingStatus": "unprocessed",
  "previewPageIds": ["preview_upload-001_p001"],
  "processedPageIds": []
}
```

Minimum `previewPages[]` record:

```json
{
  "id": "preview_upload-001_p001",
  "rawFileId": "raw_upload-001",
  "pageNumber": 1,
  "path": "previews/upload-001/page-001.png",
  "status": "needs_review",
  "inspection": "selected"
}
```

Useful preview `inspection` values include `normal`, `blank`, `rotated`, `duplicate`, `selected`, and `excluded`.

Minimum `processedPages[]` record:

```json
{
  "id": "processed_p1_math_coins_scope",
  "path": "processed/P1/Math/coins/scope.md",
  "status": "active",
  "level": "P1",
  "subject": "Math",
  "topic": "coins",
  "materialType": "scope",
  "language": "zh",
  "sourceRawFileIds": ["raw_upload-001"],
  "sourcePreviewPageIds": ["preview_upload-001_p001"]
}
```

Allowed processed page statuses are `active`, `needs_review`, `superseded`, and `archived`. Generation should select `active` pages by default.

Minimum `generatedOutputs[]` record:

```json
{
  "id": "generated_p1_math_coins_2026-05-04_practice",
  "kind": "worksheet",
  "path": "generated/P1/Math/coins/2026-05-04-practice.md",
  "status": "saved",
  "level": "P1",
  "subject": "Math",
  "topic": "coins",
  "sourceProcessedPageIds": ["processed_p1_math_coins_scope"],
  "renderedArtifacts": [
    {
      "format": "pdf",
      "path": "generated/P1/Math/coins/2026-05-04-practice.pdf",
      "sourceMarkdownPath": "generated/P1/Math/coins/2026-05-04-practice.md"
    }
  ]
}
```

Generated output `kind` should be `worksheet` or `answer_key`. Generated records must link to processed wiki page IDs, not raw file IDs.

## Frontmatter

Processed wiki Markdown frontmatter must include enough metadata for search and provenance:

```yaml
---
schemaVersion: 1
id: processed_p1_math_coins_scope
status: active
level: P1
subject: Math
topic: coins
materialType: scope
language: zh
sourceRawFileIds:
  - raw_upload-001
sourcePreviewPageIds:
  - preview_upload-001_p001
sourceFiles:
  - raw/upload-001/original.pdf
previewFiles:
  - previews/upload-001/page-001.png
review:
  status: confirmed
  confirmedBy: user
---
```

Generated Markdown frontmatter must record derived-output provenance:

```yaml
---
schemaVersion: 1
id: generated_p1_math_coins_2026-05-04_practice
status: saved
kind: worksheet
level: P1
subject: Math
topic: coins
mode: simple_practice
questionCount: 10
language: zh
includeAnswerKey: true
sourceProcessedPageIds:
  - processed_p1_math_coins_scope
---
```

## Draft JSON Shapes

Agent wiki draft:

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

Practice draft:

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
  "answerKeyMarkdown": "# Answer Key\n\n1. ...",
  "renderedArtifacts": [
    {
      "format": "pdf",
      "kind": "worksheet",
      "path": "generated/P1/Math/coins/worksheet.pdf",
      "sourceMarkdownPath": "generated/P1/Math/coins/worksheet.md",
      "renderer": "markdown-pdf"
    }
  ]
}
```
