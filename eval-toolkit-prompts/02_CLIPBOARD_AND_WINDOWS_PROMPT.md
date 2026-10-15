# STAGE 2 — Task-pane clipboard bridge and Windows launchers

You are continuing a **company-approved, local-only** Python project for evaluating outputs copied from an internal Microsoft Word LLM add-in. The add-in's response is available only in its task pane, with a dedicated Copy button and Ctrl+C support. The user starts a **fresh independent scorer session** manually and reselects the identical source versions. There is no direct model API. Stage 1 created manifests, rubric, template and synthetic source files. Inspect those real local files before integrating; if you cannot inspect files, request the manifest/template schema, not confidential content.

**Implement now:** `clipboard_bridge.py`, `quick_capture.py`, `import_output.py`, `Activate_Run.cmd`, `Capture_Generation.cmd`, `Capture_Scoring.cmd`, `Finalize_Run.cmd` (the latter may invoke the stage-3/4 scripts when present), and isolated unit tests.

Required sequence:

```
python quick_capture.py activate RUN_ID
# Operator runs generation in add-in and clicks Copy
python quick_capture.py generation
# Operator manually opens fresh scorer session and selects same source files
# Operator pastes clipboard scoring instructions, scores, and clicks Copy
python quick_capture.py scoring
```

1. Read OS clipboard only in response to a foreground user command; never start a background watcher. Use Python standard library. `tkinter` is acceptable only if available; alternatively use correctly typed Win32 `ctypes` clipboard functions (`CF_UNICODETEXT`, handle/pointer sizes, lock/unlock and try/finally). No third-party clipboard libraries.
2. Support `--run-id` override for generation/scoring; check active-run file only when explicit. Validate run exists and stage allows requested capture.
3. On generation: save **exact clipboard plain text** as `generated/RUN_ID.txt` exclusively, record SHA-256 and capture UTC timestamp without disclosing content on stdout. Build `prepared/RUN_ID_scoring_prompt.txt` from the **versioned scorer template captured in the manifest**, replacing exactly one `<<<XYZ_OUTPUT>>>` with the generated output. Warn that copying the prepared scorer prompt to clipboard replaces its contents; never erase clipboard before generation is preserved. Archive prepared prompt and its checksum.
4. On scoring: save full copied JSON text unchanged as `inbox/RUN_ID.txt` via exclusive create, record checksum and timestamp; basic JSON parse may produce a safe warning but must not rewrite raw response.
5. Detect empty clipboard, wrong state, nonexistent run, excessively large or obviously incomplete copy if technically determinable, and stop with safe messages. A nonempty string cannot prove completeness; require user confirmation/copy check as needed.
6. Provide fallback `python import_output.py generation --run-id RUN_ID --file approved_local.txt` and analogous scoring path, for locked-down clipboard environments. No bypass of DLP controls.
7. `.cmd` scripts must be robust to spaces and Unicode folder names, use `%~dp0` and quoted paths, show non-sensitive errors, and select a verified interpreter. Avoid hard-coded corporate paths and privileged setup. `Finalize_Run.cmd` can wait for the confirmation/collector added later.
8. Optional keyboard shortcuts are Windows `.lnk` assignments made manually via shortcut Properties; do not claim `Ctrl+Alt+G` or `Ctrl+Alt+S` are automatic global hotkeys.
9. Write automated synthetic file-based tests and mock clipboard handling; never read live clipboard during unattended tests.

At the end document one real-world operator walkthrough, steps still manual (two the internal add-in sessions, source reattachment, Copy), and limitations. Never use Word document text/`Selection.Text` to read the add-in task pane.
