# 09. The Project Hub: one place for progress, screens, architecture and decisions

Purpose: the owner (who may not open the repository) reads **everything about the project in one page** and can **review and approve** work there. It is both a live progress report and living system documentation. In the OruSchools project it showed, per tab: Progress, Owner review, UI assets, Screens, Demo school, System flow, Git and glossary.

Build it at **Gate 3b** (right after the documents are reviewed) as a small empty shell with the plan loaded, then keep it current as part of finishing every task. A hub that is updated "when asked" is wrong within a week.

## 1. What the hub must show (tabs)
| Tab | Shows | Source of truth | Updated when |
|---|---|---|---|
| **Progress** | big percent, phases, milestones, tasks with status and percent; link to the PR | the plan data (`plan` document, section 3 of `10`) | a task changes status, a PR is opened or merged |
| **Owner review** | every item awaiting the owner, with Approve / Request changes and a comment; the decisions log | review records (`reviews/*`) | the owner clicks; the agent reads and acts |
| **Screens** | every screen of every app, as built, per language and direction, with its status (planned / built / open PR) and per-screen review | the screens data + captured images (`11`) | after each UI task |
| **UI assets** | the design-system components exactly as rendered (only if the project has a UI library) | specimen data generated from the component library | the library changes |
| **System flow** | architecture and flow diagrams, built parts vs planned parts | diagrams in the page, mirrored from `system_architecture.md` | an ADR changes the architecture; a part is built |
| **Git and glossary** | branch tree, branch table, module ownership, PR ledger, merge rules, owner actions, glossary | read from git and the host, not from memory | branches/PRs/rules change |
| **Demo / environments** | how to run a demo with sample accounts (local only), staging URL, health | environment document | the demo or environments change |
| **Decisions (optional)** | ADR list with status, open owner questions | `documentation/decisions/` | an ADR changes |

A dashboard (a second page with charts) may read the same plan data; keep one source of truth.

## 2. Systems the hub needs
| System | What it does | Notes |
|---|---|---|
| **A page host** | serves one HTML page to the owner with sign-in or private link | In the reference project: a Claude **Artifact** (private by default, shared by link). Alternatives: GitHub Pages behind access control, an internal static site, a wiki page with embedded HTML |
| **A small shared data store the page can read and write** | holds plan data and review records; the agent writes plan data, the owner writes reviews | Reference: the Artifact's `db` capability (documents with versions). Alternative: JSON files in the repo (agent writes via PR) plus a review form that posts to a tiny API or an issue tracker |
| **An asset store** | screen images, specimen images | Reference: the artifact asset store; alternative: files in the repo or object storage |
| **A capture pipeline** | produces screen images from the real code with sample data (`11`) | headless browser for web, widget/golden tests for mobile |
| **A refresh routine** | after each task, rewrites plan data, dashboard data, git facts, screens | a documented checklist in `live_documents.md`; optionally a script that reads `gh`/`git` |
| **Access control** | who may view (owner, team) and who may write reviews | least privilege; never put secrets, tokens or real personal data on the hub |

## 3. Rules for the hub
1. **Every fact on the hub has a source the agent actually read** (git, `gh`, the plan, a test result). If it was not verified, the page says "unverified". Dates are absolute ("read from the repository on 31 Oct 2026").
2. **Never put credentials, production data or real personal data on the hub.** Demo accounts are allowed only for local emulators and are labelled "works only on a local demo".
3. **The plan data is the single source for progress.** Percent comes from tasks (equal weight, waived tasks excluded). Do not type percentages anywhere else.
4. **Reviews are data, not chat.** The agent reads `reviews/*`, acts on "changes requested", and records the resolution.
5. **Writes are versioned.** Read the document, change it, write it back with the version you read; if someone else changed it, re-read and redo the change.
6. **Light and clear.** One page, tabs, no build step required for the page itself; works on a phone; respects the project's accessibility floor (contrast, text size, focus, keyboard, right-to-left if the product has it).
7. **The hub is not the documentation.** It links to the documents and ADRs; it does not duplicate them. The architecture diagrams are the one exception, kept in sync with `system_architecture.md`.
8. **Stale is worse than absent.** If a tab cannot be kept current, remove it or label its date prominently.

## 4. Build steps (Gate 3b)
1. Copy `hub-template/` (this folder) into the host or a `hub/` folder; fill `data/*.json` from the plan. The template is a single HTML page with a store adapter that reads JSON files (works anywhere) or the artifact `db` (when present).
2. Seed the **plan data** from `implementation_plan.md` (script: parse the task tables into `plan.json`; keep the IDs identical).
3. Draw the **System flow** diagrams from `system_architecture.md` (Mermaid source in `flows.json`); built parts solid, planned parts dashed.
4. Fill the **Git** tab from `git branch -a`, `gh pr list`, and the branch-policy config; record the date read.
5. Add the **Glossary** from `glossary.md`.
6. Publish; open it as the owner would (a phone width, a keyboard-only pass); fix what is unclear.
7. Add the hub rows to `live_documents.md` and the "update the hub" line to the Definition of Done in `CLAUDE.md`.
8. Report the link and who can see it.

## 5. The refresh routine (after every task, PR or state change)
1. Plan data: set `status` and `pct` of the task (done = 100) with the PR number; keep umbrella percents derived.
2. Dashboard data (if any): rebuild the task rows from the plan data and write the dataset pointer.
3. Git tab: re-read branches and PRs from git/`gh`; update the PR ledger, owner actions, and any rule changes.
4. Screens: after a UI task, capture the new screens (`11`) and mark merged ones `built`.
5. System flow: if an ADR changed the architecture or a planned part is now built, update the diagrams and the legend.
6. Owner review: read new reviews, act, record the resolution.
7. Say in the final message of the task **what changed on the hub**.

## 6. Failure modes seen in practice (and the fix)
| Problem | Fix |
|---|---|
| Branch/commit facts went stale for days | Git tab shows the date read; refresh routine step 3 is in the Definition of Done |
| Diagram blank when its tab is hidden | Render diagrams when the tab is first shown, not at load |
| Numbers on the page disagree with the plan | Compute every percent from plan data in the page; never hard-code |
| Huge page (many images inline) | Store images as assets and reference them; keep the page under the host's size limit |
| Two writers overwrite each other | Versioned writes; re-read on conflict |
| Review comments lost | Reviews are documents with a stable key per target; the agent resolves and records |
| The page looked finished but a tab was a copy of old facts | Rule 1 and 8: dated sources, remove what cannot be kept current |
