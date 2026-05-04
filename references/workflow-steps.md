# Workflow Steps

## 1. Create Or Select Vault

Initialize an existing writable folder as the local database.

Expected result:

- `raw/`
- `previews/`
- `processed/`
- `generated/`
- `index/manifest.json`
- selected vault config updated

Validate with `status`.

## 2. Import Raw Material

Register each source file into `raw/<uploadId>/original.<ext>`.

Capture:

- upload ID
- original filename
- vault-relative raw path
- MIME type if available
- SHA-256 hash if implemented
- status
- processing status: `unprocessed`, `in_review`, or `processed`
- preview page IDs

Never mutate the user's original source file or the stored raw copy.

## 3. Read Raw Through Inspection Artifacts

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

## 4. Sort And Classify

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

Useful `materialType` values include `scope`, `worksheet`, `answer_key`, `exam`, `notes`, and `mixed`.

Do not silently invent missing metadata. If a field is inferred, say it is inferred.

## 5. Draft Processed Wiki

The processed layer is the wiki. A processed file must be human-readable Markdown, not opaque JSON, dumped extraction text, or machine-only metadata.

Draft Markdown as reviewed knowledge, but do not make it trusted until accepted.

A processed page should summarize:

- what the material covers
- key concepts
- source scope
- question patterns
- answer-key notes if present
- provenance links to raw and preview records

Processed wiki pages live under `processed/<level>/<subject>/<topic>/`.

A processed wiki page should be readable directly in a Markdown viewer or text editor. Use frontmatter for IDs, status, metadata, and provenance, then use normal Markdown headings and prose for the actual knowledge.

A confirmed page should become `active`. A non-confirmed page should remain `needs_review`.

## 6. Confirm Wiki

Only write processed wiki when the user has reviewed or explicitly accepted the draft.

Review status mapping:

- `confirmed` -> active processed knowledge
- `needs_review` -> non-active draft-like processed record

Generation should default to active pages only.

## 7. Generate Practice

Before generating, retrieve active processed wiki pages matching the request.

The agent may draft:

- worksheet Markdown
- answer key Markdown
- question count
- mode
- question type
- source processed page IDs

Generated output must save under `generated/<level>/<subject>/<topic>/`.

Generated records must link to:

```json
"sourceProcessedPageIds": ["processed_..."]
```

Do not use `sourceRawFileIds` as generation sources.

## 8. Verify Provenance

After wiki creation or generation, inspect:

- `index/manifest.json`
- processed Markdown frontmatter
- generated Markdown frontmatter
- rendered PDF artifact paths if present
- source IDs and paths
- active or needs-review status

A valid chain looks like:

```text
rawFiles[]
-> previewPages[]
-> processedPages[] active
-> generatedOutputs[]
-> rendered PDF paths, optional
```
