# MW Eval Helper deployment - prompt 6 of 7

Rules for this whole series: use only Python's standard library; install and download nothing; make no network calls; reproduce file content exactly, character for character, without reformatting, fixing or improving anything; work only inside the install folder.

Content is everything between the `~~~~~` fence lines, excluding the fences. Each file ends with a single newline after its last line.

### File: `README.md` (complete file)
Create this file.

~~~~~markdown
# MW Eval Helper

Removes the copy-paste and transcription steps when scoring Scribbler prompt
outputs. Python standard library only; nothing is installed and nothing
leaves your machine.

## Daily use

1. Run prompt XYZ in Scribbler. Copy its output from the output pane.
2. In the helper window, choose the scorer template, enter the prompt name
   and version (and optionally the context files), and click **1. Prep scorer**.
   The completed scorer prompt is now on your clipboard.
3. Start a **fresh** Scribbler session, select the **same** context files,
   paste, and run.
4. Copy the scorer's JSON output and click **2. Log score**.

The status line confirms what was logged. Errors (malformed JSON, missing
placeholder, CSV open in Excel) are shown and nothing bad is written.

Open the window by double-clicking `run_helper.bat`, or in VS Code via
*Terminal → Run Task → MW Eval: open helper window*.

## Terminal mode (if the window won't open)

```
python mw_eval_helper.py templates
python mw_eval_helper.py prep -t MY_SCORER.md -p XYZ -v 1.4 -c "Protocol v3"
python mw_eval_helper.py log -n "optional note"
python mw_eval_helper.py status
```

`--in FILE` / `--out FILE` replace the clipboard if needed.

## Scorer templates

Put each template in `scorer_templates/` as `.md` or `.txt`.

- It must contain `<<<XYZ_OUTPUT>>>` exactly once, where XYZ's output goes.
- Optional first line `SCORER_VERSION: 1.0` is logged with every score.
  Bump it whenever you change the template.
- Ask for a single JSON object with fixed keys. Any structure works:
  nested scores become columns (`criteria.faithfulness`), lists of issues are
  stored as text with a `.count` column. New keys add new columns
  automatically without losing earlier rows.

`EXAMPLE_scorer.md` shows the expected shape. Replace it with your own.

## What gets recorded

`score_log.csv` (opens in Excel), one row per scored run:
run ID, timestamp, prompt name and version, scorer template and version,
context files, SHA-256 fingerprints of XYZ's output and the scorer JSON,
notes, then every field of the scorer JSON.

`archive/` keeps `<run_id>_xyz_output.txt` and `<run_id>_scorer.json` for
every run, so any score can be traced back to the exact text it judged.

Both contain confidential content; they are excluded in `.gitignore`. Keep
the folder on an approved company location.

## Safeguards

- Duplicate JSON is detected before logging.
- A score can't silently be logged without its prep step (you are asked).
- A prepped run survives closing the window (`status` shows it).
- If the CSV is open in Excel, nothing is written and you're told to close it.

## Self-test

`python -m unittest -v test_mw_eval_helper` (uses a temporary folder).
~~~~~

Reply with what you created or appended (path and part), then wait for prompt 7.
