---
name: git-authoring
description: Authors Conventional Commits messages and pull-request content from real git changes, or carries out a complete stage-commit-push workflow when explicitly told to do it all. Supports four request-selected modes: write an exact commit command for already-staged changes; choose one coherent set of unstaged files and present staging plus commit commands; write a pull-request title and structured Markdown description from the branch diff; or, only on an unmistakable autonomous request, stage, commit, and push the work itself. Use whenever the user is about to commit, asks for a commit message or commit command, mentions writing/fixing/improving a commit, refers to staged changes, asks which files belong in a commit, asks for a pull-request title or description, or explicitly asks the agent to stage, commit, and push. Do not trigger for unrelated git work such as branching, rebasing, or resolving merge conflicts unless a commit or pull request is being written.
license: MIT
compatibility: Requires git and a Git repository. Language-agnostic; no runtime dependencies beyond git. Pull-request output is plain Markdown, so no GitHub CLI is required.
metadata:
  author: n-shadloo
  version: "2.0.1"
allowed-tools: Bash(git:*) Read
---

# Git Authoring

Turn real git changes into history a reader will thank you for six months from now. By default this skill reads what is actually staged, works out the intent behind the change, and presents an exact commit command with a Conventional Commits message: an accurate type, a well-chosen scope, an imperative subject, and — when the change warrants it — a body that explains *why* and footer trailers that carry metadata. On request, it can instead choose one coherent set of files and present staging plus commit commands, write complete pull-request content, or carry out staging, committing, and pushing itself.

The point is not output that merely passes a linter. Getting the shape right (`type(scope): subject`) is the easy part, and this skill treats it as table stakes. The value is in choosing the right type, writing a subject that says what changed, using the body to record the reasoning the diff itself can't show, and — for a pull request — giving a reviewer a title and description they can act on.

## What this skill does

- Reads the staged diff and describes the change honestly — what changed, and why it likely changed.
- Picks the correct Conventional Commit **type** and an accurate **scope**.
- Writes an imperative **subject** within length limits, a **body** that explains motivation and trade-offs when they aren't obvious, and **footer trailers** (issue links, co-authors, breaking-change notes).
- Flags **breaking changes** and notates them correctly (`!` and/or a `BREAKING CHANGE:` footer).
- Decides when staged changes should be **split into several commits**, and proposes the split.
- On request, **chooses which unstaged files** belong together as one coherent commit, then presents exact staging and commit commands.
- On request, writes a complete **pull-request title and description** — summary, what changed, testing, breaking changes — from the branch's history and its diff against the base branch.
- Matches the repository's **existing convention** when it differs from Conventional Commits.
- Keeps commit-message, file-selection, and pull-request modes **read-only**: it presents commands or Markdown and leaves execution to the user.
- On an explicit autonomous request only, **stages, commits, and pushes** the selected work end to end.

## Choose the mode

Choose exactly one mode from the user's request. Do not blend their execution rules.

1. **Commit command for already-staged changes (default).** Inspect the staged diff, write the message, and present the exact heredoc `git commit` command for the user to run.
2. **Choose files, then provide commands (on request).** Inspect staged and unstaged work, select one coherent set, and present the exact `git add` command(s) followed by the heredoc commit command for the user to run.
3. **Pull-request title and description (on request).** Inspect the branch against its base and produce the title plus structured Markdown description. Do not stage, commit, push, or open the PR.
4. **Autonomous stage, commit, and push (explicit request only).** Run the complete workflow yourself only when the user unmistakably asks you to carry out the git operations — for example, "stage, commit, and push this for me" or "do it all yourself."

A request for a message, commands, file selection, or PR content selects modes 1–3. A bare "commit this," "go ahead," or confirmation after you present commands does **not** silently switch to mode 4. If execution intent is ambiguous, stay read-only and ask for an explicit autonomous request before mutating git.

## Ground rules

The mode boundary is a hard guarantee.

- **Modes 1–3 never mutate git.** Run only read-only inspection. Never stage, commit, push, or open a pull request in these modes, even after a follow-up confirmation. Present exact commands for the user to run.
- **Modes 1–3 never add AI attribution.** Commits and pull requests read as the developer's own work. Never add an AI co-author, a `Co-authored-by` for the agent, a "Generated by"/"written with" line, or an AI author or committer identity.
- **Mode 4 is the sole execution exception.** Once explicitly selected, it may stage, commit, and push. AI attribution is permitted in this mode but not required; never invent a human co-author, reviewer, sign-off, or other trailer.
- **Never infer mode 4.** Do not treat ordinary commit wording or approval of proposed commands as permission to execute. The user must clearly ask the agent to perform the operations itself.

