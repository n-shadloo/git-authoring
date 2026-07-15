# Commit conventions

Commit-message conventions for this repository. Applies to any agent or contributor writing a git commit here.

Turn staged changes into a commit message a reader will thank you for six months from now: read what is actually staged, work out the intent behind the change, and write a Conventional Commits message with an accurate type, a well-chosen scope, an imperative subject, and — when the change warrants it — a body that explains *why* plus footer trailers that carry metadata. Getting the shape right (`type(scope): subject`) is table stakes. The value is in choosing the right type, writing a subject that says what changed, and using the body to record the reasoning the diff itself can't show.

## Workflow

### 1. Gather context

Run these read-only commands (combine them into one call):

```bash
git branch --show-current
git status --short
git diff --staged --stat
git diff --staged
git log --oneline -15
```

- `git diff --staged` is the source of truth — read the actual hunks, not just the filenames.
- `git log --oneline -15` reveals the repo's prevailing style so you can match it (step 4).
- If `git rev-parse -q --verify MERGE_HEAD` succeeds, a merge is in progress — see "Merges and reverts" below.

**If nothing is staged**, say so plainly and stop. Do not run `git add -A` to conjure something to commit — staging everything silently bundles unrelated work into one commit, which is exactly what good history avoids. Show the unstaged changes (`git status --short`) and ask what should go into this commit.

### 2. Understand the change

Read the diff and answer, for yourself: *what* changed, and *why*. Intent is rarely visible in filenames — a change under `auth/` might be a bug fix, a new feature, a rename, or a security patch. Read the hunks. If the reasoning genuinely can't be inferred and it matters for the message, ask one focused question rather than guessing.

### 3. Decide: one commit, or several?

Commit one logical change at a time. Split when you see:

- Two or more unrelated concerns (a bug fix *and* an unrelated feature).
- A fix mixed with a refactor, or functional changes mixed with formatting-only churn.
- Changes that would need different types and share no cause.
- A subject you could only write with "and" in it.

If a split is warranted, don't force one message — propose the grouping and give a `git add <specific paths>` plus a commit for each group, smallest logical unit first:

```bash
# 1. The bug fix, on its own
git add users/serializers.py users/tests/test_serializers.py
git commit -m "fix(serializers): handle null profile in user payload"
# 2. The new endpoint
git add api/urls.py api/views.py api/tests/test_transactions.py
git commit -m "feat(api): add transactions list endpoint"
# 3. The dependency bump
git add pyproject.toml poetry.lock
git commit -m "build(deps): bump djangorestframework to 3.15.2"
```

Each commit is then a single revertible, reviewable idea. If one file holds two unrelated changes, stage selectively with `git add -p` and commit the hunks separately. If the change is cohesive, continue.

### 4. Detect the repository's convention

Default to Conventional Commits, but read `git log --oneline -15` first:

- Recent commits already look like `type(scope): subject` → carry on as normal.
- They follow a *different* consistent pattern → match that pattern. A convention is a property of the repository; a lone Conventional Commit in a history that doesn't use them is the thing that looks out of place.
- The history is inconsistent → default to Conventional Commits.

Conventions worth recognising: **Django** (`Fixed #31234 -- Added a system check for duplicated field names.` — ticket ref, ` -- `, past-tense description), **Angular** (Conventional Commits with a fixed package scope list), **Linux kernel** (subsystem prefix like `net: `, mandatory body, `Signed-off-by:` trailer, no `type():`). When you adapt to a non-default convention, note it in one line so the choice is visible.

### 5. Compose the message

1. **Type** — the accurate one (see "Types" below).
2. **Scope** — the affected area, if a clear one exists (see "Scopes" below).
3. **Subject** — imperative, concise, honest.
4. **Body** — only if it adds something the diff doesn't: the *why*, trade-offs, alternatives considered, side effects. Skip it for self-evident changes.
5. **Footers** — issue links (`Closes #123`), co-authors, `BREAKING CHANGE:` when applicable.

### 6. Self-check

- Type matches what the diff actually does.
- Subject is imperative and completes "If applied, this commit will …".
- Subject ≤ 50 characters if reasonable, ≤ 72 hard; no trailing period.
- A body exists if — and only if — the change needs explaining, and it says *why*, not *how*.
- Breaking changes are notated (`!` and/or a `BREAKING CHANGE:` footer).
- Trailers are well-formed.
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

- `type` is lowercase, from the set below. `feat` must be used for a new feature; `fix` must be used for a bug fix.
- `scope` is optional, in parentheses, naming the affected area: `feat(api):`.
- `!` before the colon marks a breaking change: `feat(api)!:`.
- The subject is imperative mood ("add", not "added"/"adds"), lowercase first letter, no period. Aim for ≤ 50 characters and treat 72 as the hard limit. (The spec allows any consistent casing; lowercase is the common default most linters expect.)
- **Say what changed, specifically.** "prevent redirect loop on expired token" carries the change; "fix login", "update code", "fix stuff", "wip" carry nothing.
- **Name the effect, not the edit.** "perf(api): speed up the export by batching rows" — not "change the loop to a comprehension", which describes the diff the reader can already see.

**Body**

- Separated from the subject by one blank line — not optional when a body exists; `git log`, `shortlog`, and `rebase` rely on it.
- Explains *what* and *why*, not *how* (the diff shows how). Free-form, may span paragraphs. Wrap at ~72 characters.
- Cover what the diff can't: why the change was needed, why this approach (alternatives rejected, trade-offs accepted), and consequences the next person should know.
- Include it when the change isn't self-explanatory; omit it when it is.

**Footers / trailers**

