# Project bootstrap prompt kit

A multi-document prompt that turns an idea into a documented, guarded, tested and branch-disciplined repository with a live Project Hub, the way the OruSchools platform was built. It is **generic**: nothing in these files assumes schools, Flutter, NestJS or any other stack. The agent learns what you are building by asking you, proposes an architecture, you approve it, and only then does it write documents, guardrails, checks, the hub and code.


---

## 1. What you get at the end
A repository where:
- the **decisions** are written down as ADRs and win any argument with the code or the chat;
- **guardrails** (security, privacy, money, offline, roles ...) exist as prose *and* as automated checks;
- **hallucination checks** give an AI agent ground truths, a golden question set with "trap" questions, and a grader;
- a **Project Hub** shows progress, owner review, screens, architecture, git and decisions on one page the owner can read and review, kept current as part of every task;
- a **drift check** fails the build when documents, configuration, schema and code disagree;
- **evals** and **tests** prove each rule and each phase gate;
- **git** follows a written branching and pull-request system that CI enforces;
- every change ends with a summary of **what was run and what was not verified**.

## 2. The files
```
project-bootstrap-prompt/
  README.md                                  this file
  00-master-system-prompt.md                 PASTE THIS FIRST: role, rules, phases, gates
  01-discovery-interview.md                  the questions, defaults, and the Project Brief template
  02-documentation-set.md                    every document to produce and its required sections
  03-adr-templates.md                        ADR template, lifecycle, 15 starter ADRs
  04-guardrails-and-hallucination-checks.md  guardrail derivation; golden set, traps, grader
  05-drift-evals-tests.md                    drift checks, eval catalogue, test pyramid, CI, cadence
  06-git-branching-and-pr.md                 branching model, branch-policy check, PR template, settings
  07-agent-operating-protocol.md             standing rules for every later session
  08-phase-plan-and-gates.md                 phases, task IDs, gate documents, the live record
  09-project-hub.md                          what the hub shows, systems it needs, build and refresh
  10-hub-data-model.md                       the hub's JSON documents and the versioned-write protocol
  11-hub-screens-and-architecture.md         capturing screens, honest diagrams, reading git facts
  hub-template/
    index.html                               a working single-file hub page
    data/*.json                              sample plan, reviews, screens, flows, git, glossary, info
```

| Phase in the master prompt | Gate | Files the agent reads |
|---|---|---|
| 0 Discovery interview | 1 Project Brief confirmed | `01` |
| 1 Architecture proposal | 2 Architecture approved | `03` (starter ADRs as drafts) |
| 2 Documentation set | 3 Documents reviewed | `02`, `03`, `08` |
| 2b Project Hub | 3b Hub live | `09`, `10`, `11`, `hub-template/` |
| 3 Guardrails, hallucination checks, drift, evals, tests | 4 Checks green | `04`, `05` |
| 4 Repository, branching, pull requests | 5 Workflow enforced | `06` |
| 5 onward: build in phases | each phase gate | `07`, `08` (and the `09` refresh routine) |

---

## 3. The best way to establish it (once, for the company)

### 3.1 Give the kit one home and one owner
- Keep **one canonical copy** in a company repository (for example `engineering-templates/project-bootstrap-prompt/`), not only on a laptop. The copy in `/Users/faheem/Projects/templates/` is a working copy; commit it to that repository when you are ready.
- Name an **owner** (a person, not a team) who accepts changes. Version it (the version line at the top; tag releases `kit-v0.1`, `kit-v0.2`). Every project records the kit version it started from in `documentation/project_brief.md`.
- Changes arrive by pull request with a reason from a real project ("the agent skipped X; we added a check"), never by editing a single project's copy.

