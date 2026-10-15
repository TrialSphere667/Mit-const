# STAGE 5 — Windows + SharePoint deployment and user handover

You are a company-approved internal deployment engineer. The Python project was reconstructed using only standard-library scripts. The add-in's output lives in a Word task pane and is copied with its Copy button; users manually run generation and a **new independent scoring session** with the identical source versions. The project must not call LLM APIs or third-party services. Begin with synthetic data, and do not guess organization paths or override corporate security controls.

## Task A: Preflight (do not make assumptions)

1. Inventory actual files and command-line interfaces, note deviations from expected project.
2. In approved Windows terminal, check `py -3 --version`, `python --version` and interpreter location, `python -m unittest discover -s tests -v`; do not install packages without approval.
3. Confirm OS clipboard-read capability is supported and **explicitly approved** using a synthetic clipboard example. If access is blocked, verify the file-import fallback; never disable a security setting.
4. Ask operator for the **approved locally synced SharePoint directory** if needed, rather than guessing a path. Verify sync availability but do not change SharePoint permissions.
5. Check for approved use of local `.cmd` launchers, `.lnk` shortcuts, and internal AI coding tools. A personal-account AI extension inside VS Code must not be used for corporate content unless expressly authorized.

## Task B: Install without privilege escalation

- Configure portability under a root directory that may contain spaces/non-ASCII characters.
- Keep only reusable code/templates under `code/` or project root; runtime data under approved `manifests/`, `generated/`, `prepared/`, `inbox/`, `reports/`, `state/`.
- Prefer single operator/single report writer while files sync. Do not put data in a Git repository; use a local `.gitignore` only if allowed, excluding all real data, logs, generated outputs and confidential templates.
- Make `.cmd` scripts testable by Explorer double-click and console execution; no hidden subprocess and no hard-coded absolute path.
- Optionally create Windows **shortcuts** to Capture_Generation.cmd and Capture_Scoring.cmd, and manually assign available `Ctrl+Alt+G` / `Ctrl+Alt+S` using shortcut Properties. Never claim the toolkit installs global hotkeys automatically. If shortcut creation or policy is restricted, use double-click/terminal.

## Task C: Run a synthetic operator acceptance test

Have the user run (with a synthetic data file only):

```
python new_run.py --prompt-id XYZ --prompt-version v1.2 --scorer-id XYZ_scorer --scorer-version v1.0 --source-set-id synthetic_v1 --scorer-template "templates/scorer_instructions.txt" --sources "sources/sample_protocol.txt"
python quick_capture.py activate RUN_ID
```

Then explicitly identify what must occur in the Word add-in for a **real** evaluation:

1. Open generation session; attach source file set; run versioned generation prompt.
2. Click Copy; run generation capture launcher; verify safe confirmation and prepared scorer instructions were created/copied.
3. Open a *fresh independent* scorer session; select same source file versions; paste prepared scorer input and run it.
4. Click Copy; run scoring capture launcher.
5. Confirm both attestations and reviewer ID; collect results; inspect `reports/runs.csv`, `reports/criterion_scores.csv`, and `reports/errors.csv`.

Perform this exercise with synthetic material in the internal add-in only if policy permits it, else rely on file-based simulation and label add-in integration *untested*.

## Task D: Documentation

Create `DEPLOYMENT.md` and concise daily `SOP.md` containing exact system setup, discovered Python path, approved storage root, version-change procedure, operator clicks, commands/shortcuts, known limitations, data retention, backup/SharePoint sync issues, handling of failed/duplicate/incorrect captures, how to reopen or reproduce old runs, troubleshooting and contacts for IT/owners. Do not place confidential paths in documents intended for personal GitHub.

Finally provide an **acceptance report**:
- Verified locally / not tested / requires company approval.
- Test commands/results, hashes/versioning checks and any failed tests.
- Manual steps that remain, especially new scorer session and source reattachment.
- Any policy decisions necessary before using real clinical/regulatory material.
