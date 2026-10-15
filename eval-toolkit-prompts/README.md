# Local Word-Add-in LLM Evaluation Toolkit — Prompt-only Reconstruction Pack

**Purpose:** This folder contains *instructions only*. Give them to a **company-approved internal coding agent or internal LLM** to recreate an offline clipboard-assisted evaluation toolkit on an approved corporate computer. The previous ZIP of executable Python scripts is **not needed**.

**Privacy / distribution boundary (read first):** This package deliberately describes a generic *internal Word AI add-in*. It contains no study information, sponsor names, patient information, corporate paths, secrets, model endpoints, or generated clinical outputs. Nevertheless, **confirm that company policy permits publishing technical instructions about the intended workflow in a personal repository**. If not, keep these prompts in an approved location or paste only what is permitted. Never commit code created against corporate files, run outputs, real prompts, internal architecture, or SharePoint paths to a personal GitHub account. Do not use a personally authenticated VS Code coding agent for corporate files without explicit authorization. Copying an external prompt into an internal LLM can also be governed by company policy.

## Which file to use?

| File | When to use |
|---|---|
| `00_ONE_SHOT_MASTER_PROMPT.md` | Start here. A complete standalone specification for an internal coding agent with sufficient context and file tools. |
| `01_CORE_BUILD_PROMPT.md` | Stage 1: metadata, manifests, prompt templates, rubric, immutable records. |
| `02_CLIPBOARD_AND_WINDOWS_PROMPT.md` | Stage 2: task-pane Copy button bridge, safe file-import fallback, Windows launchers. |
| `03_VALIDATION_AND_REPORTS_PROMPT.md` | Stage 3: strict scorer validation, provenance, idempotent CSV reports. |
| `04_SYNTHETIC_TEST_AND_SECURITY_REVIEW.md` | Stage 4: automated offline tests, negative tests, security review. |
| `05_CORPORATE_DEPLOYMENT_AND_HANDOVER.md` | Stage 5: approved SharePoint path, Windows setup, operator SOP. |
| `06_OPTIONAL_PRODUCT_IMPROVEMENTS.md` | Optional: evaluate approved UI automation and create a vendor/developer feature request. |
| `DATA_CONTRACT_AND_FIXTURES.md` | Consistent example JSON schemas, rubric mathematics and synthetic fixtures. |
| `AGENT_HANDOFF_CHECKLIST.md` | What to paste, how to restart with a different agent, and independent acceptance criteria. |

## Two practical ways to use this pack

**A. File-capable approved corporate agent:** Paste `00_ONE_SHOT_MASTER_PROMPT.md`. If the agent has read/write/terminal access in an approved sandbox, ask it to create the project and execute tests. Give it `DATA_CONTRACT_AND_FIXTURES.md` for exact examples if needed. Never claim execution until tests actually ran.

**B. Chat-based internal LLM, no filesystem:** Paste `00_ONE_SHOT_MASTER_PROMPT.md` and explicitly select *CHAT OUTPUT MODE*. Ask for one source file at a time in its entirety, with its exact destination relative path. Work through `01` → `02` → `03` → `04` → `05` as capacity allows. Create the produced files locally in the approved corporate workspace. Ensure a reviewer validates code before running it.

## Non-negotiable design

1. Generation and scoring are two **independent, fresh** add-in sessions. The user reselects the *same* source files for scoring. Neither the scripts nor a new Word document proves backend session isolation.
2. The add-in's text **only appears in its task pane**, but a dedicated **Copy** button (or Ctrl+C on selection) exposes text through the Windows clipboard.
3. The locally running Python scripts process clipboard/file text, prepare a scoring instruction, preserve original results, validate JSON and generate benchmark CSVs. No connection to the add-in's models/router is available or needed.
4. Scorer-run metadata comes from an independent machine-generated run manifest, **not from the scoring model**. Preserve original responses immutably.
5. Python standard library preferred; no SaaS, no telemetry, no third-party API, no unapproved package installs, and no background clipboard watcher.
6. Only **synthetic data** until the organization's security/GxP owners approve the use case and storage/retention policy.

## Expected human workflow after reconstruction

- `new_run.py` creates manifest + unique ID → `quick_capture.py activate RUN_ID`.
- Writer runs generation in the add-in; clicks Copy → `Capture_Generation.cmd` or terminal command.
- Script saves generation and places a complete, labeled scoring prompt on clipboard.
- Writer creates a **new** add-in scoring session, reselects exactly the same source file versions, pastes the scoring prompt and runs the scorer.
- Writer clicks Copy → `Capture_Scoring.cmd`.
- Writer attests actual fresh session + source reattachment → `Finalize_Run.cmd`.
- Collector checks the scorer response against rubric and manifest and produces Excel-friendly `reports/runs.csv`, `reports/criterion_scores.csv`, `reports/errors.csv`.

**Hotkeys:** Optional `.lnk` Windows shortcut assignments `Ctrl+Alt+G` and `Ctrl+Alt+S`; not pre-installed magic global hotkeys. Double-click `.cmd` or terminal commands work without shortcuts.

**Quality status:** A successful smoke test is not software validation for GxP or proof that a model's clinical grading is correct. Maintain human review and organizational change control as appropriate.
