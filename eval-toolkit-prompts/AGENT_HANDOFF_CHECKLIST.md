# Copy/paste and agent handoff checklist

## Personal device → approved corporate agent

1. **Check your employer's policy first:** even a generic prompt in a personal GitHub repo could disclose details of internal tooling or workflow. This pack deliberately uses generic product labels. Remove/replace anything sensitive and avoid company/project names.
2. Keep only generic Markdown instructions in personal storage, if allowed. **Never upload** original source files, generated/scored clinical content, exact corporate paths, approved scorer templates, secrets, run logs or enterprise agent outputs.
3. On the work computer, **do not use a personally authenticated VS Code AI extension** for corporate workspace/code unless IT has approved it. Open an explicitly approved corporate LLM or coding agent. Paste the text of `00_ONE_SHOT_MASTER_PROMPT.md` into that agent. If work systems don't allow external prompts, follow corporate guidance instead.
4. For a chat-only corporate LLM, say: “I have no agent filesystem tools. Generate the exact first file, with its relative path and full code. Stop after that file.” Save generated files through an approved editor. Continue with `01`→`05` prompts for each stage.
5. Before executing code, ask a separate approved internal agent (or qualified colleague) to audit the generated code, using `04_SYNTHETIC_TEST_AND_SECURITY_REVIEW.md`.
6. Do not move source code, results, SharePoint links or screenshots **back** to your personal GitHub account. Keep all generated work artifacts in approved enterprise locations.

## Messages to use when changing agents

**Fresh agent handoff — build:**
> We are reconstructing a local Python/Windows prompt evaluation toolkit for a Word AI add-in with a task-pane Copy button. No model API, no cloud calls, and no external packages. A new scorer session must be created manually and must use identical reference versions. Please read the project files currently available in this *approved internal workspace*, state what has actually been implemented, identify gaps against the attached prompt specification, and continue using synthetic data only. Do not presume past agents created files unless they are present.

**Fresh agent handoff — QA:**
> Perform an independent offline audit of the local toolkit. Only inspect this designated synthetic project. Verify immutability, run provenance, version-locked rubric, fresh-session attestation semantics, complete scoring JSON, 73.0 synthetic weighted-score example, idempotent CSV, clipboard failures, data leakage risks, and test execution. Provide actual results, not assumptions.

**Fresh agent handoff — deployment:**
> The offline toolkit has passed synthetic unit tests. Help me deploy it in the approved local/SharePoint workspace with verified Python, safe `.cmd` launchers, optional Windows shortcuts and an SOP. Do not touch regulated data, change corporate settings, or claim add-in session isolation is technically verified.

## Operational acceptance checklist

- [ ] Source project created inside an approved corporate workspace.
- [ ] Only approved internal AI tools used.
- [ ] No corporate material placed in personal GitHub or an external LLM.
- [ ] No external network calls or third-party dependencies introduced.
- [ ] Generation task-pane Copy → user-triggered capture works (or fallback tested).
- [ ] Prepared scorer input includes captured response exactly once.
- [ ] Scorer session is genuinely fresh (operator procedure; manually attested).
- [ ] Same exact source document versions were manually reattached and attested.
- [ ] Run metadata, scorer rubric and source identifiers are version-locked.
- [ ] Raw generation and scorer records are immutable per run ID.
- [ ] JSON parser rejects corrupt/missing/inconsistent results.
- [ ] Synthetic scores 4/5, 3/5, 4/5 yield **73.0**.
- [ ] Report re-collection produces no duplicates.
- [ ] Errors do not reveal raw clinical text or full sensitive paths.
- [ ] SharePoint sync/access/retention and clipboard use have been approved.
- [ ] Human review and any applicable GxP validation requirements resolved before production use.

## Frequent mistakes to watch for

- “Fresh Word document” mistaken for a fresh add-in model context.
- Accidental self-scoring in the generation chat instead of independent scorer session.
- A model inventing run IDs/source hashes or reporting a score that the logger does not check.
- Retrospectively applying a changed rubric to archived results.
- Silent CSV append duplicates, spreadsheet formula injection and source text exposed by exception messages.
- Assuming Windows hotkeys work before `.lnk` shortcuts have been configured.
- Assuming a source file hash proves the add-in loaded that exact version.
- Claiming the project was tested on Word/the corporate add-in when only synthetic unit tests were run.
