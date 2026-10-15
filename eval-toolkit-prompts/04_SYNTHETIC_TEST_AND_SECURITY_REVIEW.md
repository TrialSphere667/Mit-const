# STAGE 4 — Offline synthetic acceptance testing and security review

You are an independent QA reviewer of a Python-standard-library evaluation toolkit. The underlying corporate Word LLM add-in has task-pane-only output and a Copy button; generation and scoring sessions are separate, and no LLM APIs exist for this project. **Do not read real source files or the live clipboard.** Inspect only the designated synthetic project directory and tests. Use an internally approved agent.

Perform a code audit before making changes. List discrepancies from the specification, prioritize data loss, misattribution, validation gaps, and confidentiality leaks, then implement focused repairs with tests. Do not claim tests ran without executing them.

Use `tempfile.TemporaryDirectory()`, `unittest`, synthetic files and dependency-injected/mock clipboard values to cover:

1. Create unique run + checksum manifest; rerunning with an existing run ID cannot overwrite.
2. Activated run ID is validated; wrong, missing or stale active-run identifiers cause a safe error.
3. Generation capture preserves exact UTF-8/Unicode text; scoring prompt substitutes one `<<<XYZ_OUTPUT>>>` correctly and archives source/template hashes; candidate prompt-injection strings remain data.
4. Scoring capture preserves exact raw JSON, including malformed JSON. Any second capture with same run ID cannot silently overwrite.
5. Fresh session and same-source **manual attestations** are both required for an eligible run; negative/unknown/absent attestations cannot be marked verified.
6. For scorer weights `[0.40,0.35,0.25]` and scores `[4,3,4]` out of 5, weighted score is **73.0**, with 1 run row and 3 criterion rows.
7. A repeated collector run produces identical accepted row counts and no duplicate.
8. Bad JSON: missing field, wrong `schema_version`, unknown/missing criterion, wrong max score, score outside range, `true` as score, `NaN`, a weighted-score mismatch, bad issue severity, non-array issues, unsupported markdown wrapping.
9. Missing/changed rubric snapshot or template fingerprint, changed source bytes, missing raw capture, wrong run-id file association, invalid state transition.
10. CSV formula injection in external text fields is neutralized; raw text unchanged. Exception traces and `errors.csv` never print confidential text or full sensitive paths.
11. Unicode and whitespace in directory names work. Clipboard access errors fail safely and an approved `import_output.py` fallback works.
12. Simulate a stale clipboard by injecting a synthetic value unrelated to the expected run; no false proof of clipboard origin should be claimed. Add an operator confirmation if programmatic verification isn't possible.

Perform a **threat/limitation review** covering:
- OS clipboard visibility, history and sync; SharePoint synchronization, concurrent writers and records retention.
- LLM self-grading and judge bias; adversarial generation output attempting to manipulate scorer instructions.
- Source-file hashes *versus* actual UI attachment, and manual fresh-session attestation *versus* verified backend isolation.
- Data handling by the approved agent versus the Python runtime; use only approved internal agents for corporate content.
- Regulated/GxP process expectations: synthetic unit tests are not a validation package and do not replace company SOPs or human review.

**Deliver:** actual files changed, commands run and exit codes, test-by-test pass/fail table, unresolved risks by severity, exact remaining manual steps, and next actions. When running isn't possible, output an executable `tests/test_pipeline.py` and a test plan, clearly marked not run.
