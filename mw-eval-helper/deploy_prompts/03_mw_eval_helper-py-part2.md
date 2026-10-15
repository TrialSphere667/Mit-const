# MW Eval Helper deployment - prompt 3 of 7

Rules for this whole series: use only Python's standard library; install and download nothing; make no network calls; reproduce file content exactly, character for character, without reformatting, fixing or improving anything; work only inside the install folder.

Content is everything between the `~~~~~` fence lines, excluding the fences. Each file ends with a single newline after its last line.

### File: `mw_eval_helper.py` - part 2 of 3
Append this content directly to the end of the existing `mw_eval_helper.py`. Add no extra blank lines or text.

~~~~~python
        "context_files": pending.get("context_files", ""),
        "xyz_output_sha256": pending.get("xyz_hash", ""),
        "scorer_json_sha256": jhash,
        "notes": notes,
    }
    for k, v in flatten(data).items():
        row[k if k not in META_COLS else f"scorer.{k}"] = v
    try:
        widened = append_row(row)
    except PermissionError:
        raise HelperError("Can't write score_log.csv. Close it in Excel and try again. "
                          "Nothing was logged.")
    (ARCHIVE_DIR / f"{pending['run_id']}_scorer.json").write_text(
        json.dumps(data, indent=2, ensure_ascii=False), encoding="utf-8")
    clear_pending()
    score = next((row[k] for k in row if "weighted" in k.lower()), "?")
    return {"run_id": pending["run_id"], "score": score, "widened": widened}


# -------------------------------------------- clipboard for terminal mode

def cli_clip_get():
    if sys.platform == "win32":
        r = subprocess.run(["powershell", "-NoProfile", "-Command",
                            "[Console]::OutputEncoding=[Text.Encoding]::UTF8; Get-Clipboard -Raw"],
                           capture_output=True)
        return r.stdout.decode("utf-8", errors="replace")
    if sys.platform == "darwin":
        return subprocess.run(["pbpaste"], capture_output=True, text=True).stdout
    try:
        import tkinter as tk
        root = tk.Tk(); root.withdraw()
        try:
            return root.clipboard_get()
        finally:
            root.destroy()
    except Exception:
        raise HelperError("No clipboard access. Use --in FILE instead.")


def cli_clip_set(text):
    if sys.platform == "win32":
        tmp = BASE / ".mw_eval_clip.tmp"
        tmp.write_text(text, encoding="utf-8")
        try:
            subprocess.run(["powershell", "-NoProfile", "-Command",
                            "Get-Content -Raw -Encoding UTF8 -LiteralPath '"
                            + str(tmp).replace("'", "''") + "' | Set-Clipboard"],
                           check=True)
        finally:
            tmp.unlink(missing_ok=True)
    elif sys.platform == "darwin":
        subprocess.run(["pbcopy"], input=text, text=True, check=True)
    else:
        raise HelperError("No clipboard access. Use --out FILE instead.")


# ------------------------------------------------------------ terminal UI

def cli(argv):
    p = argparse.ArgumentParser(prog="mw_eval_helper",
                                description="Clipboard bridge for Scribbler prompt scoring.")
    sub = p.add_subparsers(dest="cmd")
    pp = sub.add_parser("prep", help="Fill the scorer template with XYZ output from the clipboard")
    pp.add_argument("--template", "-t", required=True, help="File name in scorer_templates/")
    pp.add_argument("--prompt-name", "-p", required=True)
    pp.add_argument("--prompt-version", "-v", default="")
    pp.add_argument("--context", "-c", default="", help="Context files used (free text)")
    pp.add_argument("--in", dest="infile", help="Read XYZ output from a file instead of the clipboard")
    pp.add_argument("--out", dest="outfile", help="Write scorer prompt to a file instead of the clipboard")
    pl = sub.add_parser("log", help="Log scorer JSON from the clipboard")
    pl.add_argument("--notes", "-n", default="")
    pl.add_argument("--in", dest="infile", help="Read scorer JSON from a file instead of the clipboard")
    pl.add_argument("--allow-duplicate", action="store_true")
    pl.add_argument("--allow-unlinked", action="store_true",
                    help="Log even without a prep step (XYZ output not linked)")
    sub.add_parser("status", help="Show the pending run")
    sub.add_parser("templates", help="List scorer templates")
    a = p.parse_args(argv)

    try:
        if a.cmd == "prep":
            src = Path(a.infile).read_text(encoding="utf-8") if a.infile else cli_clip_get()
            prompt, run_id = prep_run(src, a.template, a.prompt_name, a.prompt_version, a.context)
            if a.outfile:
                Path(a.outfile).write_text(prompt, encoding="utf-8")
                print(f"[{run_id}] Scorer prompt written to {a.outfile}.")
            else:
                cli_clip_set(prompt)
                print(f"[{run_id}] Scorer prompt copied. Paste it into a FRESH Scribbler session "
                      "with the same context files.")
        elif a.cmd == "log":
            src = Path(a.infile).read_text(encoding="utf-8") if a.infile else cli_clip_get()
            try:
                r = log_run(src, a.notes, a.allow_duplicate, a.allow_unlinked)
            except HelperError as e:
                if str(e) == "NO_PENDING":
                    raise HelperError("No pending prep step. Run prep first, or add --allow-unlinked.")
                if str(e) == "DUPLICATE":
                    raise HelperError("This exact JSON is already logged. Add --allow-duplicate to log it again.")
                raise
            msg = f"Logged {r['run_id']}: weighted score {r['score']}."
            if r["widened"]:
                msg += " New JSON fields found; log columns extended."
            print(msg)
        elif a.cmd == "status":
            pend = read_pending()
            print(json.dumps(pend, indent=2) if pend else "No pending run.")
        elif a.cmd == "templates":
            ensure_dirs()
            print("\n".join(list_templates()) or "No templates in scorer_templates/.")
        else:
            p.print_help()
    except HelperError as e:
        print(f"ERROR: {e}", file=sys.stderr)
        return 1
    return 0


