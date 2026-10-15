You are assisting me inside Scribbler. I want your help designing a way to automate how I evaluate the outputs of our team's standard prompts. Below is the problem. First answer the questions about Scribbler in Part 5, then propose solutions as described in Part 6.

PART 1. BACKGROUND
Our medical writing team uses a library of standard prompts in Scribbler to draft and check regulated clinical documents (e.g., CSR sections, protocol amendments). To benchmark these prompts and compare versions over time, I evaluate their outputs with scoring templates I created, one per prompt type. Each scoring template instructs the LLM to judge a prompt's output against the same source documents the prompt used (e.g., faithfulness, completeness, accuracy) and to return JSON containing per-criterion scores, a weighted overall score, and a list of issues.

My current understanding of Scribbler: I select context files, enter or load a prompt, and run it. Output appears in a separate output pane, similar to a chat, and can be copied. As far as I know, outputs cannot be saved to files and there is no direct API access to the models. Correct me if any of this is wrong.

PART 2. CURRENT WORKFLOW (one evaluation run of a prompt called XYZ)
1. Run prompt XYZ in Scribbler with a set of context files selected.
2. Copy XYZ's output from the output pane.
3. Open the matching scoring template and paste XYZ's output into it at the designated spot.
4. Start a fresh Scribbler session, select the same context files, paste the completed scoring prompt, and run it.
5. Copy the JSON result from the output pane.
6. Manually transcribe the JSON (weighted score, criterion scores, issues) into a CSV log that tracks results for XYZ across runs and versions.
This is repeated for many runs and many prompts.

PART 3. THE PROBLEM
- Steps 3 and 6 are mechanical but done by hand every run: slow, tedious and error-prone (transcription mistakes, wrong template, wrong prompt version logged).
- Because each run is costly, fewer runs get done, limiting how reliably prompts can be benchmarked and versions compared.
- The CSV log has no reliable link back to the exact output scored, the scoring template version, or the source files used, so past scores are hard to audit or reproduce.
- Nothing systematically catches malformed or incomplete scorer output, duplicates, or changes to the scorer's JSON structure over time.

PART 4. REQUIREMENTS
Must:
- Scorer independence: scoring runs in a separate, fresh session from the one that generated the output, with the same source documents available. Never merge generation and scoring into one session.
- Traceability: every logged score is attributable to the prompt and version, the scoring template and version, the run date, the source document set, and the exact output text judged.
- Data stays internal: no external services; content is confidential clinical and regulatory material.
- Maintainable by a medical writer, not a dedicated software engineer.
- Robust to imperfect LLM output: malformed JSON, extra text around the JSON, missing fields, and scoring templates whose JSON structure changes over time.
Should:
- Reduce the manual steps per run to the minimum Scribbler allows.
- Make results easy to analyse across runs and versions (e.g., in Excel).
- Prevent duplicate or mis-attributed log entries.
- Keep the raw outputs of both steps so any score can be re-checked.
- Avoid installing additional software where possible (locked-down corporate environment).

PART 5. QUESTIONS ABOUT SCRIBBLER (answer these first)
For each question, answer from what you actually know about Scribbler. Label every answer as one of: KNOWN (you are confident this is how Scribbler works), UNSURE (your best understanding, needs checking), or UNKNOWN (you don't know). Do not guess features into existence; I will verify UNSURE items with the Scribbler team.
1. Can Scribbler save or export outputs (to a file, to the Word document, to SharePoint or OneDrive)?
2. Can prompts be chained, so one prompt's output is passed automatically into another prompt?
3. Can a saved prompt or template contain a variable or placeholder that Scribbler fills in, for example with a previous output or pasted text?
4. Is there a batch mode, or a way to run one prompt over several inputs?
5. How do sessions work? Does starting a new session fully clear prior context, and can a new session reuse the previous session's context file selection?
6. Can Scribbler be instructed or configured to always return output in a fixed format such as JSON, and does it ever add text around the JSON?
7. Does Scribbler record which model, prompt and context files were used for each run, and can I see or export that information?
8. Can Scribbler insert output directly into the Word document, where a Word macro or other tool could then pick it up?
9. Is there a shared prompt library, and does it support versioning of prompts and templates?
10. Can I choose between different underlying models, so the same scoring template could be run by more than one model for comparison?
11. Is there any official way for other tools (Word macros, Excel, Power Automate, scripts) to interact with Scribbler?
12. What is the maximum input length, and could a long prompt output plus scoring template plus context files exceed it?

PART 6. PROPOSE SOLUTIONS
Using your answers from Part 5:
1. Propose 2-4 solution options, from the simplest I could use today to the most automated. For each option, state:
   - Which steps of the current workflow it removes or shortens
   - What it depends on (Scribbler features, Office features, scripts), marking any dependency that rests on an UNSURE or UNKNOWN answer
   - How it meets each Must requirement in Part 4, or which one it compromises
   - Effort to set up and maintain, and the main risks
2. Recommend one option to start with and explain why.
3. Separately, list any Scribbler features that, if the Scribbler team added them, would remove the remaining manual steps. Write these as short feature requests.
Do not propose anything that sends content outside the corporate environment or that runs scoring in the same session as generation.