### 3.2 Choose the company defaults once, so every project starts from them
Edit the kit (not each project) for things that are the same everywhere:
1. **Stack standards** you want by default (languages, cloud, identity provider). Add a short `12-company-defaults.md` listing them and the *banned* alternatives, and point `00` at it. Leave the kit stack-neutral if teams really differ.
2. **Regulatory guardrails** your sector always needs (data protection, payments, health, children's data). Put the categories in `04`, marked "unverified until counsel confirms", with the name of your legal contact.
3. **Branch and release rules** (`06`): branch names, who approves production, required CI checks, the host-settings checklist.
4. **Where the hub lives** (`09`): which page host, who can view, who can write reviews. Fill this in once so no team invents its own.
5. **Attribution and commit conventions** your company requires for AI-assisted work (`06`, section 4).
6. **Security contacts and escalation** (`07`, section 8): who to ask when a secret or personal data appears.

### 3.3 Prepare the tooling the kit assumes
- An AI coding agent that can read and write files, run commands and open pull requests (written for Claude Code).
- A code host with pull requests, required status checks, protected branches and environments (examples use GitHub and GitHub Actions; the host CLI, for example `gh`, makes git facts reliable).
- A scripting runtime for the zero-dependency drift and branch-policy scripts (examples are `.mjs`; use your standard language).
- A place for the hub page (3.2, item 4) and, if the project has screens, a headless browser or the stack's golden-test tooling for screenshots.
- A secret store the team already uses. **No secret ever goes in the repository, the prompt, or the hub.**

### 3.4 Pilot before you roll it out
1. Pick a **small, low-risk, real project** (two or three apps at most, no regulated data). Do not pilot on a critical system.
2. Run the whole kit on it with the project owner present at every gate.
3. Keep a **pilot log**: where the agent asked a bad question, skipped a step, invented a path, wrote a check that never failed, or the owner got confused. Time each gate.
4. After the pilot, fix the kit from the log (the retrospective in `08`, section 8), bump the version, and only then offer it to other teams.
5. Judge it with three numbers: time from idea to Gate 5; how many documents or checks were wrong when first written; whether a new person could run the checks and understand the hub in an hour.

### 3.5 Roles
| Role | Does |
|---|---|
| **Kit owner** | accepts changes, tags versions, keeps the company defaults current |
| **Project owner** | answers the interview, approves each gate, decides open questions, holds the owner actions |
| **Tech lead** | reviews the architecture (Gate 2) and the guardrails (Gate 4); required reviewer on security rules |
| **Security / legal contact** | confirms any legal statement the agent marks "unverified"; reviews the privacy ADR |
| **Developers** | work through pull requests under `CLAUDE.md`; run the checks locally |
| **The agent** | interviews, proposes, writes, checks, reports honestly |

### 3.6 Habits that make it stick
- **Never skip a gate** to save time. The gates are what make the later phases cheap.
- **Put every decision in an ADR the same day.** Chat is not a record.
- **Refresh the hub as part of finishing a task**, not afterwards.
- **Start every new session by reading `CLAUDE.md`** and the open pull requests, and have the agent say where things stand in two lines before it acts.
- **Run the monthly golden-set check** and add an item for every hallucination you see.
- **Read the weekly drift report.** Warnings that grow are a smell.
- **Keep the owner in the loop through the hub**, not long chat threads.

---

## 4. How to use it on a new project

### Before the first session (about 15 minutes)
1. Create an empty repository (or a clean branch) on your code host. Do not add code.
2. Copy this folder into it as `documentation/templates/project-bootstrap-prompt/`, **or** keep it outside and give the agent its absolute path. Either way the agent must be able to read every file (`00` to `11` and `hub-template/`).
3. Write down, in a note you can paste, what you already know: what you are building, who for, what exists today, constraints (law, deadline, company standards), and the apps you imagine. Rough is fine; section 5 has an example.
4. Decide who the **project owner** is (the person who approves gates) and make sure they are in the session.
5. Decide where the hub will live and who may view it (3.2, item 4).

### The first session
1. Open the AI agent in the repository root.
2. Paste the **entire contents of `00-master-system-prompt.md`** as your first message, plus one line: *"The companion files are in `<path>`. Read each when the phase names it."*
3. The agent replies with one question: what are you building, who is it for, what problem does it solve, what exists, and what apps you imagine. Paste your note.
4. **Answer the interview** (`01`). It asks at most four questions at a time. If you do not know, say "you decide"; it records the default and why.
5. **Gate 1:** it reads the **Project Brief** back. Correct anything wrong, then say "confirmed". Nothing is built before this.
6. **Gate 2:** it proposes the architecture (apps, boundaries, stack with one rejected alternative each, diagrams, tenancy/identity/authorization, phases, open decisions) and drafts the starter ADRs. **This is the cheapest time to change your mind.** Accept, change or strike each ADR and answer the open decisions, or let defaults stand (they stay marked as defaults).
7. **Gate 3:** it writes the documents, `CLAUDE.md` and the ADRs. Read `CLAUDE.md`, `guardrails.md` and the plan: every later session will obey them.
8. **Gate 3b:** it builds the **Project Hub** from `hub-template/`, seeds the plan, draws the diagrams, reads git facts, and gives you the link. Open it on a phone and with the keyboard only; tell the agent what is unclear.
9. **Gate 4:** it writes the drift check, evals, golden set (and grader) and the first tests, **each shown failing before it passes**, and reports what ran. Ask it to show one check failing on a deliberately wrong input.
10. **Gate 5:** it creates the branch model, CI, PR template and CODEOWNERS and opens the first pull request through that very workflow. It lists the **owner actions** it cannot do (branch protection, environments, secrets, hub access). **Do them.**
11. Merge that PR (the owner merges). The project is established.

### What you say at each gate
| You say | Meaning |
|---|---|
| "proceed" | accept the gate and continue |
| "change X" | revise; the agent updates the documents, not just the chat |
| "what are the options for X" | the agent lists 2 to 4 with a recommendation and cost |
| "you decide" | the agent picks the default and records it as **Decided by the assistant** |
| "stop, explain" | the agent summarises state in under 15 lines |

### Every later session
1. Start with: *"Read `CLAUDE.md`, the latest gate document and the open pull requests, and tell me in two lines where we are."*
2. Give one task at a time by ID (for example `P1-3a`). The agent follows the task loop in `07`: read, plan in 8 lines, **evals first**, build, check, document, open a PR with evidence, refresh the hub, report.
3. It reports in a fixed format (Done / PR / Ran / Not verified / Owner actions / Next). Read **Not verified** and **Owner actions** first.
4. Merging and deploying are yours. The agent opens PRs and does not merge to the pre-production or production branches.
5. When you answer a question in chat, check the agent wrote it into the ADR or plan.

### Changing the project later
- **A new feature:** add it to the plan with task IDs; the agent says what it depends on and what it displaces; write the module document first.
- **A decision changes:** a new ADR that supersedes the old one; update the affected tasks; add a golden item or drift rule if the old belief could return.
- **A hallucination:** add a golden item and, if it can be detected mechanically, a drift rule.
- **A new app or service:** update the architecture, the ownership map in the branch-policy config, the hub diagrams, and the drift planned-paths.

---

## 5. Worked example of an opening answer
> "We're building a booking and invoicing system for small physiotherapy clinics. Clinic staff use it at a desk; patients book from their phones. Today they use paper diaries and WhatsApp. We have to follow Indian data-protection law for health data and want a pilot with three clinics in four months. I imagine a staff web app, a patient phone app, an admin site for us to manage clinics, and an API. Payments through an Indian provider. Our company standard is TypeScript and Postgres."

The agent should then ask about tenancy (one database for all clinics?), sensitive data, offline needs, who approves refunds, languages and so on, a few questions at a time, and write the brief.

## 6. Troubleshooting
| Symptom | Fix |
|---|---|
| The agent starts coding or choosing a stack before Gate 2 | Say "stop, you skipped a gate; return to Gate 1/2" and point at `00`, section 2, rules 1 and 3. |
| The agent invents file names or library versions | Remind it: "verify before asserting" (`07`, section 1). Add the miss to the golden set. |
| Documents contradict each other | Ask it to run the drift check and list the conflicts; resolve with an ADR. |
| The context window fills mid-project | State lives in the documents. Start a new session with the "two lines where we are" prompt. |
| The hub is stale | Ask for the refresh routine (`09`, section 5) and make "hub refreshed" part of every PR checklist. |
| A check never fails | It is decoration. Ask the agent to prove it fails on a bad input (`05`, section 8). |
| Branch rules are ignored | Host protection is not on; do the owner actions from the Gate 5 report. |
| The kit does not fit our stack | Edit the company defaults (3.2) and bump the version; do not hand-patch one project. |

## 7. Placeholders
Files use `{{LIKE_THIS}}` for values the agent fills in from your answers. A placeholder left in a delivered document is a bug; the drift check searches for them.

## 8. Honest limits
The kit makes errors cheaper to catch; it does not make an agent infallible. Anything about the law, a vendor's current API, a price or a deadline must be checked by a person against the source. The kit tells the agent to say "unverified" instead of guessing, and the golden set includes trap questions to test that. The kit has not been piloted yet (3.4); treat the first project as the test.

## 9. Changing the kit
1. Raise a pull request against the canonical copy.
2. State the project, the failure, and the rule or check that would have prevented it.
3. Update the version line and add a changelog line.

### Changelog
- 0.1 (2026-10-31): first version: interview, documents, ADRs, guardrails, hallucination checks, drift/evals/tests, git and PR system, agent protocol, phase plan, Project Hub and template.
