---
name: git-commit-writer
description: Writes Conventional Commits messages from staged git changes. Reads the staged diff, infers what changed and why, chooses an accurate type and scope, and produces a properly structured commit — subject, body, and footer trailers — with correct breaking-change notation. Use whenever the user is about to commit, asks for a commit message, mentions writing/fixing/improving a commit, refers to their staged changes, or says things like "commit this", "what should this commit say", or "write a commit message". Also use before running git commit so the message follows the convention. Do not trigger for unrelated git work such as branching, rebasing, or resolving merge conflicts unless a commit message is being written.
license: MIT
compatibility: Requires git and a Git repository. Language-agnostic; no runtime dependencies beyond git.
metadata:
  author: n-shadloo
  version: "1.1.0"
allowed-tools: Bash(git:*) Read
---

# Git Commit Writer

Turn staged changes into a commit message a reader will thank you for six months from now. This skill reads what is actually staged, works out the intent behind the change, and writes a Conventional Commits message with an accurate type, a well-chosen scope, an imperative subject, and — when the change warrants it — a body that explains *why* and footer trailers that carry metadata.

The point is not a message that merely passes a linter. Getting the shape right (`type(scope): subject`) is the easy part, and this skill treats it as table stakes. The value is in choosing the right type, writing a subject that says what changed, and using the body to record the reasoning the diff itself can't show.

## What this skill does

- Reads the staged diff and describes the change honestly — what changed, and why it likely changed.
- Picks the correct Conventional Commit **type** and an accurate **scope**.
- Writes an imperative **subject** within length limits, a **body** that explains motivation and trade-offs when they aren't obvious, and **footer trailers** (issue links, co-authors, breaking-change notes).
- Flags **breaking changes** and notates them correctly (`!` and/or a `BREAKING CHANGE:` footer).
- Decides when staged changes should be **split into several commits**, and proposes the split.
- Matches the repository's **existing convention** when it differs from Conventional Commits.
- Commits the message when asked — never before.

## Workflow

Work through these in order. For a small, single-purpose change this file is enough on its own; reach for a reference file only when a step points to one.

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

**If nothing is staged**, say so plainly and stop. Do not run `git add -A` to conjure something to commit — staging everything silently bundles unrelated work into one commit, which is exactly what good history avoids. Show the unstaged changes (`git status --short`) and ask what should go into this commit.

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
5. **Footers** — issue links (`Closes #123`), co-authors, `BREAKING CHANGE:` when applicable.

### 6. Self-check

Before presenting, confirm:

- Type matches what the diff actually does.
- Subject is imperative and completes "If applied, this commit will …".
- Subject ≤ 50 characters if reasonable, ≤ 72 hard; no trailing period.
- A body exists if — and only if — the change needs explaining, and it says *why*, not *how*.
- Breaking changes are notated (`!` and/or a `BREAKING CHANGE:` footer).
- Trailers are well-formed (`references/conventional-commits.md`).
- Nothing is invented — every claim is backed by the diff.

### 7. Present, and commit only when asked

Show the message. Commit only on an explicit request ("commit it", "go ahead"). When committing, commit the **staged** set as-is; never re-stage silently. Use a method that preserves the multi-line message exactly:

```bash
git commit -F - <<'COMMIT_MSG'
type(scope): subject

Body paragraph explaining why.

Closes #123
COMMIT_MSG
```

The quoted heredoc delimiter (`'COMMIT_MSG'`) stops the shell from expanding backticks or `$` inside the message. For a subject-only commit, `git commit -m "type(scope): subject"` is fine.

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

For a fuller gallery — breaking changes, multi-paragraph bodies, co-authored commits, and worked splits — see `references/examples.md`.

## Reference files

Read these on demand; don't load them for routine commits.

- **`references/conventional-commits.md`** — the full grammar, complete type taxonomy with SemVer impact, breaking-change rules, the footer/trailer catalog, and revert/merge handling. Read when unsure about a type, breaking-change formatting, or trailer syntax.
- **`references/examples.md`** — an annotated gallery of exemplary commits, including breaking changes, rationale-heavy bodies, trailers, and worked splits. Read when composing a non-trivial message or a split.
- **`references/craft.md`** — how to write a subject and body that communicate, the *why*-not-*how* principle, and a catalogue of common bad commits with fixes. Read when lifting a mechanical message.
- **`references/scopes-and-repos.md`** — choosing a scope (with backend/Django/DRF and other ecosystem cues), and detecting and matching a repository's existing convention. Read when picking a scope or entering an unfamiliar repo.
