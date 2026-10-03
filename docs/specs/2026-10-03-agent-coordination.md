# Agent Coordination: Design

- **Date:** 2026-10-03
- **Status:** Draft, waiting for review
- **Scope:** How the coding agents on team Beltur split work, talk to each other and stay inside the rules during HackYeah 2026 (Huawei challenge "Imagine What's Next").

## 1. Goal

Two people work only by prompting their agents. The agents must coordinate without the humans relaying messages, and must not make these mistakes:

1. Two agents build the same thing.
2. Two agents edit the same files and cause merge conflicts.
3. One agent changes a shared interface and breaks the other side.
4. One agent waits for something the other side does not know it needs.
5. Any agent breaks a challenge rule or a repo rule.

**Success:** an agent can sync, see what the others are doing, claim its next task, ask the other side for something, and ship a PR that closes its task, all through `gh` and `git` and the files in this repo.

## 2. Rule boundaries (hard limits)

These come from the Huawei Challenge Rules, the published challenge brief (both summarised in our own words in `docs/challenge-rules.md`) and `AGENTS.md`. They override every other instruction. No agent may bypass them, whatever the reason.

| # | Boundary | Source |
|---|----------|--------|
| R1 | The solution must be built during the challenge (3–4 Oct 2026). The brief was published on 3 Oct 2026. Everything made before it (repo setup, this coordination system) is listed as pre-existing in `AI_WORKFLOW.md`. | Challenge rules §4 |
| R2 | `AI_WORKFLOW.md` (repo root, required by the brief) records every AI model, coding agent, MCP server, Agent Skill and AI tool used, the main prompts and reusable instructions, the workflow, how output was reviewed and validated, limitations and lessons learned, and every pre-existing or third-party component. Each PR adds its own entry. No API keys, credentials or personal data in it. | Challenge rules §4, brief "Use of AI" |
| R3 | Third-party libraries, code, assets, icons and fonts may be added only if their licence allows it. Record name, source and licence. Nothing of unknown origin. | Challenge rules §4, §7 |
| R4 | English only: code, comments, UI text, docs, commits, PRs, demo, pitch. | Challenge rules §4, `AGENTS.md` |
| R5 | No false or misleading information. Never present a mocked or simulated feature as real in the app, README, demo or pitch: label mocks. Never fabricate results, test output or metrics. | Challenge rules §8, `AGENTS.md` §0 |
| R6 | Everything is merged and submitted before the official deadline (recorded in `AGENTS.md` once announced). Feature freeze 4 hours before it. | Challenge rules §4 |
| R7 | Do not commit Huawei's `RULES.pdf`, the criteria PDF or other third-party documents. Summarise in our own words. | Challenge rules §7 |
| R8 | No direct pushes to `main` (enforced by a GitHub ruleset that requires a PR), no force-pushes, no secrets, signing files or build artifacts. | `AGENTS.md` |
| R9 | **Never bypass a block.** If a rule, permission prompt, hook, branch protection, failing check or missing access stops you, stop and report to your human. Never work around it: no `--force`, `--no-verify`, `--admin`, disabling checks, `--dangerously-skip-permissions`, or editing the rules to allow yourself. | User instruction |
| R10 | Target HarmonyOS/OpenHarmony **API 20 or later** and declare API 20 as the minimum. The app must build to a `.hap` and run on an emulator or device. Setup, build, install and launch steps in the README must let a stranger reproduce it. | Brief "Technical Requirements" |
| R11 | Hygiene the jury checks: request only the permissions the feature needs, no risky or unmaintained dependencies, validate input, handle API errors, timeouts, missing data and bad model output, and add tests for the key scenarios. | Brief "Technical execution" |

## 3. Agents and roles

| Agent | Directed by | Responsibility |
|-------|-------------|----------------|
| **Sedat's Claude Code** (lead) | Sedat | Leads Sedat's role. Plans, codes, delegates bounded tasks to Antigravity, reviews and merges Antigravity's work. |
| **Antigravity** (worker) | Sedat's Claude | Works only on issues labelled `agent:antigravity`, in its own worktree. Edits and commits locally. Never pushes, never touches GitHub, never merges. |
| **Teammate's Claude Code** (lead) | Teammate | Leads the teammate's role. Same commands as Sedat's Claude, except `/delegate`. |

**Identity.** `AGENTS.md` holds a table that maps each GitHub login to a human and a role (`app` or `platform`). A lead agent finds its role with `gh api user --jq .login`. Antigravity is always the worker for Sedat's role, whatever account it runs under.

