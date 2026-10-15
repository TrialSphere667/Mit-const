# STAGE 3 — JSON validation, independent score checks and reporting

You are extending a company-approved, offline Python/Windows toolkit. It captures generation as `generated/RUN_ID.txt`, scorer response as `inbox/RUN_ID.txt`, and machine-controlled metadata as `manifests/RUN_ID.json`. Scoring happens manually in a separate fresh add-in session with the same source files. All project dependencies must remain in the Python standard library; only synthetic data for tests. Inspect actual existing files and adapt with minimal refactoring.

**Implement now:** `confirm_run.py`, `collect_results.py`, unit tests and an explicit scorer schema. Do not make external requests or use cloud/LLM evaluators.

- `confirm_run.py RUN_ID --reviewer SYNTHETIC_REVIEWER`: ask **separately** whether a genuinely fresh scorer session was used and the same exact source versions were selected. Save reviewer/time and `manual_attestation`, not machine proof. If any answer is no/unknown, mark provisional/noneligible rather than silently accepting. Support safe noninteractive flags for tests *only*, with names explicit enough not to be used accidentally.
- Collector requires run manifest, generation, prepared prompt, scorer response, correct scorer template and rubric fingerprint, and positive attestations. Validate record hashes where possible and fail closed on inconsistencies. Do not rewrite old raw output.
- Strict scorer JSON contract (exact criterion names for each scorer version):

```json
{
  "schema_version": "1",
  "weighted_score": 73.0,
  "criteria": {
    "faithfulness": {"score": 4, "max_score": 5, "issues": ["synthetic issue"]},
    "completeness": {"score": 3, "max_score": 5, "issues": ["synthetic omission"]},
    "accuracy": {"score": 4, "max_score": 5, "issues": []}
  },
  "issues": [{"severity": "major", "description": "synthetic example"}]
}
```

Weights `0.40`, `0.35`, `0.25` and max scores 5 imply `73.0` by `100*Σ(weight * score/max)/Σ(weight)`. Do not trust scorer-reported score: independently calculate and check within an explicitly documented small rounding tolerance, otherwise reject/flag. Reject missing/extra rubric criteria, invalid JSON, booleans as numeric values, wrong max scores, nonfinite/out-of-range values, invalid severities, and tampered rubric/source fingerprints. Preserve exact raw JSON even when rejected.

Generate normalized, Excel-friendly, stable-order, deterministic reports:

1. `reports/runs.csv`: one row per structurally valid attested run, metadata, score, hashes/status, human-review flags (no clinical output text).
2. `reports/criterion_scores.csv`: one row per criterion per accepted structural record.
3. `reports/errors.csv`: run ID, safe error code, safe diagnostic (no raw clinical content or issue narratives).

Rebuild reports from immutable per-run records to avoid duplicate appended rows. Atomic `os.replace` of temporary CSVs, normalized UTF-8 encoding, deterministic sorting and CSV formula-injection neutralization for exported text fields beginning with `=`, `+`, `-`, `@`. Recommend one report writer for SharePoint-synced directories.

Qualify reporting language: a structurally valid score is **not** medically verified. Any `critical` error and relevant major error require human review before a quality-pass determination. Do not hide severe errors within the weighted score.

Run synthetic tests if file tools exist; otherwise output complete sources and tests and say *not executed*. Describe failure handling and reconciliation procedure. Never invent test results.
