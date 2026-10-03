# Agent Coordination Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Give Sedat's Claude, Antigravity and yeet30's Claude one shared rulebook, a GitHub Issues task model and five commands, so they coordinate without humans relaying messages and never break the challenge or repo rules.

**Architecture:** GitHub Issues and labels are the shared state. `AGENTS.md` is the single rulebook, imported by `CLAUDE.md` and `GEMINI.md`. Each routine step is written in plain text in `AGENTS.md` and wrapped as a Claude Code slash command in `.claude/commands/`. Antigravity runs headless in its own worktree and never writes to GitHub.

**Tech Stack:** Markdown, `gh` CLI 2.94, `git` 2.52, Claude Code slash commands, Antigravity CLI (`agy`), GitHub rulesets.

**Spec:** `docs/specs/2026-10-03-agent-coordination.md`

## Global Constraints

- English only in every file, commit and PR (R4).
- Commit subject under 72 characters, body explains why. No `Co-Authored-By: Claude` line (`AGENTS.md` §6).
- Work on branch `docs/agent-coordination`. Never push to `main`. Merge only through a PR (R8).
- Never bypass a block: no `--force`, `--no-verify`, `--admin`, `--dangerously-skip-permissions`, no editing rules to allow yourself (R9).
- `RULES.pdf` is never committed (R7).
- Antigravity runs only in `X:\Beltur-agy`, always with `--sandbox`, never pushes or uses `gh`.
- Team table values: `sedatsan` = Sedat, role `TBD`. `yeet30` = teammate, role `TBD`. Prelint status: `not confirmed`.
- Commands must work in Git Bash on Windows and in a macOS shell: no GNU-only flags such as `date -d`.

## Review Focus

1. **Role still `TBD`:** `/next-task`, `/request` and `/delegate` stop and say "Role for <login> is not set in AGENTS.md". They never guess a role. (Test in Task 4.)
2. **Uncommitted changes:** `/sync` and `/next-task` stop and list the modified tracked files. They never stash, reset or discard. (Test in Task 4.)
3. **Issue already assigned to someone else:** `/next-task` skips it, and after claiming it re-reads the assignees. If anyone else is on it, it removes only itself and moves on. (Test in Task 4.)
4. **`gh` fails (offline or logged out):** every command stops and reports the exact error. None continues on stale or missing data. (Test in Task 4.)
5. **Antigravity edits a file outside its allowlist:** `/delegate` rejects the whole diff and says which file broke the allowlist. (Test in Task 5.)

---

### Task 1: GitHub labels, ruleset and tracking issue

**Files:** none (GitHub settings only)

**Interfaces:**
- Produces: labels `role:app`, `role:platform`, `P0`, `status:in-progress`, `status:blocked`, `request`, `contract`, `agent:antigravity`, `chore`; ruleset "Protect main" with `pull_request` (0 approvals), `non_fast_forward`, `deletion`; tracking issue number `<T>` used in Task 6.

- [ ] **Step 1: Check the current state**

Run: `gh api repos/sedatsan/Beltur/rules/branches/main --jq '[.[].type]|sort'`
Expected: `["deletion","non_fast_forward"]`, so `pull_request` is missing.

- [ ] **Step 2: Create the labels**

`gh label create <name> --color <hex> --description "<text>"`, one per label in Interfaces. Descriptions come from spec §4. Colours: roles blue, `P0` red, statuses yellow, `request`/`contract` purple, `agent:antigravity` grey, `chore` grey.

- [ ] **Step 3: Add the pull-request rule to ruleset 24411024**

`gh api -X PUT repos/sedatsan/Beltur/rulesets/24411024` with the existing two rules plus `{"type":"pull_request","parameters":{"required_approving_review_count":0,"dismiss_stale_reviews_on_push":false,"require_code_owner_review":false,"require_last_push_approval":false,"required_review_thread_resolution":false}}`. Leave the bypass list empty.

- [ ] **Step 4: Verify**

Run: `gh api repos/sedatsan/Beltur/rules/branches/main --jq '[.[].type]|sort'`
Expected: `["deletion","non_fast_forward","pull_request"]`
Run: `gh label list --json name --jq '[.[].name]'`
Expected: contains all nine labels from Interfaces.

- [ ] **Step 5: Create and claim the tracking issue**

