# 11. Screens, architecture and git on the hub

How to fill the three tabs that need real work: **Screens**, **System flow** and **Git and glossary**.

## 1. Screens: capture from the real code
Principle: the owner reviews what the code actually renders, with sample data, in each language and direction. Never draw mock-ups.

### Web apps
1. A throwaway test (not committed, or committed under `tools/hub/`) renders each screen component with a fake API and sample data, once per language (default and, if the product has right-to-left, one RTL language).
2. It writes the rendered HTML (`document.body.innerHTML`) to a temporary folder.
3. Assemble a page from that HTML plus the app's **built stylesheet** (so it looks exactly as shipped). A small script (`tools/hub/build-shots-css.mjs`) concatenates the built CSS and the fonts.
4. Screenshot each page with a headless browser at the device width (desktop 1280, phone 390) to PNG; trim, compress (keep each under ~150 KB).
5. Upload the PNGs to the asset store; write the `screens` document entries with `images` pointing to them.
6. Delete the throwaway test if it was temporary.

### Mobile apps (Flutter or similar)
- Use widget tests with golden files (`matchesGoldenFile`) or a test that pumps the screen with fake services and writes a PNG, in each language/direction, at a phone size and a larger text scale. Fonts must be the bundled ones, never network fonts, or the images differ between machines.
- Mark which screens were captured from the simulator and which from tests; say which.

### What to capture
- Each screen's **default data** state; add **empty**, **loading** (if visible), **error/offline**, and **permission denied** only where the screen differs materially.
- At least one **right-to-left** and one **large text** capture per app, if the product supports them.
- Role variants only where the screen differs by role.

### Per-screen metadata
App, title, one-sentence purpose, task ID and PR, roles, states, status (`planned` shows a placeholder card with the intended content; `built` shows the image; `open-pr` shows the image with an "Open pull request" tag).

### Review
Each screen card has a Review button (Approve / Request changes + comment). "Request changes" creates a work item the agent must resolve or reply to; resolution is recorded on the review record.

## 2. System flow: keep architecture honest
- Source: the diagrams in `system_architecture.md` and the ADRs. The hub copies the Mermaid source into `flows`.
- **Colour means status**: built and tested (solid, green), planned or not tested against the real service (dashed, grey). A part turns green only when its checks ran, not when code exists.
- Update triggers: an ADR accepted or amended, a planned part built, a boundary changed, a vendor changed.
- Minimum set: (1) components and boundaries; (2) one request through every guard (token → rate limit → tenant → permission → validation → rules → transaction with audit); (3) onboarding of an organisation and its users; (4) the branch model and checks; (5) every hard flow (sync, payments, notifications, analytics).
- Render diagrams lazily (when the tab is first opened); a diagram drawn in a hidden tab may render blank.
- Add a one-line note under each diagram saying what is **not** shown.

## 3. Git and glossary: read, don't remember
Gather at refresh time (and print `readAt`):
```
git fetch --all --prune
git branch -a -vv
git for-each-ref --format='%(refname:short) %(committerdate:short)' refs/heads refs/remotes
git rev-list --left-right --count origin/<prod>...origin/<pre>      # how far production is behind
gh pr list --state all --limit 50 --json number,title,state,headRefName,baseRefName,mergedAt
```
Then: tree of branches (prod, pre, app branches, open cross/topic branches), table of branches (owner, purpose, ahead/behind), module ownership map (from the branch-policy config), PR ledger (task, state), the merge rules (copied from the branching ADR), **owner actions** (branch protection, environments, secrets, deploys) and the glossary.
- Show red status lines for: production far behind pre-production; branch protection off; failing required checks; long-open PRs.
- If the host CLI is not available, say "unverified: could not read the host" instead of reusing old facts.

## 4. Optional tabs
- **Demo**: how to start a local demo with sample accounts and what each account can try. Explicitly local-only; refuses to run against a real project.
- **Decisions**: ADR list (number, title, status, date) and open owner questions with options.
- **Risks**: top risks from the plan with owner and date.
- **Environments**: staging/production URLs, last deploy, health, versions (from the host, with the date read).

## 5. Checklist before telling the owner "the hub is current"
- [ ] Plan data matches `implementation_plan.md` (IDs) and the merged PRs.
- [ ] Overall and per-phase percents computed by the page.
- [ ] Every new screen from this task is on the Screens tab with images in each language.
- [ ] Diagram statuses match what exists.
- [ ] Git tab shows today's read date and matches `gh`.
- [ ] No secrets, tokens or real personal data anywhere on the page.
- [ ] The page opens on a phone and with the keyboard only; contrast and focus visible.
- [ ] The final message of the task lists what changed on the hub.
