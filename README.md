# File Vault Workflow

`File Vault Workflow` is a Codex skill for the local school file vault workflow. It is designed for moving messy school materials through a reviewed, source-traceable path: raw files, previews, processed wiki knowledge, and generated practice.

The explicit trigger is `$file-vault-workflow`.

## Why It Exists

The `file_management` architecture is not a generic file manager or a direct raw-file-to-worksheet generator. It depends on trust boundaries:

- raw files are preserved evidence
- previews are inspection artifacts
- processed Markdown is trusted knowledge only after review
- generated worksheets and answer keys are derived from active processed wiki pages
- `index/manifest.json` links the artifact chain

Use this skill when you want Codex to follow that workflow instead of improvising a simpler but less reliable path.

## Install

Copy this folder to:

```text
~/.codex/skills/file-vault-workflow/
```

The source folder may be named `file_vault_workflow/`, but the internal skill name is `file-vault-workflow`. Invoke it with `$file-vault-workflow`, not `$file_vault_workflow`.

## Usage

Examples:

```text
use $file-vault-workflow and initialize a local vault
use $file-vault-workflow and import these raw school files with previews
use $file-vault-workflow and sort this imported worksheet into the processed wiki
use $file-vault-workflow and draft a wiki page from this raw upload, but keep it review-gated
use $file-vault-workflow and generate practice from active processed wiki pages
use $file-vault-workflow and verify manifest provenance for this generated worksheet
```

## What It Does

- initializes or inspects the local vault/database layout
- preserves raw school files without mutation
- uses previews or inspection artifacts as evidence
- sorts and classifies material into level, subject, topic, material type, and language
- drafts processed wiki content with source links
- requires confirmation before processed wiki content becomes active knowledge
- generates worksheets and answer keys from active processed wiki pages
- verifies provenance through `index/manifest.json`

## What It Does Not Do

- it does not treat raw PDFs, images, or DOCX files as trusted knowledge
- it does not bypass the processed wiki review gate
- it does not generate practice directly from raw files
- it does not replace the `file_management` architecture docs
- it does not silently accept uncertain AI classification
- it does not own polished UI, DOCX export, cloud sync, or exam-style generation before the core workflow is reliable

## Relationship To `file_management`

This skill is based on the `file_management` architecture contract:

```text
raw files
-> previews
-> processed wiki
-> generated worksheets
```

Use the runtime code and manifest as the final source of truth for actual behavior. Use the architecture docs for intent, boundaries, and workflow rules.