`gh issue create --title "Set up agent coordination workflow" --label chore --assignee @me --label status:in-progress --body "<goal + done-when from spec §1 and §9; links the spec>"`. Record its number as `<T>`.

---

### Task 2: Rules summary, AI usage log, ignore RULES.pdf

**Files:**
- Create: `docs/challenge-rules.md`
- Create: `docs/AI_USAGE.md`
- Modify: `.gitignore` (append a `# --- Third-party documents (never commit) ---` block with `RULES.pdf`)

**Interfaces:**
- Produces: `docs/AI_USAGE.md` with headings `## Tools`, `## Pre-existing work (before the brief)`, `## Third-party components`, `## Contributions by agent`. Later tasks and every future PR append to these.

- [ ] **Step 1: Write the failing checks**

```bash
git check-ignore -q RULES.pdf && echo IGNORED || echo NOT_IGNORED
grep -c "20%" docs/challenge-rules.md
grep -c "^## " docs/AI_USAGE.md
```
Expected now: `NOT_IGNORED`, then "No such file" twice.

- [ ] **Step 2: Write `docs/challenge-rules.md`**

In our own words, no copied passages: organiser and dates (3–4 Oct 2026, Tauron Arena Kraków), objective, team size (up to 6), submission (English, before the deadline, may include description, slides, screenshots, demo, repo), AI and pre-existing work allowed if disclosed, the scoring table with all six weights, the 50% threshold, an IP summary (you keep ownership, winners grant Huawei a 3-year non-exclusive demo/promo licence, no commercial rights), disqualification grounds. End with "What this means for us": a mapping of R1–R7 to the rule sections.

- [ ] **Step 3: Write `docs/AI_USAGE.md`**

The four headings from Interfaces.
- Tools: Claude Code (Anthropic), Antigravity (Google Gemini), Prelint.
- Pre-existing work: the repo setup on 2026-10-03 before the brief. That covers README, CONTRIBUTING, templates, `.gitignore`, `.gitattributes`, LICENSE, `.mcp.json`, `AGENTS.md`/`CLAUDE.md`/`GEMINI.md`, this coordination system, and spec and plan. It is all workflow tooling with no product code.
- Third-party components: "None yet".
- Contributions by agent: one row per PR (PR, agent, summary).

- [ ] **Step 4: Append the ignore block to `.gitignore`**

- [ ] **Step 5: Run the checks**

Same commands as Step 1. Expected: `IGNORED`, a count of at least 6, and `4`.

- [ ] **Step 6: Commit**

```bash
git add docs/challenge-rules.md docs/AI_USAGE.md .gitignore
git commit -m "Add challenge rules summary and AI usage log"
```

---

### Task 3: AGENTS.md rulebook and entrypoints

**Files:**
- Modify: `AGENTS.md` (currently untracked, written by Antigravity). Add sections. Keep the existing text except where named below.
- Modify: `CLAUDE.md`, whose full content becomes the line `@AGENTS.md` under a one-line comment
- Modify: `GEMINI.md`, the same
- Modify: `.github/ISSUE_TEMPLATE/task.md` (add `Depends on:` and `Allowed files (for delegation):` sections)

**Interfaces:**
- Produces, in `AGENTS.md`:
  - `## 12. Team & roles`: a table with columns `GitHub login | Human | Role | Agents`. Rows: `sedatsan | Sedat | TBD | Claude Code (lead), Antigravity (worker)` and `yeet30 | Teammate | TBD | Claude Code (lead)`. Also the lines `Prelint reviews PRs: not confirmed` and `Submission deadline: TBD`.
  - `## 13. Rule boundaries`: R1–R9 copied verbatim from spec §2.
  - `## 14. Coordination routine`: the labels table (spec §4), the ownership table (spec §5), and the steps of `sync`, `next-task`, `request`, `ship`, `delegate` in plain text (spec §6). Each step carries the exact `gh`/`git` commands that Task 4 and Task 5 use.
  - `## 15. Antigravity limits`: spec §7 bullets.
- Edits to existing text:
  - In §10 "Hierarchy", add yeet30 as lead of their role.
  - In §11 "Slave Execution Protocol", change the Obsidian lines so they say Obsidian is Sedat's personal log and GitHub Issues/PRs are the shared record.
  - In §10 "Conventions", change the Prelint bullet to point at `docs/`.