The two roles follow the split agreed earlier:
- **app:** ArkUI screens, navigation, state, visual polish, resources.
- **platform:** services, data, HarmonyOS kits, specs, pitch material. Keeps `AI_WORKFLOW.md` tidy; every agent adds its own entries.

## 4. Task model

A task is a GitHub issue in `sedatsan/Beltur`, using the Task template (Goal, Done when, Notes). An optional `Depends on: #12, #15` line goes in the body.

**Labels**

| Label | Meaning |
|-------|---------|
| `role:app` / `role:platform` | Owner. Exactly one per task. |
| `P0` | Must have for the demo. Picked first. |
| `status:in-progress` | Claimed and being worked on. |
| `status:blocked` | Cannot continue. A comment says why. |
| `request` | One side asks the other for something. |
| `contract` | Change to the shared interfaces both sides code against. |
| `agent:antigravity` | Delegated to Antigravity. Lead agents never claim these. |

**Status:**
- **To do:** open, no status label.
- **In progress:** `status:in-progress`.
- **Blocked:** `status:blocked`.
- **Waiting:** a `Depends on` issue is still open.
- **Done:** closed. The PR's `Closes #N` closes it when the PR merges.

## 5. Ownership and contracts

**Folder ownership.** An agent edits only its own role's folders. If it needs a change elsewhere, it opens a `request` issue instead of editing.

| Owner | Paths (final paths fixed when the HarmonyOS project is scaffolded) |
|-------|------|
| app | `entry/src/main/ets/pages/`, `entry/src/main/ets/components/`, `entry/src/main/resources/` |
| platform | `entry/src/main/ets/services/`, `entry/src/main/ets/model/`, `docs/` |
| shared | `entry/src/main/ets/contracts/`, `entry/src/main/module.json5`, `AppScope/`, `oh-package.json5`, `build-profile.json5`, `hvigorfile.ts`, `AGENTS.md`, `.claude/` |
| any agent (append only) | `AI_WORKFLOW.md`: each PR adds its own entries (R2) |

**Shared files change only through a dedicated small PR.** The PR is labelled `contract` (for interfaces) or `chore` (for config), contains nothing else, and is merged as soon as it builds. The other side sees it at its next `/sync`.

**Mocks.** When an agent needs something the other side has not built yet, it codes against the contract with a clearly named mock (`Mock*` class, `// MOCK:` comment) and opens a `request`. Mocks must be replaced or labelled as mocks before the demo (R5).

## 6. The routine

Every step is written in plain text in `AGENTS.md`, so any agent can follow it. Claude Code also gets slash commands in `.claude/commands/` that run the same steps.

### `/sync` (start of every session and before every new task)
1. `git fetch origin`. Update your branch with `origin/main` (merge, never rebase a shared branch).
2. Print a short briefing:
   - open `request` issues for my role (handle these before new tasks)
   - my `status:in-progress` issue, if any
   - the other agents' `status:in-progress` issues and their open PRs, with the files those PRs touch
   - `contract` PRs that are open, or merged in the last 12 hours
   - anything `status:blocked`

### `/next-task`
1. Refuse if I already have a `status:in-progress` issue (one task per agent).
2. Find open issues with my role label, no assignee, no status label and not `agent:antigravity`. Take `P0` first, then the oldest. Skip any whose `Depends on` issues are still open.
3. Check again that the issue is unclaimed, then claim it: assign myself and add `status:in-progress`.
4. `git switch -c feat/<N>-<slug> origin/main` (`fix/` for bugs).
5. State the plan and the "Done when" checks before writing code.

### `/request`
1. `gh issue create` with labels `request` and the *other* role. The body says exactly what is needed (signature, data shape, behaviour), why, and which issue is waiting on it.
2. Keep working against a mock. Mark my own task `status:blocked` only if no mock is possible.

### `/ship` (safe to run again; it continues where it stopped)
1. Merge `origin/main` into my branch. Resolve conflicts only inside my own folders. If a conflict touches the other role's files, stop and report.
2. Verify: build with `hvigorw assembleHap` once the project exists, plus any tests. If verification cannot run, say so in the PR. Never claim it builds without running it (R5).
3. Add this PR's entry to `AI_WORKFLOW.md`: agent, main prompt, how the output was verified, plus any new tool, dependency or asset (R2, R3).
4. Push the branch and open a PR: `Closes #N`, files touched, contract changes (yes/no), how it was verified.
5. Merge with `gh pr merge --squash --delete-branch` only when:
   - the PR has no conflicts
   - verification passed
   - Prelint has posted and has no blocking finding

   If Prelint has not posted yet, report back and stop. The human runs `/ship` again later. Until Prelint is confirmed to review PRs on this repo, ask the human before merging.
