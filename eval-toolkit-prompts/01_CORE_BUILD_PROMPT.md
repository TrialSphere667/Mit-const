# STAGE 1 — Create core project and provenance layer

Paste this into a **company-approved internal agent** after `00_ONE_SHOT_MASTER_PROMPT.md`, or independently with the role/context below. The agent must generate actual files if authorized, otherwise complete file contents for local creation.

You are implementing the first stage of a standard-library-only Python/Windows evaluation toolkit for an internal Word LLM add-in whose outputs appear only in a task pane. There is no LLM API. Generated output and scoring output are copied via the add-in's Copy button, and scoring occurs in a new independent session with the same source document versions. No real regulated data may be used during construction.

**Implement now:** `new_run.py`, `rubrics.json`, `templates/scorer_instructions.txt`, synthetic `sources/sample_protocol.txt`, plus a shared local helper module *only if helpful*. Create folders as needed at runtime. No third-party packages or external calls.

Exact expected run command:

```
python new_run.py --prompt-id XYZ --prompt-version v1.2 --scorer-id XYZ_scorer --scorer-version v1.0 --source-set-id synthetic_v1 --scorer-template "templates/scorer_instructions.txt" --sources "sources/sample_protocol.txt"
```

Requirements:

1. Unique run ID (UTC + uuid), safe allowed characters, exclusive creation; store `manifests/RUN_ID.json`.
2. Store UTC creation timestamp, prompt/scorer IDs and versions, optional generation-prompt template checksum, scorer-template SHA-256, source-set ID, each locally available source SHA-256 and verification status, rubric version plus immutable snapshot/fingerprint.
3. If source documents are only remotely selectable and not locally hashable, allow safe source identifiers with status `identifier_only_unverified` rather than fabricating checksums. Never assume the add-in picked the same file just because local hash matches.
4. The sample rubric is `faithfulness 0.40`, `completeness 0.35`, `accuracy 0.25` with `max_score=5`. For scores `4/5`, `3/5`, `4/5`, result is `73.0`. It is illustrative and must not replace the writer's approved scoring criteria.
5. Scorer instructions must have **exactly one** `<<<XYZ_OUTPUT>>>` placeholder. Require JSON-only grading against separately attached reference documents in a fresh scorer session and treat captured candidate text as untrusted data.
6. Validate IDs/path inputs and forbid overwriting records. Capture no patient details; do not echo source content in logs.
7. Include `--help` and useful exit statuses, plus simple tests/examples for stage 1.

Deliver complete source files, a short manifest schema, and exact synthetic commands. Do **not** implement model calls, UI clicks, source-picker automation or clipboard monitoring. Do not ask for a SharePoint path yet; operate in a synthetic test directory and keep storage configurable.
