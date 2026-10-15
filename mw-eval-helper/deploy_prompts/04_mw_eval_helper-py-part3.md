# MW Eval Helper deployment - prompt 4 of 7

Rules for this whole series: use only Python's standard library; install and download nothing; make no network calls; reproduce file content exactly, character for character, without reformatting, fixing or improving anything; work only inside the install folder.

Content is everything between the `~~~~~` fence lines, excluding the fences. Each file ends with a single newline after its last line.

### File: `mw_eval_helper.py` - part 3 of 3
Append this content directly to the end of the existing `mw_eval_helper.py`. Add no extra blank lines or text.

~~~~~python
            if str(e) == "DUPLICATE":
                if messagebox.askyesno("MW Eval", "This exact JSON is already in the log. Log a duplicate?"):
                    log(True, allow_unlinked)
                return
            return messagebox.showerror("MW Eval", str(e))
        msg = f"Logged {r['run_id']}: weighted score {r['score']}."
        if r["widened"]:
            msg += " New JSON fields found; log columns extended."
        status.set(msg)
        v["notes"].set("")
        save_settings()

    def open_folder():
        if sys.platform == "win32":
            import os
            os.startfile(BASE)
        elif sys.platform == "darwin":
            subprocess.run(["open", str(BASE)])

    btns = ttk.Frame(frm)
    btns.grid(row=len(fields), column=0, columnspan=2, pady=(8, 0))
    ttk.Button(btns, text="1. Prep scorer", command=prep).grid(row=0, column=0, padx=4)
    ttk.Button(btns, text="2. Log score", command=log).grid(row=0, column=1, padx=4)
    ttk.Button(btns, text="Open folder", command=open_folder).grid(row=0, column=2, padx=4)
    root.mainloop()


def main(argv=None):
    argv = sys.argv[1:] if argv is None else argv
    if argv:
        return cli(argv)
    try:
        import tkinter  # noqa: F401
    except ImportError:
        print("tkinter is not available, so the window can't open.\n"
              "Use terminal mode instead, e.g.:\n"
              "  python mw_eval_helper.py prep -t MY_SCORER.md -p XYZ -v 1.0\n"
              "  python mw_eval_helper.py log\n"
              "Run with --help for all options.")
        return 1
    gui()
    return 0


if __name__ == "__main__":
    sys.exit(main())
~~~~~

Reply with what you created or appended (path and part), then wait for prompt 5.