- [ ] **Step 1: Write the failing check**

```bash
claude -p "From your project instructions only: quote rule R9, then say what role yeet30 has. Two lines." --allowedTools "Read"
```
Expected now: it cannot quote R9, because `CLAUDE.md` only links `AGENTS.md` and R9 doesn't exist yet.

- [ ] **Step 2: Add sections 12–15 and the three edits to `AGENTS.md`**

- [ ] **Step 3: Rewrite `CLAUDE.md` and `GEMINI.md` to import with `@AGENTS.md`**

- [ ] **Step 4: Extend the task template**

Add `## Depends on` (format `Depends on: #12, #15`, delete if none) and `## Allowed files (for delegation)` (one path or glob per line).

- [ ] **Step 5: Run the check**

Same command as Step 1. Expected: line 1 contains "Never bypass a block". Line 2 says yeet30's role is not set yet (TBD).

- [ ] **Step 6: Commit**

```bash
git add AGENTS.md CLAUDE.md GEMINI.md .github/ISSUE_TEMPLATE/task.md
git commit -m "Add team roles, rule boundaries and routine to AGENTS.md"
```

---

### Task 4: Lead-agent slash commands

**Files:**
- Create: `.claude/commands/sync.md`, `.claude/commands/next-task.md`, `.claude/commands/request.md`, `.claude/commands/ship.md`

Each file has frontmatter `description:`, plus `argument-hint:` where an argument is used. The body tells the agent to follow `AGENTS.md` §14 for that step and lists the commands below. If any `gh` or `git` command fails, it stops and reports the error (Review Focus 4).

**Interfaces:**
- Consumes: the `AGENTS.md` §12 table (login to role), §14 steps, labels from Task 1.
- Produces: `/sync`, `/next-task`, `/request <what the other side must provide>`, `/ship`.

Exact commands (role comes from §12 for `gh api user --jq .login`; `<OTHER>` is the other role):
- **Shared first step (dirty check):** `git status --porcelain --untracked-files=no`. If the output is not empty, list the files and stop (Review Focus 2).
- **`/sync`:**
  - `git fetch origin --prune`
  - on `main`: `git pull --ff-only`. On any other branch: `git merge --no-edit origin/main`. On conflict, `git merge --abort` and stop.
  - then these five queries:
    - `gh issue list --state open --label request --label role:<ROLE> --json number,title,author`
    - `gh issue list --state open --assignee @me --label status:in-progress --search "-label:agent:antigravity" --json number,title`
    - `gh issue list --state open --label status:in-progress --json number,title,assignees,labels`
    - `gh pr list --state open --json number,title,author,headRefName,files`
    - `gh pr list --label contract --state merged --limit 5 --json number,title,mergedAt` and `gh issue list --state open --label status:blocked --json number,title`
  - then print a briefing of at most 15 lines.
- **`/next-task`:**
  - Role `TBD`: stop with "Role for <login> is not set in AGENTS.md" (Review Focus 1). Run the dirty check.
  - If I already have an in-progress issue, stop.
  - Candidates: `gh issue list --state open --label role:<ROLE> --search "no:assignee -label:status:in-progress -label:status:blocked -label:agent:antigravity sort:created-asc" --json number,title,labels,body --limit 50`. Take `P0` first. Skip an issue unless every `Depends on` issue is closed (`gh issue view <n> --json state`).
  - Claim: `gh issue edit <N> --add-assignee @me --add-label status:in-progress`. Then `gh issue view <N> --json assignees --jq '[.assignees[].login]'`. If anyone else is listed, run `gh issue edit <N> --remove-assignee @me` and try the next candidate (Review Focus 3).
  - Branch and announce: `git switch -c <feat|fix>/<N>-<slug> origin/main`, then `gh issue comment <N> --body "Claimed by <agent> (<login>) on branch <branch>."`
- **`/request`:**
  - Role check as in `/next-task`.
  - `gh issue create --title "Request: <summary>" --label request --label role:<OTHER> --body` with sections: What is needed (signature or data shape), Why, Waiting issue `#<N>`, Mock in place (path).
  - Then `gh issue comment <my N> --body "Waiting on #<new>; using mock <path>."`
