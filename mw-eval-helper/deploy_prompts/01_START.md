# MW Eval Helper deployment - prompt 1 of 7 (START)

I'm going to give you a small Python tool as a series of 7 prompts. Your job is to recreate its files exactly on my computer, then verify them. Don't do anything beyond what each prompt asks.

Rules for this whole series: use only Python's standard library; install and download nothing; make no network calls; reproduce file content exactly, character for character, without reformatting, fixing or improving anything; work only inside the install folder.

## Background (for context only)
I'm a regulatory medical writer. I score the outputs of our standard prompts in Scribbler (our AI add-in for Microsoft Word) using scoring templates. The tool, MW Eval Helper, removes the manual copy-paste: "Prep scorer" fills a scorer template with a prompt's output from the clipboard, and "Log score" parses the scorer's JSON from the clipboard into a CSV log, archiving the raw texts for traceability.

## How the series works
- Prompts 2 to 6 each contain one or more files, or one part of a large file, between lines of five tildes (`~~~~~`). The content is everything between those fence lines, excluding the fence lines themselves.
- **If you can create files** (agent mode): create each file at the stated path inside the install folder, saved as UTF-8. For a file split into parts, create it with part 1 and append each later part directly to the end, adding nothing between parts.
- **If you can't create files** (chat only): reply with each file's content in one code block, exactly as given, and tell me the path to save it to. For split files, tell me to paste each part at the end of the same file. Never summarise or shorten content.
- After each prompt, reply only with what you created (path, part number) or the content to save, then wait for the next prompt.
- Prompt 7 verifies everything with fingerprints, so any copying error will be caught.

## Now
1. Ask me for the install folder (suggest `%USERPROFILE%\MW_Eval_Helper`). If it exists and isn't empty, list its contents and ask before continuing.
2. If you can run commands, run `python --version` (3.8+ needed; if not found, try `py -3 --version` and use `py -3` for later commands). Report the result.
3. Then wait for prompt 2.
