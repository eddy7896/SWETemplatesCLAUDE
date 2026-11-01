# 10. Hub data model and update protocol

Everything the hub shows lives in a few small JSON documents. The page only renders them. Whether they sit in a shared database (reference: an artifact's `db`) or in files (`hub-template/data/*.json`), the shapes are identical, so the host can change without rewriting the page.

## 1. Documents
| Document | Writer | Reader | Purpose |
|---|---|---|---|
| `plan/tasks` | agent | page, dashboard | all tasks with status and percent |
| `reviews/<key>` | owner (via the page) | agent, page | one record per reviewed target |
| `screens` | agent | page | the screens catalogue (metadata + image references) |
| `flows` | agent | page | diagram sources and legend |
| `git` | agent | page | branch tree, ledger, rules, owner actions, read date |
| `glossary` | agent | page | terms |
| `datasets/tasks` (dashboard only) | agent | dashboard page | pointer to the tasks table file and its update note |

## 2. `plan/tasks`
```json
{
  "updatedAt": "2026-10-31T12:00:00Z",
  "items": [
    {
      "id": "P2-3a",
      "name": "Invoices: reverse instead of edit (server)",
      "phase": 2,
      "phaseName": "Core and finance",
      "milestone": "",
      "order": 41.1,
      "status": "todo | doing | done | blocked | waived",
      "pct": 0,
      "pr": "#118",
      "adr": "ADR-0007",
      "owner": ""
    }
  ]
}
```
Rules:
- `id` is stable and never reused; sub-tasks use a letter suffix; umbrella tasks keep their ID and a percent **derived** from their children (average of non-waived children).
- `order` is a number so a new task can be inserted between two others (41.1 between 41 and 42); sort by it.
- `pct` is 0 to 100 and equals 100 only when the task is merged and its checks pass. `waived` tasks are excluded from every average.
- The page computes: overall percent = average of `pct` over non-waived **leaf** tasks (or all tasks if you choose, but be consistent and say so on the page); per phase the same.
- Phases that are only proposed (waiting for an ADR) are included with status `todo`/`doing` and the page notes "proposal".
- Writes carry the document's `version`; on conflict, re-read and redo.

## 3. `reviews/<key>`
`key` is the target ID made safe (`P2-3a`, `screen_web-admin_invoices`, `adr_0007`).
```json
{
  "target": "P2-3a",
  "kind": "task | phase | milestone | screen | component | adr | other",
  "label": "Invoices reverse, not edit",
  "decision": "approved | changes",
  "comment": "required when decision is changes",
  "at": "2026-10-31T12:34:56Z",
  "by": "opaque user id (optional)",
  "resolved": null
}
```
- The page writes `decision`, `comment`, `at`. The agent later writes `resolved`: `{ "at": "...", "by": "agent", "note": "what was changed", "pr": "#123" }`.
- The Owner review tab lists every record whose `decision` is `changes` and not resolved, then everything awaiting review (items with no record and status `done`/`partial`).
- Only users with write access can save a review; read-only viewers see the buttons disabled with a plain sentence.

## 4. `screens`
```json
{
  "updatedAt": "2026-10-31",
  "items": [
    {
      "id": "web-admin-invoices",
      "app": "web-admin",
      "title": "Invoices",
      "note": "What the screen is for, in one sentence",
      "status": "planned | built | open-pr",
      "pr": "#118",
      "task": "P2-3b",
      "images": { "en": "assets/web-admin-invoices.en.png", "ur": "assets/web-admin-invoices.ur.png" },
      "device": "desktop | phone",
      "states": ["empty", "loading", "error", "data"],
      "roles": ["BURSAR", "PRINCIPAL"]
    }
  ]
}
```
Rules: one entry per screen **state group** worth reviewing (default data plus empty and error where they differ); images are real renders with sample data, not mock-ups; a language with no image falls back to the default and the page says so; `status` flips to `built` only when the PR is merged.

## 5. `flows`
```json
{
  "updatedAt": "2026-10-31",
  "diagrams": [
    { "id": "connect", "title": "How the pieces connect", "note": "Built parts are green, planned parts are grey.", "kind": "mermaid", "source": "flowchart LR\n  ..." }
  ],
  "legend": [{ "label": "Built and tested", "class": "built" }, { "label": "Planned or not tested against the real service", "class": "plan" }]
}
```
Recommended diagrams: (1) how the pieces connect, (2) what happens to one request (every guard and what it answers), (3) how a customer/organisation and its people get onto the platform, (4) how work reaches production (the branch model), then one per hard flow (sync, payments, an analytics pipeline).

## 6. `git`
```json
{
  "readAt": "2026-10-31",
  "status": [{ "tone": "bad | ok", "text": "Production is 12 commits behind staging." }],
  "tree": [{ "name": "main", "kind": "prod", "note": "production", "children": [] }],
  "branches": [{ "name": "app/api", "owner": "api", "purpose": "...", "ahead": 3, "behind": 0 }],
  "ownership": [{ "area": "apps/api", "branch": "app/api" }],
  "ledger": [{ "pr": "#118", "title": "P2-3: ...", "state": "merged | open", "branch": "cross/p2-3", "task": "P2-3" }],
  "rules": ["Never push to main or staging directly."],
  "ownerActions": ["Switch on branch protection for main and staging."]
}
```
Facts come from `git` and the host CLI at refresh time; `readAt` is shown on the page.

## 7. `glossary`
`[{ "term": "Tenant", "definition": "...", "group": "Platform" }]`, generated from `glossary.md`; the page offers a search box.

## 8. Dashboard (optional second page)
A dashboard reads one table (`tasks.json` rows: `phase, phase_name, id, task, status, pct`) generated from the plan data. Update order after each task: (1) plan data, (2) regenerate the rows file, (3) upload it as an asset, (4) write the dataset pointer (`source.url`, `updated.at`, `updated.by`) with the document version. Never edit the dashboard's rows by hand.

## 9. Versioned-write protocol
1. Read the document and note its `version`.
2. Change only the fields you own.
3. Write with `if_version = version`. If refused, read again and re-apply your change; never overwrite blindly.
4. Prefer one batch write when several documents change together (plan data and dataset pointer).
5. After writing, read back one value to confirm.

## 10. Seeding from the plan
A seed script (`tools/hub/seed-plan.mjs`) reads the task tables in `implementation_plan.md` and `phaseN_plan.md` (columns: ID, Task, Size, Acceptance, Status) and writes `plan.json`. It must (a) keep IDs, (b) fail if two rows share an ID, (c) map statuses, (d) leave `pct` 0 for `todo`. Run it once at Gate 3b and again only to add new tasks, never to reset progress.
