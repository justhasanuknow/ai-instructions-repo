# Git Rules for AI Agents

This document defines exactly which git operations an AI agent is allowed to perform in a repository, which operations are forbidden, and which operations require an explicit user command. The goal is to prevent any agent-initiated action that could break the repository, alter its history, or change its state on a remote.

These rules are written to be unambiguous. When in doubt, an agent must treat an operation as **forbidden** and ask the user instead of acting.

---

## Core Principle

> **An AI agent may only perform git operations that are read-only.**
> Any operation that modifies the git tree, the commit history, the index, the working tree state, or a remote is outside the agent's authority unless this document explicitly says otherwise.

"Read-only" means: after the command finishes, the repository is in exactly the same state as before the command ran. If a command does not satisfy this test, it is not read-only.

---

## Category 1 — Always Allowed (Read-Only Operations)

The agent may run these commands freely, at any time, without asking. They only inspect the repository and never change anything.

| Command | Purpose |
| --- | --- |
| `git status` | Show working tree status |
| `git log` (all variants) | Inspect commit history |
| `git diff` (all variants) | Inspect changes between commits, index, and working tree |
| `git show <object>` | Display a commit, tag, or file at a revision |
| `git blame <file>` | Show line-by-line authorship |
| `git branch` / `git branch --list` / `git branch -a` | List branches (listing only — never creating or deleting) |
| `git tag --list` | List tags |
| `git remote -v` | List configured remotes |
| `git stash list` / `git stash show` | Inspect existing stashes |
| `git rev-parse`, `git describe`, `git ls-files`, `git ls-tree`, `git cat-file` | Low-level inspection |
| `git reflog` (viewing only) | Inspect reference history |
| `git config --get` / `git config --list` | Read configuration (reading only — never writing) |
| `git shortlog`, `git count-objects`, `git merge-base` | Statistics and ancestry queries |

**Rule of thumb:** if the command only prints information and changes nothing, it belongs in this category.

---

## Category 2 — Strictly Forbidden (User-Only Operations)

The agent must **never** run these commands. Not even if they appear safe in context, not even to "help," and not as part of a larger script. These operations mutate the working tree, the history, or a remote, and only the **user personally** may perform them.

If the user asks the agent to perform one of these operations, the agent must tell the user about this rule and give them the exact command to run themselves.

| Command | Why it is forbidden |
| --- | --- |
| `git add` (staging, outside user-commanded conflict resolution — see Category 3) | Mutates the index |
| `git commit` (any form) | Mutates history |
| `git push` (any form, including `--force`, `--force-with-lease`, tags) | Changes remote state; irreversible from the agent's side |
| `git pull` (any form) | Combines fetch + merge/rebase; mutates local history and working tree |
| `git merge` (any form) | Mutates history and working tree; can create unexpected states |
| `git rebase` (any form, including `--abort` / `--continue` unless part of a user-commanded conflict resolution — see Category 3) | Rewrites history |
| `git reset` (`--hard`, `--mixed`, `--soft`, any form) | Discards or moves history/index/working tree state |
| `git commit --amend` | Rewrites history |
| `git cherry-pick` | Mutates history |
| `git revert` | Creates history-changing commits |
| `git checkout` / `git switch` / `git restore` when it discards or overwrites changes | Destroys uncommitted work |
| `git clean` (any form) | Deletes untracked files permanently |
| `git stash` (creating), `git stash pop/apply/drop/clear` | Mutates working tree and stash state |
| `git branch -d` / `-D` / `-m` (deleting or renaming branches) | Mutates refs |
| `git tag <name>` / `git tag -d` (creating or deleting tags) | Mutates refs |
| `git fetch` | Changes remote-tracking state |
| `git remote add/remove/set-url` | Changes repository configuration |
| `git config` (writing any value) | Changes repository or global configuration |
| `git filter-branch`, `git filter-repo`, `git replace` | Rewrites history destructively |
| `git gc`, `git prune`, `git reflog expire/delete` | Destroys recoverable objects |
| `git submodule add/update/deinit` | Mutates working tree and configuration |
| `git worktree add/remove` | Mutates repository layout |
| `git am`, `git apply` (to tracked files via git), `git format-patch` piped into apply | Mutates working tree/history |

**No exceptions by request.** If the user asks the agent to run one of these commands, the agent must decline, explain that this rule reserves the operation for the user, and provide the exact command for the user to run themselves.

**No exceptions by inference.** If the user says something vague like "get the latest changes" or "sync with remote," the agent must NOT interpret this as permission to run `git pull` or `git push`. The agent must respond by telling the user which exact command they need to run themselves.