# ------------------------------------------------------------- window UI

def gui():
    import tkinter as tk
    from tkinter import messagebox, ttk

    ensure_dirs()
    try:
        saved = json.loads(SETTINGS.read_text(encoding="utf-8"))
    except Exception:
        saved = {}

    root = tk.Tk()
    root.title(f"MW Eval Helper {__version__}")
    root.attributes("-topmost", True)
    root.resizable(False, False)

    v = {k: tk.StringVar(value=saved.get(k, "")) for k in
         ("prompt_name", "prompt_version", "context_files", "notes", "template")}

    frm = ttk.Frame(root, padding=10)
    frm.grid()
    fields = [("Scorer template", "template"), ("Prompt name", "prompt_name"),
              ("Prompt version", "prompt_version"), ("Context files", "context_files"),
              ("Notes", "notes")]
    for i, (label, key) in enumerate(fields):
        ttk.Label(frm, text=label).grid(row=i, column=0, sticky="w", pady=2)
        if key == "template":
            cb = ttk.Combobox(frm, textvariable=v[key], values=list_templates(),
                              width=34, state="readonly")
            cb.configure(postcommand=lambda: cb.configure(values=list_templates()))
            cb.grid(row=i, column=1, pady=2)
        else:
            ttk.Entry(frm, textvariable=v[key], width=37).grid(row=i, column=1, pady=2)

    pend = read_pending()
    status = tk.StringVar(value=(f"Pending run {pend['run_id']}: copy the scorer JSON, then Log score."
                                 if pend else "Copy XYZ's output, then Prep scorer."))
    ttk.Label(frm, textvariable=status, wraplength=340, foreground="#555").grid(
        row=len(fields) + 1, column=0, columnspan=2, sticky="w", pady=(8, 0))

    def save_settings():
        try:
            SETTINGS.write_text(json.dumps({k: x.get() for k, x in v.items()}), encoding="utf-8")
        except Exception:
            pass

    def clip_get():
        try:
            return root.clipboard_get()
        except tk.TclError:
            return ""

    def clip_set(text):
        root.clipboard_clear()
        root.clipboard_append(text)
        root.update()

    def prep():
        if read_pending() and not messagebox.askyesno(
                "MW Eval", "A prepped run hasn't been logged yet. Replace it with a new one?"):
            return
        try:
            prompt, run_id = prep_run(clip_get(), v["template"].get(), v["prompt_name"].get(),
                                      v["prompt_version"].get(), v["context_files"].get())
        except HelperError as e:
            return messagebox.showwarning("MW Eval", str(e))
        clip_set(prompt)
        save_settings()
        status.set(f"[{run_id}] Scorer prompt copied. Paste into a FRESH Scribbler session "
                   "with the same context files, then copy the JSON and click Log score.")

    def log(allow_dup=False, allow_unlinked=False):
        fallback = {k: v[k].get() for k in ("prompt_name", "prompt_version", "context_files", "template")}
        try:
            r = log_run(clip_get(), v["notes"].get(), allow_dup, allow_unlinked, fallback)
        except HelperError as e:
            if str(e) == "NO_PENDING":
                if messagebox.askyesno("MW Eval", "No Prep step recorded, so XYZ's output can't be "
                                       "linked to this score. Log anyway?"):
                    log(allow_dup, True)
                return
~~~~~

Reply with what you created or appended (path and part), then wait for prompt 4.