- One blank line after the body. Each is `Token: value` (or `Token #value`), tokens using `-` for spaces: `Reviewed-by`, `Co-authored-by`, `Refs`.
- Everything except the literal `BREAKING CHANGE` is case-insensitive to tools; `BREAKING CHANGE` must be uppercase (`BREAKING-CHANGE` is a synonym).

## Types

- **feat** — a new feature. → SemVer MINOR.
- **fix** — a bug fix. → SemVer PATCH.
- **docs** — documentation only: READMEs, docstrings, comments, guides.
- **style** — changes that don't affect meaning: whitespace, formatting, semicolons, import order. Not "CSS/visual styling".
- **refactor** — a code change that neither fixes a bug nor adds a feature: renames, extractions, restructuring with identical behaviour.
- **perf** — a performance improvement. → SemVer PATCH.
- **test** — adds missing tests or corrects existing ones.
- **build** — build system or external dependencies: bundler config, `pyproject.toml`, lockfiles. Dependency bumps are conventionally `build(deps):` — or `chore(deps):` in many repos, so match the repo.
- **ci** — CI configuration and scripts: GitHub Actions, GitLab CI, pipelines.
- **chore** — routine maintenance touching neither source nor tests: `.gitignore`, tooling config, housekeeping.
- **revert** — reverts a previous commit.

`docs`, `style`, `refactor`, `test`, `build`, `ci`, and `chore` trigger no release on their own. Any type combined with a breaking change is a SemVer MAJOR. Tools such as semantic-release and release-please read these to decide the version bump and generate the changelog — which is why an accurate type is worth the thought.

**Choosing between close types**

- **fix vs refactor** — did behaviour change for the user? A corrected defect is `fix`; code reshaped with identical behaviour is `refactor`.
- **feat vs fix** — is the capability new, or was it meant to work already? A newly added ability is `feat`; making an existing-but-broken ability work is `fix`.
- **refactor vs perf** — same output faster is `perf`; same output cleaner is `refactor`.
- **style vs refactor** — `style` never changes the parsed or compiled meaning. Renaming a variable is `refactor`, not `style`.
- **chore vs build** — dependency and build-tool changes are `build`; unrelated housekeeping is `chore`.

When two types could fit because the commit really does two things, that's usually a signal to split it (step 3).

## Scopes

A scope names the part of the codebase the change affects. It's optional — omit it when no single area fits or when the change is repo-wide.

- **Derive it from what changed**, not from a fixed menu: changes under `payments/` → `payments`.
- **One clear scope beats several.** If a change spans many areas, either it's genuinely cross-cutting (drop the scope) or it should be split.
- **Keep scopes stable and coarse.** Reuse the handful the repo already uses (visible in `git log`) rather than inventing a new one per commit. `auth` is a better scope than `login-form-validation`.
- **Match existing casing and vocabulary.** If the repo writes `feat(API)` or `feat(ui)`, follow suit.

## Breaking changes

A breaking change correlates with a SemVer MAJOR bump and can accompany any type. Signal it with `!` before the colon, a `BREAKING CHANGE:` footer, or both. If you use `!` without a footer, the subject itself must describe the break. Prefer adding the footer whenever a migration note would help the reader.

```text
feat(api)!: return ISO 8601 timestamps

BREAKING CHANGE: `created` is now an ISO 8601 string, not a Unix epoch
integer. Clients parsing it as a number must update.
```

Backends with external consumers (a mobile or web client hitting your API) should treat any incompatible change to a request or response shape as breaking, and spell out the migration in the footer.

## Trailers

Only add trailers that are true. An invented reviewer or co-author is worse than none.

- `Closes #123` / `Fixes #123` / `Resolves #123` — GitHub and GitLab close the issue when the commit merges into the default branch; on other branches it's referenced only.
- `Refs #123` — references without closing.
- `Co-authored-by: Full Name <email>` — a genuine collaborator on the change (pairing, mob, applying someone's patch); appears in the contributors graph.
- `Reported-by:` / `Suggested-by:` / `Helped-by:` — credit for the report, the idea, or debugging help.
- `Reviewed-by:` / `Acked-by:` / `Tested-by:` — review, agreement, and testing sign-off.
- `Signed-off-by: Full Name <email>` — a Developer Certificate of Origin sign-off, required by projects such as the Linux kernel. Add it with `git commit -s`.
- `BREAKING CHANGE: <what breaks and the migration>` — uppercase, exactly.

## Merges and reverts

Merge commits are usually left in git's default form and are typically excluded from changelog tooling, so you don't normally write one by hand. If a merge is in progress and a message is wanted, keep git's default unless asked otherwise.

For a revert, use the `revert` type, keeping the original subject, with a footer identifying the reverted commit (`revert: feat(orders): add idempotency keys` + `Refs: 676104e`). `git revert` generates a default message ("Revert \"…\""); reshape it to the convention.

## Examples

Self-explanatory change — one line is the whole message, since a body would just restate the diff:

```text
fix(auth): prevent redirect loop on expired token
```

Feature with rationale — the body captures the reasoning `git blame` can't reconstruct:

```text
feat(orders): add idempotency keys to checkout

Client retries on a flaky connection were submitting checkout twice and
creating duplicate orders. Callers now send an Idempotency-Key header;
a repeated key returns the original order instead of creating a new one.

Keys are stored for 24h, which covers realistic retry windows without
growing the table unbounded.

Closes #482
```

---

Optional deeper reading — not needed to write a correct commit: `references/` in this repository holds the full Conventional Commits grammar (`conventional-commits.md`), an annotated example gallery with worked splits (`examples.md`), message craft (`craft.md`), and scope/convention guidance (`scopes-and-repos.md`).