## Workflow

This is mode 1, the default path: writing a commit command for what is already staged. Work through the steps in order; for a small, single-purpose change this file is enough on its own, so reach for a reference file only when a step points to one. The other three modes activate only when the user asks for them and each has its own section after this workflow.

### 1. Gather context

Run these read-only commands (combine them into one call). Nothing is staged here.

```bash
git branch --show-current
git status --short
git diff --staged --stat
git diff --staged
git log --oneline -15
```

- `git diff --staged` is the source of truth — read the actual hunks, not just the filenames.
- `git log --oneline -15` reveals the repo's prevailing style so you can match it (step 4).
- If `git rev-parse -q --verify MERGE_HEAD` succeeds, a merge is in progress — see `references/conventional-commits.md` for merge and revert handling.

**If nothing is staged**, say so plainly and stop. Do not run `git add -A` to conjure something to commit — staging everything silently bundles unrelated work into one commit, which is exactly what good history avoids. Show the unstaged changes (`git status --short`) and ask what should go into this commit. (If the user has asked you to choose what to stage, follow "Choosing which files to stage" below instead.)

### 2. Understand the change

Read the diff and answer, for yourself: *what* changed, and *why*. Intent is rarely visible in filenames — a change under `auth/` might be a bug fix, a new feature, a rename, or a security patch. Read the hunks. If the reasoning genuinely can't be inferred and it matters for the message, ask one focused question rather than guessing.

### 3. Decide: one commit, or several?

Before writing anything, judge whether the staged changes belong together. Commit one logical change at a time. Split when you see:

- Two or more unrelated concerns (a bug fix *and* an unrelated feature).
- A fix mixed with a refactor, or functional changes mixed with formatting-only churn.
- Changes that would need different types and share no cause.

If a split is warranted, don't write one message — propose the grouping and give the sequence of `git add <specific paths>` plus a commit for each group, smallest logical unit first. See `references/examples.md` for worked splits. If the change is cohesive, continue.

### 4. Detect the repository's convention

Default to Conventional Commits. But look at `git log --oneline -15` first: if the repo consistently uses a *different* convention, match it rather than imposing Conventional Commits on a history that doesn't use it. Common cases — Django's `Fixed #nnnn -- Description`, Angular's fixed scope list, the kernel's `Signed-off-by` — are covered in `references/scopes-and-repos.md`. When you adapt to a non-default convention, note it in one line so the choice is visible.

### 5. Compose the message

Build it in this order. See "The format" below for the rules, `references/craft.md` to lift a mechanical message, and `references/conventional-commits.md` for the full type taxonomy and trailer catalog.

1. **Type** — the accurate one (feat, fix, refactor, …).
2. **Scope** — the affected area, if a clear one exists (`references/scopes-and-repos.md`).
3. **Subject** — imperative, concise, honest.
4. **Body** — only if it adds something the diff doesn't: the *why*, trade-offs, alternatives considered, side effects. Skip it for self-evident changes.
5. **Footers** — issue links (`Closes #123`), genuine co-authors, `BREAKING CHANGE:` when applicable.

### 6. Self-check

Before presenting, confirm:

- Type matches what the diff actually does.
- Subject is imperative and completes "If applied, this commit will …".
- Subject ≤ 50 characters if reasonable, ≤ 72 hard; no trailing period.
- A body exists if — and only if — the change needs explaining, and it says *why*, not *how*.
- Breaking changes are notated (`!` and/or a `BREAKING CHANGE:` footer).
- Trailers are well-formed (`references/conventional-commits.md`) and true — no AI attribution.
- Nothing is invented — every claim is backed by the diff.

### 7. Present the exact commit command

Present the exact command for the user to run. Do not execute it. The command commits the **staged** set as-is; never add a staging command silently in mode 1. Always use the quoted-heredoc form, including for a subject-only message, so every emitted commit command has one consistent, expansion-safe shape:

```bash
git commit -F - <<'COMMIT_MSG'
type(scope): subject

Body paragraph explaining why.

Closes #123
COMMIT_MSG
```

The quoted heredoc delimiter (`'COMMIT_MSG'`) stops the shell from expanding backticks or `$` inside the message.

## Mode 2: Choose which files to stage (only when asked)

This runs **only when the user asks the agent to pick what belongs in one commit** — "choose the files that belong together and give me the commands," "prepare staging and commit commands for the right files," and the like. When the user hasn't asked, the default is unchanged: if nothing is staged, say so and stop (step 1).

