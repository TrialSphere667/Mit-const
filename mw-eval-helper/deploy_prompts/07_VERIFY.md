# MW Eval Helper deployment - prompt 7 of 7 (VERIFY AND FINISH)

Rules for this whole series: use only Python's standard library; install and download nothing; make no network calls; reproduce file content exactly, character for character, without reformatting, fixing or improving anything; work only inside the install folder.

If you can't run commands, give me each command below and I'll run them in the VS Code terminal from the install folder and paste the results back to you.

## Step 1 - Fingerprint check
Save this as `_check_files.py` in the install folder and run `python _check_files.py`. It ignores line-ending differences.

~~~~~python
import hashlib, pathlib
expected = {
    "mw_eval_helper.py": "5039d10bb704bd3eb2d1c58fab5f0802cb25183788f5a75dddf9ced25d1f55e4",
    "test_mw_eval_helper.py": "2dc564e41eda79f2aef1272bb15da19505f5977da82b9d45039e636d9fd5fda7",
    "scorer_templates/EXAMPLE_scorer.md": "833efcb93598a6f1e65acd2fd48a9bb6eea4241e8cdf0fc7064d3644f5ce39e2",
    "run_helper.bat": "a3f4404683df934169f06f3e6ca0f8a30d150106196fd23c969cf03f8a95ec38",
    ".vscode/tasks.json": "8a8b194916e166e23f6b944c687551a758a063b7c99c958c72db497f0687d8a3",
    ".gitignore": "0a0fd4baaa7353110eaf3f929df7ef9e3d4433abde7a265132443957c4c97f70",
    "README.md": "395d10f3445c2995f5ffb56ebc237be7294a306e5639c5590ca05cc67ed641f5"
}
bad = 0
for name, h in expected.items():
    p = pathlib.Path(name)
    if not p.exists():
        print("MISSING ", name); bad += 1; continue
    got = hashlib.sha256(p.read_text(encoding="utf-8").replace("\r\n", "\n").encode("utf-8")).hexdigest()
    ok = got == h
    bad += not ok
    print("OK      " if ok else "MISMATCH", name)
print("ALL FILES OK" if not bad else f"{bad} file(s) need fixing")
~~~~~

Every file must print OK. For any MISMATCH, tell me which file; I'll resend the prompt(s) containing it so you can recreate it from scratch (for split files, recreate from part 1). A common cause is a missing or extra blank line where parts were joined. Don't patch files by guessing. When all are OK, delete `_check_files.py`.

## Step 2 - Self-test
Run `python -m unittest -v test_mw_eval_helper`. All 10 tests must pass (they use a temporary folder). If any fail, show me the output and stop.

## Step 3 - Smoke test
1. Create `smoke_xyz.txt` containing: `Smoke test output: 120 participants, dose 5 µg/kg, ≥18 years, Änderung.`
2. Create `smoke_scorer.json` containing: `{"weighted_score": 90.0, "criteria": {"faithfulness": 5, "completeness": 4}, "issues": [], "summary": "smoke test"}`
3. Run `python mw_eval_helper.py prep -t EXAMPLE_scorer.md -p SMOKE -v 0.0 --in smoke_xyz.txt --out smoke_prompt.txt`. Confirm smoke_prompt.txt contains the smoke text with µ, ≥ and Ä intact, and no `<<<XYZ_OUTPUT>>>`.
4. Run `python mw_eval_helper.py log --in smoke_scorer.json`. Confirm score_log.csv has one row with prompt_name SMOKE and weighted_score 90.0.
5. Delete smoke_xyz.txt, smoke_scorer.json, smoke_prompt.txt, score_log.csv and the two SMOKE_* files in archive/. Confirm `python mw_eval_helper.py status` prints "No pending run."

## Step 4 - Clipboard
Run:
~~~~~text
python -c "import mw_eval_helper as m; m.cli_clip_set('Clipboard test µ ≥ Ä'); print(repr(m.cli_clip_get().strip()))"
~~~~~
It should print `'Clipboard test µ ≥ Ä'`. (This overwrites my clipboard.)

## Step 5 - Window
Run `python -c "import tkinter; print('tkinter OK')"`. If OK, run `python mw_eval_helper.py` and ask me to confirm a window titled "MW Eval Helper 1.0.0" opens with EXAMPLE_scorer.md in the template dropdown. If tkinter isn't available, remind me to use terminal mode (see README.md).

## Step 6 - My scorer templates
Ask me for my existing scorer templates (paste them or give paths). For each:
1. Save a copy in `scorer_templates/` as .md or .txt (never edit my originals).
2. Propose replacing the spot where I paste the output being evaluated with `<<<XYZ_OUTPUT>>>`, exactly once.
3. Propose a first line `SCORER_VERSION: 1.0` (or the template's existing version).
4. Check it asks for one JSON object with fixed key names; if not, suggest wording, but don't change the criteria, weights or rules.
5. Show me the proposed edits and wait for my approval before saving.
Then run `python mw_eval_helper.py templates` and ask whether to keep EXAMPLE_scorer.md.

## Step 7 - Report
Summarise: install folder, Python command, result of each step, templates set up, anything I still need to do. Day to day I start the tool by double-clicking run_helper.bat, or in VS Code via Terminal > Run Task > "MW Eval: open helper window".
