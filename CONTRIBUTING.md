# How we work

We're two people with limited time, so the rules are short.

## Branches

- `main` should always build and be ready to demo. No force-pushes or deletes (GitHub blocks both).
- Do your work on a short-lived branch:
  - `feat/<short-name>`: new feature
  - `fix/<short-name>`: bug fix
  - `docs/<short-name>`: README, pitch, screenshots
  - `chore/<short-name>`: config, tooling, cleanup

## Daily flow

```bash
git switch main
git pull
git switch -c feat/my-thing
# ...work, commit often...
git push -u origin feat/my-thing
gh pr create --fill
```

- Open a PR into `main`. Ask the other person to look at it if they're free. If they're busy, merging your own PR is fine.
- We squash-merge, so each PR becomes one commit on `main`. Merged branches are deleted automatically.
- If you're both editing the same files, say so in chat before you start.

## Working from a fork

If you don't have write access to `sedatsan/Beltur`, work from your own fork instead.
Set `origin` to your fork and `upstream` to the main repo:

```bash
git remote add upstream https://github.com/sedatsan/Beltur.git
```

Then the daily flow becomes:

```bash
git switch main
git pull upstream main
git push origin main
git switch -c feat/my-thing
# ...work, commit often...
git push -u origin feat/my-thing
gh pr create --repo sedatsan/Beltur --base main --fill
```

## Prelint

[Prelint](https://prelint.com) reviews every PR against our specs and decisions.
It reads the Markdown files in the repo, so put product specs and decisions in `docs/`.

The repo's `.mcp.json` connects Claude Code to Prelint's MCP server, so the agent can
check our decisions while it writes code. The first time you open the project in Claude Code,
approve the `prelint` server, then run `/mcp` → **prelint** → sign in with your own Prelint account.

## Commits

Write short, imperative messages: `Add login screen`, `Fix crash on rotate`.
A prefix is optional but helps: `feat:`, `fix:`, `docs:`, `chore:`.

## Never commit

- Signing material (`*.p12`, `*.p7b`, `*.cer`, `*.csr`) or signing passwords.
- API keys or `.env` files.
- Build output (`build/`, `oh_modules/`, `.hvigor/`). `.gitignore` already excludes it.

## Language

Code, comments, commits, PRs and docs are all in **English**. The final submission must be in English.