- **`/ship`:**
  - Refuse on `main`.
  - Commit pending changes only if every changed path is in my role's folders, a shared file covered by a contract or chore branch, or `docs/AI_USAGE.md`. Otherwise stop.
  - `git fetch origin && git merge --no-edit origin/main`. On a conflict outside my folders, `git merge --abort` and stop.
  - Verify: if `hvigorfile.ts` exists, run `hvigorw assembleHap --no-daemon`. Otherwise record "No build yet (project not scaffolded)".
  - If `git diff --name-only origin/main...HEAD` includes `oh-package.json5` or new files under `resources/`, then `docs/AI_USAGE.md` must also be in the diff. Otherwise stop.
  - `git push -u origin HEAD`.
  - `gh pr view --json number,state` to find an existing PR. If none, `gh pr create --base main --title <title> --body` with sections Closes #N, Files touched, Contract changes, Verification, AI usage.
  - Merge gate: `gh pr view <n> --json mergeable,reviews,comments`. It must be `MERGEABLE`, verification must have passed, and Prelint must have posted with no blocking finding. While §12 says `Prelint reviews PRs: not confirmed`, ask the human before merging.
  - Merge: `gh pr merge <n> --squash --delete-branch`. Only for my own PRs, plus Antigravity PRs for Sedat's Claude.

- [ ] **Step 1: Write the four command files**

- [ ] **Step 2: Test Review Focus 1 (role TBD)**

Run: `claude -p "/next-task" --allowedTools "Bash(gh:*)" "Bash(git:*)" "Read"`
Expected: output contains `Role for sedatsan is not set in AGENTS.md`. Then `gh issue list --assignee @me --label status:in-progress` still shows only `<T>`.

- [ ] **Step 3: Test Review Focus 2 (uncommitted changes)**

Append one line to `README.md`, then run `claude -p "/sync" --allowedTools "Bash(gh:*)" "Bash(git:*)" "Read"`.
Expected: it stops and lists `README.md`, and runs no merge. Undo with `git restore README.md`.

- [ ] **Step 4: Test the happy path of `/sync`**

Run: `claude -p "/sync" --allowedTools "Bash(gh:*)" "Bash(git:*)" "Read"`
Expected: a briefing of 15 lines or fewer that lists `<T>` as my in-progress issue and makes no changes (`git status --porcelain --untracked-files=no` stays empty).

- [ ] **Step 5: Test Review Focus 3 and 4 by reading**

Confirm that `next-task.md` contains the re-read of assignees after claiming and the remove-only-myself step, and that all four files contain the stop-on-`gh`-error instruction.
Run: `grep -l "remove-assignee @me" .claude/commands/next-task.md && grep -L "stop" .claude/commands/*.md`
Expected: prints `next-task.md`, then nothing (every file mentions stopping).

- [ ] **Step 6: Commit**

```bash
git add .claude/commands
git commit -m "Add sync, next-task, request and ship commands"
```

---

### Task 5: `/delegate` and the Antigravity dry run

**Files:**
- Create: `.claude/commands/delegate.md`

**Interfaces:**
- Consumes: the `AGENTS.md` §15 limits; the `Allowed files (for delegation)` section from the task template (Task 3).
- Produces: `/delegate <issue number>`, for Sedat's Claude only.

`delegate.md` must say:
- Stop unless the login is `sedatsan`, the role is set, the issue carries Sedat's role label, and the issue has a non-empty `Allowed files` section.
- Claim: `gh issue edit <N> --add-label agent:antigravity --add-label status:in-progress --add-assignee sedatsan`.
- Worktree:
  - If `../Beltur-agy` is missing: `git worktree add ../Beltur-agy -b agy/<N>-<slug> origin/main`.
  - Otherwise `git -C ../Beltur-agy status --porcelain` must be empty, then `git -C ../Beltur-agy switch -c agy/<N>-<slug> origin/main`.
- Run, from inside `../Beltur-agy`: `agy -p "<prompt>" --mode accept-edits --sandbox --print-timeout 20m`. The prompt template is fixed in the file and holds:
  - the issue title, Goal, Done when and Allowed files
  - R1–R9
  - "Commit only on agy/<N>-<slug>. Never push. Never run gh. Never edit files outside Allowed files. If anything blocks you, stop and report."
