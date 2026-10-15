# OPTIONAL — Investigate future one-click evaluation support

You are a technical product analyst preparing *non-invasive proposals* for a regulated clinical writing tool. Do not implement UI automation without a distinct approval. The current tool is an internal Word add-in whose responses appear in a task pane and have a Copy button; no model/router API is available to end users. Independent scoring requires a **fresh add-in session** and selection of identical source document versions.

**Deliver two separately prioritized tracks.**

## Track 1: Attended Power Automate for desktop feasibility (synthetic only)

- First confirm corporate availability and authorization. Do not install apps or browser extensions.
- Describe a manual experiment using the UI element picker against the task pane's Copy button, fresh-session button and source file picker; identify whether accessibility/UIA selectors are stable.
- Use element selectors rather than mouse-coordinate guessing; if selectors are not stable or expose controls poorly, stop instead of fragile automation.
- Design explicit user confirmations for any source-selection ambiguity. A UI click is *not* proof of actual backend context isolation.
- Prevent clipboard history, hidden background watchers, network data export and unapproved credential storage. Return a go/no-go feasibility table with risks, not fabricated test results.

## Track 2: Developer feature request

Write a concise internal ticket for a native **Evaluate Linked Prompt** feature:

1. After generation, start a genuinely new scorer backend session (new context/session ID, no carry-over from generation).
2. Reattach the identical source document *version IDs*, not merely display names.
3. Run a selected version-controlled scorer template on the generated response and same source set.
4. Export both immutable responses, timestamps, generation/scoring model identifiers/configurations, source version IDs, session IDs, prompt/scorer hashes, JSON validation status and weighted score.
5. Persist through an IT-approved audited storage layer, optionally SharePoint export or approved internal endpoint; no external SaaS.
6. Introduce manual review status, critical-error alerts, benchmark summary/report integration, and permissions/retention controls.
7. Add testable acceptance criteria proving the scorer actually had a new backend context and identical source-set versions.

Priority order: **native export + provenance** first, source-set cloning and scorer session automation second, dashboards third. Provide a quick MVP scope and a longer-term roadmap that avoids overengineering the writer-operated interim solution.