1. **Inspect the unstaged work.** Read the real changes, not filenames: `git status --short`, `git diff --stat`, and `git diff` (add `git diff --staged` if some things are already staged).
2. **Group by intent, then select one coherent set.** Choose the files — or, when a file mixes concerns, the hunks — that form a single self-contained change, the way a careful developer would group them: related changes only. Do **not** stage everything at once, and do **not** stage an arbitrary mix. If the working tree holds several unrelated changes, pick the single most coherent group to commit first and name the remaining groups as the next commits, rather than bundling them.
3. **Present the exact staging, then the commit.** Give `git add <specific paths>` for the chosen set (or `git add -p <path>` when one file needs splitting), then compose the message with the normal discipline (steps 4–6) and present the quoted-heredoc commit command.
4. **Leave execution to the user.** Never run `git add`, `git commit`, or `git push` in mode 2, including after the user confirms the proposed grouping.

A worked selection is in `references/examples.md`.

## Mode 3: Pull requests (only when asked)

This runs **only when the user asks for pull-request help** — "write a PR title and description", "draft the PR for this branch", and the like. It never fires as part of a commit request. `references/pull-requests.md` carries the full guidance; the essentials:

1. **Find the base branch automatically.** Prefer `git symbolic-ref refs/remotes/origin/HEAD` (strip to the branch name); if that isn't set, fall back to whichever of `origin/main` or `origin/master` exists, then to local `main`/`master`. State the base you settled on.
2. **Read what the branch actually did** against that base (all read-only):

   ```bash
   git log <base>..HEAD --oneline
   git diff <base>...HEAD --stat
   git diff <base>...HEAD
   ```

   The three-dot `<base>...HEAD` diffs from the merge base, so it shows only this branch's work.