- Review:
  - `git -C ../Beltur-agy status --porcelain`
  - `git -C ../Beltur-agy diff --name-only origin/main` (committed and uncommitted)
  - If any path is outside Allowed files: reject, name the path, `git -C ../Beltur-agy reset --hard origin/main` (Antigravity's own branch only), and comment on the issue (Review Focus 5).
  - Otherwise read the full diff and run the Done-when checks.
- Accept: commit anything left uncommitted with subject suffix ` (Antigravity)`, add a row to `docs/AI_USAGE.md` under Contributions by agent, then run `/ship` from `../Beltur-agy`. If it's rejected twice, Sedat's Claude does the task itself.
- Never use `--dangerously-skip-permissions`. If Antigravity is blocked by a permission prompt, report it and do not escalate (R9).

- [ ] **Step 1: Write `delegate.md`**

- [ ] **Step 2: Create the dry-run worktree from this branch**

Run: `git worktree add ../Beltur-agy -b agy/dry-run docs/agent-coordination`
Expected: `Preparing worktree (new branch 'agy/dry-run')`

- [ ] **Step 3: Antigravity dry run (read-only)**

Run in `X:\Beltur-agy`: `agy -p "Read AGENTS.md. List rules R1-R9 in one line each, then list everything you must never do as the Antigravity worker. Do not edit any file." --mode plan --sandbox --print-timeout 10m`
Expected: all nine rules are restated, including "never push", "never run gh" and "never bypass a block". `git -C ../Beltur-agy status --porcelain` is empty afterwards.
If `agy` errors or wants a permission prompt, stop and report it to Sedat. Do not raise its permissions.

- [ ] **Step 4: Test Review Focus 5 (allowlist violation)**

1. In the worktree, create a scratch file outside a pretend allowlist (`echo x > ../Beltur-agy/OUTSIDE.txt`).
2. Follow the review steps of `delegate.md` with allowlist `docs/AI_USAGE.md`.
Expected: it rejects and names `OUTSIDE.txt`. Clean up with `rm ../Beltur-agy/OUTSIDE.txt`.

- [ ] **Step 5: Remove the dry-run worktree**

Run: `git worktree remove ../Beltur-agy && git branch -D agy/dry-run`
Expected: `git worktree list` shows only `X:/Beltur`.

- [ ] **Step 6: Commit**

```bash
git add .claude/commands/delegate.md
git commit -m "Add delegate command for Antigravity worker"
```

---

### Task 6: Docs pointers and ship through the routine

**Files:**
- Modify: `README.md`: add `## Development workflow` before `## License`. Three sentences: agents follow `AGENTS.md`, tasks are GitHub Issues, AI usage is logged in `docs/AI_USAGE.md`. Fill in the Team table with `@yeet30`.
- Modify: `CONTRIBUTING.md`: add one line under "How we work" pointing agents to `AGENTS.md` §14. Remove the "merging your own PR is fine" sentence that conflicts with the merge gate.
- Modify: `docs/specs/2026-10-03-agent-coordination.md` §11: mark the teammate login as known (`yeet30`, has write access).
- Modify: `docs/AI_USAGE.md`: add a row for this PR under Contributions by agent.

- [ ] **Step 1: Make the four edits**

- [ ] **Step 2: Check that no forbidden file is staged**

Run: `git status --porcelain && git diff --cached --name-only | grep -E "RULES.pdf|\.p12|\.env" || echo CLEAN`
Expected: `CLEAN`.

- [ ] **Step 3: Commit**

```bash
git add README.md CONTRIBUTING.md docs/specs/2026-10-03-agent-coordination.md docs/AI_USAGE.md
git commit -m "Point docs to the agent workflow and record yeet30"
```

- [ ] **Step 4: Run `/ship` for this branch, following `.claude/commands/ship.md` exactly**

Expected:
- the branch is pushed
- a PR exists with `Closes #<T>`
- the merge gate stops and asks Sedat before merging, because Prelint is not confirmed

- [ ] **Step 5: After Sedat approves, merge**

Run: `gh pr merge <n> --squash --delete-branch`, then `git switch main && git pull --ff-only`.
Expected: `gh issue view <T> --json state --jq .state` gives `CLOSED`, and `git log --oneline -1` on `main` shows the squash commit.
