# MW Eval Helper deployment - prompt 2 of 7

Rules for this whole series: use only Python's standard library; install and download nothing; make no network calls; reproduce file content exactly, character for character, without reformatting, fixing or improving anything; work only inside the install folder.

Content is everything between the `~~~~~` fence lines, excluding the fences. Each file ends with a single newline after its last line.

### File: `mw_eval_helper.py` - part 1 of 3
Create this file with this content. Parts 2-3 will follow in the next prompts.

~~~~~python
"""
MW Eval Helper - clipboard bridge between Scribbler runs and the score log.

Standard library only. Nothing is sent over the network.

Usage
  python mw_eval_helper.py              window with two buttons (needs tkinter)
  python mw_eval_helper.py prep  ...    terminal mode: build scorer prompt
  python mw_eval_helper.py log   ...    terminal mode: log scorer JSON
  python mw_eval_helper.py status       show the pending run, if any
  python mw_eval_helper.py --help       all options

Workflow
  1. Run prompt XYZ in Scribbler. Copy its output from the output pane.
  2. Prep: the scorer template is filled with XYZ's output and put on the
     clipboard.
  3. Fresh Scribbler session, select the SAME context files, paste, run.
  4. Copy the scorer's JSON output. Log: a row is appended to the CSV log;
     raw XYZ output and scorer JSON are archived under the same run ID.

Folder layout (created on first run, next to this script)
  scorer_templates/   scorer templates (.txt or .md), each containing the
                      placeholder <<<XYZ_OUTPUT>>> exactly once.
                      Optional first line:  SCORER_VERSION: 1.0
  archive/            raw XYZ outputs and scorer JSON per run (audit trail)
  score_log.csv       the benchmark log (opens directly in Excel)
"""

__version__ = "1.0.0"

import argparse
import csv
import datetime as dt
import hashlib
import json
import re
import subprocess
import sys
from pathlib import Path

BASE = Path(__file__).resolve().parent
TEMPLATES_DIR = BASE / "scorer_templates"
ARCHIVE_DIR = BASE / "archive"
LOG_CSV = BASE / "score_log.csv"
SETTINGS = BASE / ".mw_eval_settings.json"
PENDING = BASE / ".mw_eval_pending.json"
PLACEHOLDER = "<<<XYZ_OUTPUT>>>"

META_COLS = [
    "run_id", "logged_at", "prompt_name", "prompt_version",
    "scorer_template", "scorer_version", "context_files",
    "xyz_output_sha256", "scorer_json_sha256", "notes",
]


class HelperError(Exception):
    """A problem the user can fix; message is shown as-is."""


# ------------------------------------------------------------ core helpers

def sha256(text):
    return hashlib.sha256(text.encode("utf-8")).hexdigest()


def ensure_dirs():
    TEMPLATES_DIR.mkdir(exist_ok=True)
    ARCHIVE_DIR.mkdir(exist_ok=True)


def extract_json(text):
    """Pull the JSON object out of pasted output, tolerating code fences and
    stray text before or after it."""
    t = (text or "").strip()
    fence = re.search(r"```(?:json)?\s*(.*?)```", t, re.DOTALL | re.IGNORECASE)
    if fence:
        t = fence.group(1).strip()
    start, end = t.find("{"), t.rfind("}")
    if start == -1 or end <= start:
        raise HelperError("No JSON object found. Copy the scorer's JSON output first.")
    try:
        obj = json.loads(t[start:end + 1])
    except json.JSONDecodeError as e:
        raise HelperError(f"JSON is malformed: {e.msg} (line {e.lineno}, col {e.colno}). "
                          "Nothing was logged.")
    if not isinstance(obj, dict):
        raise HelperError("Top-level JSON must be an object {...}.")
    return obj


def flatten(obj, prefix=""):
    """Nested objects become dotted columns. Lists of plain values are
    joined with '; '. Lists of objects (e.g. issues) are stored as JSON text
    plus a .count column."""
    out = {}
    if isinstance(obj, dict):
        for k, v in obj.items():
            out.update(flatten(v, f"{prefix}{k}."))
    elif isinstance(obj, list):
        key = prefix.rstrip(".")
        if all(not isinstance(i, (dict, list)) for i in obj):
            out[key] = "; ".join(str(i) for i in obj)
        else:
            out[key] = json.dumps(obj, ensure_ascii=False)
        out[f"{key}.count"] = len(obj)
    else:
        out[prefix.rstrip(".")] = obj
    return out


def read_log():
    if not LOG_CSV.exists():
        return [], []
    with LOG_CSV.open(newline="", encoding="utf-8-sig") as f:
        r = csv.DictReader(f)
        return list(r.fieldnames or []), list(r)