3. **Write the PR as plain Markdown** — a strong, specific title, then a description with **Summary** (why this branch exists), **What changed** (the substantive changes grouped by intent, not a file dump), **Testing** (how it was verified — from tests in the diff, CI, or manual steps; say what you could and couldn't confirm), and **Breaking changes** (spell out any contract change and its migration, or state "None"). Add a short reviewer/testing checklist when it earns its place. Tailor the emphasis to the PR's type (feature, fix, refactor, docs, breaking).
4. **Ground every claim in the history and diff.** Don't assert tests passed if the branch adds none — say testing is unverified instead. Verify as thoroughly as the branch allows.
5. **Never open the PR yourself.** Output the Markdown for the user to paste, or offer the exact command for them to run — for example `gh pr create --base <base> --title "…" --body-file <file>` — and leave running it to them. A follow-up "go ahead" does not change mode 3 into an execution mode.

## Mode 4: Autonomous stage, commit, and push

Enter this mode only when the user explicitly asks the agent to perform the complete operation itself. This is a deliberate exception to the read-only rules above; never activate it from a request that merely asks for a message, commands, file selection, or PR content.

1. **Inspect before mutating.** Read the current branch, status, staged and unstaged diffs, recent history, remotes, and upstream. Use the staged and unstaged hunks — not filenames alone — to understand the work. Check which commits, if any, are already ahead of the upstream because a normal push will publish them too.
2. **Select one coherent commit.** Preserve intentional staged work. Add only specific related paths or hunks needed to complete that commit; never use `git add -A` or sweep unrelated changes into it. If existing staged changes conflict with a coherent grouping, or a path requires hunk selection that cannot be performed reliably, stop and ask one focused question rather than rewriting the user's index blindly.
3. **Stage and verify.** Run the specific `git add <paths>` commands (or carefully use `git add -p` when interactive selection is available), then reread `git diff --staged --stat` and `git diff --staged`. Do not commit until the staged diff is the intended single change.
4. **Commit with the quoted heredoc.** Compose and execute the message using exactly this form, even for a subject-only commit:

   ```bash
   git commit -F - <<'COMMIT_MSG'
   type(scope): subject

   Optional body explaining why.

   Optional-Trailer: value
   COMMIT_MSG
   ```

   Do not bypass hooks, rewrite an existing commit, or amend unless the user explicitly requested that separate action.
5. **Push safely.** Push the current branch normally to its configured upstream. If no upstream exists and `origin` is the unambiguous intended remote, use `git push -u origin HEAD`. Never force-push. If the remote or destination is ambiguous, or the push is rejected, stop and report the exact state instead of guessing or rewriting history.
6. **Report the result.** State the committed SHA and subject, the pushed branch and remote, and any work intentionally left unstaged. If staging or committing succeeds but pushing fails, say so plainly and do not claim completion.

AI attribution is allowed only in this explicitly selected mode. It is optional, not automatic; add it only when appropriate to the user's stated preference or the operating environment. All other trailers must still be true.

## The format

```text
<type>[optional scope][!]: <subject>
<blank line>
[optional body]
<blank line>
[optional footer(s)]
```

**Subject line**

- `type` is lowercase, from the set below.
- `scope` is optional, in parentheses, naming the affected area: `feat(api):`.
- `!` before the colon marks a breaking change: `feat(api)!:`.
- The subject is imperative mood ("add", not "added"/"adds"), lowercase first letter, no period. Aim for ≤ 50 characters and treat 72 as the hard limit. (The Conventional Commits spec allows any consistent casing; lowercase is the common default most linters expect.)

**Body**

- Separated from the subject by one blank line — not optional when a body exists; `git log`, `shortlog`, and `rebase` rely on it.
- Explains *what* and *why*, not *how* (the diff shows how). Wrap at ~72 characters.
- Include it when the change isn't self-explanatory; omit it when it is.

**Footers / trailers**

- One blank line after the body. Each is `Token: value` (or `Token #value`), tokens using `-` for spaces: `Reviewed-by`, `Co-authored-by`, `Refs`.
- Issue links: `Closes #123` / `Fixes #123` (GitHub closes the issue on merge to the default branch) or `Refs #123` (reference only).
- Breaking change: `BREAKING CHANGE: <what breaks and the migration>` (uppercase, exactly).
- Only add trailers that are true. In modes 1–3, never add AI attribution. In mode 4, AI attribution is permitted but optional.

## Types (quick reference)

- **feat** — a new feature (SemVer MINOR).
- **fix** — a bug fix (SemVer PATCH).
- **docs** — documentation only.
- **style** — formatting/whitespace, no behaviour change.
- **refactor** — a code change that neither fixes a bug nor adds a feature.
- **perf** — a performance improvement (SemVer PATCH).
- **test** — adding or correcting tests.
- **build** — build system or dependencies (e.g. `build(deps):`).
- **ci** — CI configuration and scripts.
- **chore** — maintenance that touches neither source nor tests.
- **revert** — reverts a previous commit.

Any type combined with a breaking change is a SemVer MAJOR. For precise "use when" guidance and less common types, see `references/conventional-commits.md`.

## Breaking changes

Notate a breaking change in one of two ways (either is valid; use the footer when a migration note helps):

```text
feat(api)!: return ISO 8601 timestamps

BREAKING CHANGE: `created` is now an ISO 8601 string, not a Unix epoch
integer. Clients parsing it as a number must update.
```

Backends with external consumers (a mobile or web client hitting your API) should treat any incompatible change to a request or response shape as breaking, and spell out the migration in the footer.

## A few examples

**Simple fix, no body needed:**

```text
fix(auth): prevent redirect loop on expired token
```

**Feature with rationale:**

```text
feat(orders): add idempotency keys to checkout

Duplicate submissions from client retries were creating double orders.
Callers now pass an Idempotency-Key header; a repeated key returns the
original result instead of creating a new order.

Closes #482
```

**Maintenance:**

```text
chore(deps): bump djangorestframework to 3.15.2
```

For a fuller gallery — breaking changes, multi-paragraph bodies, co-authored commits, worked splits, and a file-selection walkthrough — see `references/examples.md`.

## Reference files

Read these on demand; don't load them for routine commits.

- **`references/conventional-commits.md`** — the full grammar, complete type taxonomy with SemVer impact, breaking-change rules, the footer/trailer catalog, and revert/merge handling. Read when unsure about a type, breaking-change formatting, or trailer syntax.
- **`references/examples.md`** — an annotated gallery of exemplary commits, including breaking changes, rationale-heavy bodies, trailers, worked splits, and a file-selection walkthrough. Read when composing a non-trivial message, a split, or a staging selection.
- **`references/craft.md`** — how to write a subject and body that communicate, the *why*-not-*how* principle, and a catalogue of common bad commits with fixes. Read when lifting a mechanical message.
- **`references/scopes-and-repos.md`** — choosing a scope (with backend/Django/DRF and other ecosystem cues), and detecting and matching a repository's existing convention. Read when picking a scope or entering an unfamiliar repo.
- **`references/pull-requests.md`** — base-branch detection, gathering the branch's history and diff, and writing a PR title and a structured description (summary, changes, testing, breaking changes) with type-aware emphasis and a reviewer checklist. Read when writing pull-request content.
