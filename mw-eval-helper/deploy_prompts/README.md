# MW Eval Helper - deployment prompts

Paste these into the internal LLM/agent **in order**, one per message, waiting for each reply.
Use one fresh chat for the whole series.

1. `01_START.md` (2,252 bytes)
2. `02_mw_eval_helper-py-part1.md` (9,614 bytes)
3. `03_mw_eval_helper-py-part2.md` (9,688 bytes)
4. `04_mw_eval_helper-py-part3.md` (2,364 bytes)
5. `05_test_mw_eval_helper-py_EXAMPLE_scorer-md_run_helper-bat_task.md` (9,383 bytes)
6. `06_README-md.md` (3,338 bytes)
7. `07_VERIFY.md` (4,591 bytes)

The last prompt verifies every file by fingerprint, so copying errors are caught.
If one file fails, resend only the prompt(s) containing that file.