def append_row(row):
    """Append a row. If the scorer JSON gained new fields, rewrite the file
    with the extended header so no earlier data is lost. Returns True when
    the header was extended."""
    header, rows = read_log()
    if not header:
        header = META_COLS + [c for c in row if c not in META_COLS]
        with LOG_CSV.open("w", newline="", encoding="utf-8-sig") as f:
            w = csv.DictWriter(f, fieldnames=header)
            w.writeheader()
            w.writerow(row)
        return False
    new_cols = [c for c in row if c not in header]
    if new_cols:
        header = header + new_cols
        rows.append(row)
        tmp = LOG_CSV.with_suffix(".tmp")
        with tmp.open("w", newline="", encoding="utf-8-sig") as f:
            w = csv.DictWriter(f, fieldnames=header)
            w.writeheader()
            w.writerows(rows)
        tmp.replace(LOG_CSV)
        return True
    with LOG_CSV.open("a", newline="", encoding="utf-8-sig") as f:
        csv.DictWriter(f, fieldnames=header).writerow(row)
    return False


def already_logged(json_hash):
    _, rows = read_log()
    return any(r.get("scorer_json_sha256") == json_hash for r in rows)


def list_templates():
    if not TEMPLATES_DIR.exists():
        return []
    return sorted(p.name for p in TEMPLATES_DIR.iterdir()
                  if p.is_file() and p.suffix.lower() in (".txt", ".md"))


def load_template(name):
    path = TEMPLATES_DIR / name
    if not path.is_file():
        raise HelperError(f"Template not found: {name}. Available: {', '.join(list_templates()) or 'none'}")
    text = path.read_text(encoding="utf-8")
    m = re.match(r"\s*SCORER_VERSION:\s*(\S+)[ \t]*\r?\n", text)
    version = m.group(1) if m else ""
    body = text[m.end():] if m else text
    n = body.count(PLACEHOLDER)
    if n != 1:
        raise HelperError(f"Template {name} must contain {PLACEHOLDER} exactly once (found {n}).")
    return body, version


def new_run_id(prompt_name):
    safe = re.sub(r"[^A-Za-z0-9_-]+", "-", prompt_name or "run").strip("-") or "run"
    return f"{safe}_{dt.datetime.now():%Y%m%d-%H%M%S}"


def read_pending():
    try:
        return json.loads(PENDING.read_text(encoding="utf-8"))
    except Exception:
        return None


def clear_pending():
    try:
        PENDING.unlink()
    except FileNotFoundError:
        pass


# ------------------------------------------------------- the two actions

def prep_run(xyz_text, template, prompt_name, prompt_version="", context_files=""):
    """Build the scorer prompt and record the pending run. Returns
    (scorer_prompt, run_id)."""
    ensure_dirs()
    xyz = (xyz_text or "").strip()
    if not prompt_name.strip():
        raise HelperError("Enter the prompt name (e.g. XYZ).")
    if not template:
        raise HelperError("Choose a scorer template.")
    if not xyz:
        raise HelperError("Clipboard is empty. Copy XYZ's output from Scribbler first.")
    if PLACEHOLDER in xyz:
        raise HelperError("The clipboard holds a scorer prompt, not XYZ's output.")
    body, version = load_template(template)
    run_id = new_run_id(prompt_name)
    (ARCHIVE_DIR / f"{run_id}_xyz_output.txt").write_text(xyz, encoding="utf-8")
    PENDING.write_text(json.dumps({
        "run_id": run_id, "xyz_hash": sha256(xyz), "template": template,
        "scorer_version": version, "prompt_name": prompt_name,
        "prompt_version": prompt_version, "context_files": context_files,
        "prepped_at": dt.datetime.now().isoformat(timespec="seconds"),
    }, indent=2), encoding="utf-8")
    return body.replace(PLACEHOLDER, xyz), run_id


def log_run(scorer_text, notes="", allow_duplicate=False, allow_unlinked=False,
            fallback=None):
    """Parse scorer output and append it to the log. `fallback` supplies
    prompt metadata when there is no pending run. Returns a result dict."""
    ensure_dirs()
    data = extract_json(scorer_text)
    pending = read_pending()
    if pending is None:
        if not allow_unlinked:
            raise HelperError("NO_PENDING")
        fb = fallback or {}
        pending = {"run_id": new_run_id(fb.get("prompt_name", "")), "xyz_hash": "",
                   "template": fb.get("template", ""), "scorer_version": "",
                   "prompt_name": fb.get("prompt_name", ""),
                   "prompt_version": fb.get("prompt_version", ""),
                   "context_files": fb.get("context_files", "")}
    jhash = sha256(json.dumps(data, sort_keys=True, ensure_ascii=False))
    if not allow_duplicate and already_logged(jhash):
        raise HelperError("DUPLICATE")
    row = {
        "run_id": pending["run_id"],
        "logged_at": dt.datetime.now().isoformat(timespec="seconds"),
        "prompt_name": pending.get("prompt_name", ""),
        "prompt_version": pending.get("prompt_version", ""),
        "scorer_template": pending.get("template", ""),
        "scorer_version": pending.get("scorer_version", ""),
~~~~~

Reply with what you created or appended (path and part), then wait for prompt 3.