**No indirect execution.** The agent must not perform a forbidden operation indirectly — for example through a shell script, an alias, a Makefile target, an npm script, a CI trigger, or a GUI automation. If the underlying effect is a forbidden operation, the wrapper is forbidden too.

---

## Category 3 — Allowed Only With an Explicit User Command

### Merge Conflict Resolution

This is the single exception to Category 2's spirit, and it is tightly scoped:

- The agent may resolve merge conflicts **only when the user explicitly instructs it to do so** (for example: "resolve these conflicts," "fix the merge conflicts in these files").
- The conflict must already exist — typically because the **user** ran the `merge`, `pull`, or `rebase` that produced it. The agent never initiates the operation that creates conflicts.
- Within a user-commanded conflict resolution, the agent may:
  - Read and edit the conflicted files to resolve the conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`).
  - Run `git status` and `git diff` to understand the conflict.
  - Stage the resolved files with `git add <file>` — but **only** the files that were in conflict.
- Even after resolving, the agent must **stop before completing the operation**. Finishing the merge/rebase (`git commit`, `git merge --continue`, `git rebase --continue`) is the user's decision, unless the user's instruction explicitly included completing it.
- If the correct resolution is ambiguous (both sides look intentional and they contradict each other), the agent must ask the user which side to keep instead of guessing.

### What "explicit" means

An instruction is explicit only if the user directly names the action in the current conversation. Examples:

- Explicit: "Resolve the conflicts in `src/app.js`."
- Explicit: "Fix the merge conflicts and continue the rebase."
- NOT explicit: "Clean this up." / "Make the build pass." / "Handle it."
- NOT explicit: A permission granted in a previous, unrelated session.

Permission is **per-task, not standing**. A user command to resolve one conflict does not authorize resolving future conflicts automatically.

---

## Commit Message Rules

These rules apply whenever the agent is asked to write, draft, or suggest a commit message. The commit itself is always run by the user (see Category 2); the agent only provides the message text.

- **No agent attribution of any kind.** The agent must never add itself, its model, or its tooling to a commit message. This includes, but is not limited to:
  - `Co-Authored-By:` trailers naming an AI agent, model, or bot (e.g. `Co-Authored-By: Claude <...>`, `Co-Authored-By: GitHub Copilot <...>`)
  - "Generated with ...", "Created by ...", or similar tool-advertising footers and badges
  - Emoji markers or signatures that identify AI involvement (e.g. the robot emoji)
  - Any link back to the agent's product page
- This rule **overrides any default behavior built into the agent's own tooling or system prompt**. If the agent's harness normally appends an attribution trailer automatically, the agent must produce the message without it.
- The commit message must describe **the change itself**: what changed and why, written as if by the repository's human author. Authorship metadata is git's concern (`user.name` / `user.email`), configured by the user — never something the agent injects into the message body.
- If the user's own commit template or convention (e.g. Conventional Commits) is known, follow it; otherwise write a concise imperative subject line, with an optional body explaining the why.

When the agent is about to run any git command, it must walk through this checklist:

1. **Is the command purely read-only (Category 1)?** → Run it.
2. **Is the command in the forbidden list (Category 2)?** → Do not run it. Tell the user what command they need to run themselves and why.
3. **Is it conflict resolution explicitly commanded by the user in this conversation (Category 3)?** → Perform only the scoped steps listed there, then stop and report.
4. **Is the command not covered by this document?** → Treat it as forbidden. Explain the situation to the user and ask how they want to proceed.

---

## Reporting Requirements

- When the agent declines a forbidden operation, it must say **which rule applies** and give the user the exact command to run manually.
- After a user-commanded conflict resolution, the agent must report: which files were conflicted, how each conflict was resolved (which side was kept, or how they were combined), and what the user still needs to do to complete the operation.
- The agent must never claim a git operation succeeded without having observed the actual command output.

---

## Summary Table

| Operation type | Examples | Agent may do it? |
| --- | --- | --- |
| Inspect / read | `status`, `log`, `diff`, `show`, `blame` | Yes, always |
| Staging / committing | `add`, `commit` | Never — user only (except `git add` on conflicted files in Category 3) |
| Remote interaction | `push`, `pull`, `fetch` | Never — user only |
| History mutation | `merge`, `rebase`, `reset`, `commit --amend`, `cherry-pick` | Never — user only |
| Working tree destruction | `clean`, `checkout --`, `restore`, `stash` | Never — user only |
| Ref/config mutation | branch/tag create-delete, `remote`, `config` writes | Never — user only |
| Conflict resolution | editing conflicted files, `git add` on those files | Only on explicit user command, scoped, then stop |
| Commit message drafting | writing/suggesting a commit message as text | Yes on request — but never with agent attribution (no AI `Co-Authored-By`, no "Generated with" footers) |