6. Lead agents merge only their own PRs. Sedat's Claude also merges Antigravity's PRs after review. Never merge the other lead's PR.

### `/delegate <N>` (Sedat's Claude only)
1. The issue must be for Sedat's role, small and well defined, with an explicit list of the files Antigravity may touch.
2. Label it `agent:antigravity` and `status:in-progress`, and assign `sedatsan`.
3. Prepare Antigravity's worktree: `git worktree add ../Beltur-agy -b agy/<N>-<slug> origin/main` (or reuse the worktree with a fresh branch).
4. Run Antigravity headless inside that worktree:
   `agy -p "<prompt>" --mode accept-edits --sandbox`
   The prompt contains the issue goal, the Done-when checks, the file allowlist, the rule boundaries R1–R11, and these limits: commit only on `agy/<N>-<slug>`, never push, never use `gh`, never touch files outside the allowlist.
5. Review the whole diff (`git diff origin/main...agy/<N>-<slug>`). Reject any change outside the allowlist, any rule breach, and anything unverified.
6. If it is accepted, Sedat's Claude runs `/ship` for that branch: push, PR, merge, and an AI usage entry crediting Antigravity. If it is rejected twice, Sedat's Claude does the task itself.

## 7. Antigravity safety

- It runs only in `../Beltur-agy`, never in Sedat's main working tree.
- Always `--sandbox`, never `--dangerously-skip-permissions`. If a permission prompt blocks it in headless mode, that is a block (R9): Sedat's Claude reports it and does not raise Antigravity's permissions.
- No network writes. Antigravity does not push, open PRs, comment on issues or call `gh`. All GitHub writes go through a lead agent after review.
- Every Antigravity diff is reviewed in full before anything leaves the machine.
- Antigravity's Obsidian log (`X:\Vault\...`) stays as Sedat's personal log. GitHub is the shared record.
- If the sandbox stops Antigravity from committing, it leaves its changes uncommitted in the worktree, and Sedat's Claude commits them after review.
- Before the first real delegation, a read-only dry run (`--mode plan`) checks that Antigravity reads `AGENTS.md` and can restate the rule boundaries.

## 8. Repository changes

| File | Change |
|------|--------|
| `AGENTS.md` | Fix the stack section to API 20+. Add: Team & roles table, Rule boundaries (R1–R11), Coordination routine (section 6 in plain text), Antigravity limits. Mark the Obsidian log as personal. Keep the existing content. |
| `CLAUDE.md` | Replace the link with `@AGENTS.md` so Claude Code actually loads it. |
| `GEMINI.md` | Same, `@AGENTS.md`. |
| `.claude/commands/{sync,next-task,request,ship,delegate}.md` | Slash commands that run section 6. |
| `docs/challenge-rules.md` | Summary in our own words of the Huawei rules and the challenge brief: challenge areas, technical requirements, required deliverables, scoring weights and what the jury checks. |
| `AI_WORKFLOW.md` | Required by the brief. Tools, reusable instructions and config, workflow, review and validation, pre-existing work, third-party components, per-PR prompt and contribution log, limitations and lessons learned. |
| `.gitignore` | Add `RULES.pdf` and `CRITERIA*.pdf`. |
| `CONTRIBUTING.md`, `README.md` | Point to `AGENTS.md` for the agent workflow. Add a short "Development workflow" section to the README (transparency criterion). |
| GitHub | Create the labels from section 4. Extend the `main` ruleset to require a pull request (0 approvals) so R8 is enforced, not just written down. |

## 9. Verification

1. `gh label list` shows every label from section 4.
2. `gh api repos/sedatsan/Beltur/rules/branches/main` lists `pull_request`, `non_fast_forward` and `deletion`.
3. Claude Code loads the rules: `claude -p` in the repo, asked to restate R9, answers correctly.
4. Antigravity dry run in plan mode: it restates the rule boundaries and its limits, and makes no edits.
5. Use the routine on itself: this work is tracked as an issue, claimed with `/next-task` and shipped with `/ship`, and its PR closes the issue.

## 10. Out of scope

A task-board website, syncing with Obsidian, CI pipelines, size estimates, milestones.

## 11. Open items (do not block the build)

- Which human takes `app` and which takes `platform`.
- ~~The teammate's GitHub login~~: `yeet30`, has write access.
- The official submission deadline (added to `AGENTS.md` when announced).
- Confirm that the Prelint GitHub app reviews PRs on `sedatsan/Beltur`.
