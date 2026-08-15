---
name: git-authoring
description: >-
  Authors Conventional Commits messages, pull-request content, and release
  notes from real git changes, reviews an incoming pull request for the
  maintainer, or runs the stage-commit-push workflow when told to. Six
  modes: commit command for staged changes; file selection plus commands;
  pull-request title and description; autonomous stage, commit, and push; a
  version's release note; and a review comment or squash-merge message to
  paste. Use whenever the user is about to commit, asks for a commit message
  or command, mentions writing or improving a commit, refers to staged
  changes, asks which files belong in a commit, asks for a pull-request
  title or description, asks for release notes or a changelog entry, asks
  the agent to stage, commit, and push, or asks for help reviewing,
  approving, rejecting, or merging someone else's pull request. Do not
  trigger for unrelated git work such as branching, rebasing, or resolving
  merge conflicts unless a commit, pull request, review, or release note is
  being written.
license: MIT
compatibility: >-
  Requires git and a Git repository. Language-agnostic; no runtime
  dependencies beyond git. Pull-request and release-note output is plain
  Markdown, so GitHub CLI is optional for modes 1–5 — there it only detects
  prior release style and, on explicit request in mode 4, publishes a
  release. Mode 6 requires an authenticated `gh` to read the pull request,
  but still only reads: it writes Markdown for the user to paste.
metadata:
  author: n-shadloo
  version: "2.4.0"
allowed-tools: Bash(git:*) Bash(gh:*) Read
---

# Git Authoring

Turn real git changes into history a reader will thank you for six months from now. By default this skill reads what is actually staged, works out the intent behind the change, and presents an exact commit command with a Conventional Commits message: an accurate type, a well-chosen scope, an imperative subject, and — when the change warrants it — a body that explains *why* and footer trailers that carry metadata. On request, it can instead choose one coherent set of files and present staging plus commit commands, write complete pull-request content, write the release note for a version, review an incoming pull request with you and hand you the text to paste, or carry out staging, committing, and pushing itself.

The point is not output that merely passes a linter. Getting the shape right (`type(scope): subject`) is the easy part, and this skill treats it as table stakes. The value is in choosing the right type, writing a subject that says what changed, using the body to record the reasoning the diff itself can't show, and — for a pull request or a release note — giving the reader something they can act on.

## What this skill does

- Reads the staged diff and describes the change honestly — what changed, and why it likely changed.
- Picks the correct Conventional Commit **type** and an accurate **scope**.
- Writes an imperative **subject** within length limits, a **body** that explains motivation and trade-offs when they aren't obvious, and **footer trailers** (issue links, breaking-change notes).
- Adds **no attribution trailers by default**, in any mode — co-author, sign-off, review, and AI or agent identity are opt-in only.
- Flags **breaking changes** and notates them correctly (`!` and/or a `BREAKING CHANGE:` footer).
- Decides when staged changes should be **split into several commits**, and proposes the split.
- On request, **chooses which unstaged files** belong together as one coherent commit, then presents exact staging and commit commands.
- On request, writes a complete **pull-request title and description** — summary, what changed, testing, breaking changes — from the branch's history and its diff against the base branch.
- On request, writes the **release note** for a version from the real range since the last release, matching the project's own release style.
- Matches the repository's **existing convention** when it differs from Conventional Commits.
- On request, **reviews an incoming pull request** — reads the diff, checks, and existing comments through `gh`, separates what actually blocks a merge from what is merely a suggestion, works the decision through with the maintainer, and produces the review comment or the squash-merge message as Markdown to paste.
- Keeps commit-message, file-selection, pull-request, release-note, and review modes **read-only**: it presents commands or Markdown and leaves execution to the user.
- On an explicit autonomous request only, **stages, commits, and pushes** the selected work end to end — and, on a further explicit request, tags and publishes a release.

## Choose the mode

Choose exactly one mode from the user's request. Do not blend their execution rules.

1. **Commit command for already-staged changes (default).** Inspect the staged diff, write the message, and present the exact heredoc `git commit` command for the user to run.
2. **Choose files, then provide commands (on request).** Inspect staged and unstaged work, select one coherent set, and present the exact `git add` command(s) followed by the heredoc commit command for the user to run.
3. **Pull-request title and description (on request).** Inspect the branch against its base and produce the title plus structured Markdown description. Do not stage, commit, push, or open the PR.
4. **Autonomous stage, commit, and push (explicit request only).** Run the complete workflow yourself only when the user unmistakably asks you to carry out the git operations — for example, "stage, commit, and push this for me" or "do it all yourself." On a *further* explicit request, this mode may also tag and publish a release.
5. **Release note for a version (on request).** Establish the range since the last release, read what actually landed, and produce the note as a Markdown block. Do not tag, publish, or write a file.
6. **Review an incoming pull request (on request).** Read someone else's PR through `gh`, work the decision through with the user, and produce the review comment or the squash-merge message as a Markdown block for them to paste into GitHub. Do not approve, request changes, comment, or merge.

A request for a message, commands, file selection, PR content, release notes, or a review selects modes 1–3, 5, and 6. A bare "commit this," "go ahead," or confirmation after you present commands or a note does **not** silently switch to mode 4. If execution intent is ambiguous, stay read-only and ask for an explicit autonomous request before mutating git.

## Ground rules

The mode boundary is a hard guarantee.

- **Modes 1–3, 5, and 6 never mutate git or GitHub.** Run only read-only inspection. Never stage, commit, push, open a pull request, tag, publish a release, or approve, request changes on, comment on, or merge a pull request in these modes, even after a follow-up confirmation. Present exact commands or Markdown for the user to run.
- **No attribution trailers by default — in every mode, mode 4 included.** No `Co-authored-by:`, no `Signed-off-by:`, no `Reviewed-by:`, no "Generated by"/"written with" line, and no AI or agent identity, in commits or in pull-request descriptions. The commit author is whatever `git config user.name` / `user.email` resolves to. Never look up, infer, or attribute the work to anyone else, and never invent a name or an email address. Trailers are added only through the opt-in in "Footers / trailers" below.
- **Mode 4 is the sole execution exception.** Once explicitly selected, it may stage, commit, and push — and, on a separate explicit request, tag and publish a release. It is an exception to the read-only rule and to nothing else — the attribution default above applies to it unchanged.
- **Never infer mode 4.** Do not treat ordinary commit wording or approval of proposed commands as permission to execute. The user must clearly ask the agent to perform the operations itself.
- **Never infer publishing.** Producing a release note in mode 5, or the user approving one, is not permission to tag or publish. Publishing takes mode 4 *and* an explicit request to publish, together.
- **Mode 6 never writes to GitHub, and mode 4 does not extend to it.** Deciding with the user to approve, reject, or merge a pull request produces text they paste; it is never carried out by the agent, and no autonomous request changes that. Mode 4 covers staging, committing, and pushing — never reviewing or merging someone else's work.

## Workflow

This is mode 1, the default path: writing a commit command for what is already staged. Work through the steps in order; for a small, single-purpose change this file is enough on its own, so reach for a reference file only when a step points to one. The other five modes activate only when the user asks for them and each has its own section after this workflow.

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

**Trailers are excluded from this.** Convention detection covers subject shape, scope vocabulary, and tense — never trailers. Even if every past commit in the repo carries `Signed-off-by`, do not add one: a trailer is an assertion about who did the work and who vouches for it, which is not something to infer from pattern matching. Trailers come only from the opt-in below.

### 5. Compose the message

Build it in this order. See "The format" below for the rules, `references/craft.md` to lift a mechanical message or apply the body rules in full, and `references/conventional-commits.md` for the full type taxonomy and trailer catalog.

1. **Type** — the accurate one (feat, fix, refactor, …).
2. **Scope** — the affected area, if a clear one exists (`references/scopes-and-repos.md`).
3. **Subject** — imperative, concise, honest.
4. **Body** — usually none. Most commits are subject-only: there is no minimum length and no bullet quota. Include a body only when it adds one of — the problem or trigger behind the change; a non-obvious decision plus the alternative rejected and why; a consequence a reader wouldn't predict from the diff; migration, operational, or compatibility impact; a reference the diff can't carry (issue ID, spec link, incident). **Omit the body unless it adds one of those**, and never invent a reason to fill the space — an invented reason misleads. No restating the subject, no narrating the diff or listing files, no generic value claims ("improves maintainability", "for consistency"), no "This commit…" preamble. Test every line: could a reviewer recover this from `git show` alone? If yes, cut it.
5. **Footers** — `BREAKING CHANGE:` when applicable, and an issue reference (`Closes #123`) when the user supplied the issue or the branch name makes it unambiguous. No attribution trailers unless they were explicitly requested.

### 6. Self-check

Before presenting, confirm:

- Type matches what the diff actually does.
- Subject is imperative and completes "If applied, this commit will …".
- Subject ≤ 50 characters if reasonable, ≤ 72 hard; no trailing period.
- A body exists if — and only if — it adds the problem, a rejected alternative, an unpredictable consequence, migration impact, or a reference the diff can't carry. No body is the common, correct case.
- Every body line survives the `git show` test: nothing restates the subject, narrates the diff, lists files, or makes a generic value claim.
- Breaking changes are notated (`!` and/or a `BREAKING CHANGE:` footer).
- Trailers are well-formed (`references/conventional-commits.md`) and true. No attribution trailer is present unless the user asked for one this session or a standing instruction in the repo's agent context file calls for it — and none was inferred from history.
- Nothing is invented — every claim traces to a hunk, a referenced issue, or something the user said.

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
3. **Write the PR as plain Markdown** — a strong, specific title, then a description with **Summary** (why this branch exists), **What changed** (the substantive changes grouped by intent, not a file dump), **Testing** (how it was verified — from tests in the diff, CI, or manual steps; say what you could and couldn't confirm), and **Breaking changes** (spell out any contract change and its migration). Add a short reviewer/testing checklist when it earns its place. Tailor the emphasis to the PR's type (feature, fix, refactor, docs, breaking). Apply the same discipline as a commit body: drop any section with nothing real to say rather than filling it with "N/A", "None", or a restatement of the summary. Two substantive sections beat five padded ones.
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
   COMMIT_MSG
   ```

   Do not bypass hooks, rewrite an existing commit, or amend unless the user explicitly requested that separate action.
5. **Push safely.** Push the current branch normally to its configured upstream. If no upstream exists and `origin` is the unambiguous intended remote, use `git push -u origin HEAD`. Never force-push. If the remote or destination is ambiguous, or the push is rejected, stop and report the exact state instead of guessing or rewriting history.
6. **Report the result.** State the committed SHA and subject, the pushed branch and remote, and any work intentionally left unstaged. If staging or committing succeeds but pushing fails, say so plainly and do not claim completion.

Mode 4 changes nothing about attribution. It adds no `Co-authored-by:`, no `Signed-off-by:`, and no AI or agent identity, and the commit author stays whatever `git config user.name` / `user.email` resolves to. Being told to do the work is not a request to be credited for it. Trailers appear here only under the same opt-in that governs every other mode.

### Publishing a release (mode 4, on a further explicit request)

Mode 4 may also tag and publish a release — but only when the user explicitly asks to publish. Producing a release note, or the user approving one, is never enough. See `references/release-notes.md` for the full workflow.

**Check availability first:** `command -v gh` and `gh auth status`. If either fails, fall back to the release note as a Markdown block in the conversation and say which check failed. An unavailable `gh` is never a reason to stop — the note is the deliverable.

Then create an annotated tag matching the detected tag style, push it, and publish with `gh release create <tag> --title "<style-matched title>" --notes-file <file>`, matching the repo's prior pattern for latest/prerelease flags rather than choosing one.

**Hard limits:**

- **Never overwrite or move an existing tag or release.** If either already exists for this version, stop and report it.
- **Never `--force`, never delete a tag or release.**
- Mode 4's other constraints are unchanged: still never opens pull requests, still never force-pushes, still never `git add -A`.

## Mode 5: Release notes (only when asked)

This runs **only when the user asks for release-note or changelog content** — "write the release notes for 2.3.0", "what goes in the changelog for this version". It never fires as part of a commit or PR request. `references/release-notes.md` carries the full guidance; the essentials:

1. **Check the repo already versions itself** — version-shaped tags, a version field in the project, or existing GitHub releases. If none exist, say so and stop. Do not invent a versioning scheme or create the repo's first tag unprompted; offer an unversioned summary of what changed instead.
2. **Establish the range, and fetch first.** `git fetch --tags`, then find the most recent version tag reachable from `HEAD`. Cross-check it against the project's version field; **if they disagree, report the discrepancy and state which one you used as the base** rather than silently picking.
3. **Read the range, not just the subjects.** `git log <base>..HEAD --oneline` for the story, then `git diff <base>..HEAD` for what actually landed — a commit message can understate or misdescribe what it shipped.
4. **Match the repo's release style.** Prior releases outrank the canonical template: detect tag prefix, title style, heading levels, and section names from `gh release view <last>` or an existing `CHANGELOG.md`. Note any deviation once rather than reformatting the project's history.
5. **Group by user-visible effect,** not by file or commit, and exclude internal churn that changes no behaviour. Every entry traces to a real commit or hunk; omit any section with nothing real in it rather than writing "None". The commit-body discipline in `references/craft.md` applies to every line.
6. **Output the note as a Markdown block, and stop.** Writing a file happens only on explicit request; publishing is mode 4 plus its own explicit request.

## Mode 6: Review an incoming pull request (only when asked)

This runs **only when the user asks for help reviewing or landing a pull request** — "review PR 412", "should I merge this", "look at this PR with me". It is written for the person on the receiving end: the maintainer deciding what to do with someone else's branch. It never fires as part of a commit or PR-authoring request. `references/pr-review.md` carries the full guidance; the essentials:

1. **Gather the pull request, read-only.** Nothing here writes.

   ```bash
   gh pr view <n>
   gh pr diff <n>
   gh pr checks <n>
   gh pr view <n> --json commits,author,isCrossRepository,baseRefName,closingIssuesReferences
   gh repo view --json mergeCommitAllowed,squashMergeAllowed,rebaseMergeAllowed
   ```

   Read `CONTRIBUTING.md` if the repo has one. On a fork PR, workflows often need maintainer approval before they run — **empty `gh pr checks` output means "not run," not "passing."** Say which, never conflate them. On a large diff, read `gh pr diff <n> --name-only` first and then pull the files that matter.

2. **Separate what blocks the merge from what doesn't.** Two buckets, and only two. Something is **blocking** only if merging it leaves the project worse off than not merging: it breaks the build or a test, loses or corrupts data, opens a security hole, breaks a documented contract, or does not do what the PR claims. Everything else — naming, structure, style, a nicer approach — is a **suggestion**, and a suggestion is never on its own a reason to withhold a merge.

3. **Calibrate the bar to the project.** A small project is not a large one. Where there is no stated convention, no CI gate, and few contributors, the blocking bar is high and most findings are suggestions the maintainer is free to take or leave. Do not manufacture a blocker to justify a review. **If nothing blocks, say so in one plain line** before anything else, then offer the suggestions as optional.

4. **Talk it through before writing anything.** Present what the PR does, what blocks it if anything, and what is merely suggested — then ask what the user wants to do. Do not draft a review comment or a merge message until they have said which. When they lean one way, help them get there; when they ask what you would do, say so, and say plainly when a finding is a matter of taste rather than a defect.

5. **Produce exactly one Markdown block, for the decision they made.** Either a **review comment** to paste into GitHub's review box, or the **merge-commit message** if they are accepting. Not both, not preemptively. The user pastes it and clicks the button themselves.

6. **Act on nothing.** Never run `gh pr review`, `gh pr comment`, `gh pr merge`, `gh pr close`, or any other write verb. A follow-up "go ahead", "approve it", or "yes, merge" is a decision about what the *block should say* — never permission to execute it. Confirm the decision, hand over the text, and stop.

### The merge-commit message

If the user is accepting, the message depends on how the repo merges — which step 1 already read:

- **Squash merge** — GitHub prefills the subject from the PR title and the body from every branch commit concatenated. That default is usually noise, and it is the commit that lands on the default branch and lives in `git log` forever. Write it fresh from the diff with the normal discipline (steps 4–6 of the main workflow); the contributor's commit subjects are a hint at intent, not source text to clean up.
- **Merge commit** — GitHub's `Merge pull request #<n> from …` default is fine. Leave it alone unless the user asks.
- **Rebase merge** — no new message exists to write. Say so rather than producing one.

**Co-authors are transcribed here, never invented.** A squash collapses every commit on the branch into one, and GitHub credits only the PR author — so when `gh pr view --json commits` shows more than one distinct author, carry each one across as a `Co-authored-by: Name <email>` trailer, copied exactly from the real commits. This is the one place the skill emits an attribution trailer without being asked, and it is still not inference: it preserves credit the squash would otherwise destroy. Add `Closes #<n>` when the PR genuinely links an issue. Nothing else changes about the attribution rules — no `Signed-off-by:`, no agent identity, no name or address that did not come from a real commit.

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
- **Always available — these are not attribution.** `BREAKING CHANGE: <what breaks and the migration>` (uppercase, exactly), required by the grammar; and issue references — `Closes #123` / `Fixes #123` (GitHub closes the issue on merge to the default branch) or `Refs #123` (reference only) — when the user supplies the issue or it is unambiguous from the branch name. Never guess an issue number.
- **Attribution trailers are off by default, in every mode.** `Co-authored-by:`, `Signed-off-by:`, `Reviewed-by:`, and AI or agent attribution are emitted only when **one** of these holds: the user asks in the session ("sign this off", "add Sam as co-author", "credit the agent"), or a standing instruction exists in the consuming repo's own agent context file (`AGENTS.md`, `CLAUDE.md`, or equivalent). Nothing else opts in — not history, not the branch name, not the diff.
- On an explicit request, all of these are supported: `Co-authored-by:`, `Signed-off-by:`, `Reviewed-by:`, issue-reference footers, and AI or agent attribution.
- **One exception, in mode 6 only.** When squash-merging a pull request whose branch has more than one commit author, `Co-authored-by:` lines are transcribed from the real commit metadata without being asked, because the squash would otherwise destroy authorship that already exists. Transcription is not inference: every name and address comes from an actual commit. See `references/pr-review.md`.
- Named humans come only from values the user supplies, or — in the mode 6 case above — from real commit metadata. Never invent a name or an email address, and only add trailers that are true.

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

For a fuller gallery — breaking changes, multi-paragraph bodies, the default and opt-in trailer forms, worked splits, and a file-selection walkthrough — see `references/examples.md`.

## Reference files

Read these on demand; don't load them for routine commits.

- **`references/conventional-commits.md`** — the full grammar, complete type taxonomy with SemVer impact, breaking-change rules, the footer/trailer catalog, and revert/merge handling. Read when unsure about a type, breaking-change formatting, or trailer syntax.
- **`references/examples.md`** — an annotated gallery of exemplary commits, including breaking changes, rationale-heavy bodies, trailers, worked splits, and a file-selection walkthrough. Read when composing a non-trivial message, a split, or a staging selection.
- **`references/craft.md`** — how to write a subject and body that communicate, the *why*-not-*how* principle, and a catalogue of common bad commits with fixes. Read when lifting a mechanical message.
- **`references/scopes-and-repos.md`** — choosing a scope (with backend/Django/DRF and other ecosystem cues), and detecting and matching a repository's existing convention. Read when picking a scope or entering an unfamiliar repo.
- **`references/pull-requests.md`** — base-branch detection, gathering the branch's history and diff, and writing a PR title and a structured description (summary, changes, testing, breaking changes) with type-aware emphasis and a reviewer checklist. Read when writing pull-request content.
- **`references/pr-review.md`** — reviewing an incoming pull request: gathering it read-only, the fork-CI caveat, the blocking-versus-suggestion test and how to calibrate it to project size, discussing the decision with the maintainer, and writing the review comment or the squash-merge message with co-author transcription. Read when reviewing or landing someone else's pull request.
- **`references/release-notes.md`** — the versioning precondition, establishing the range since the last release, detecting and matching a repo's release style, the canonical section template, content rules, output targets, and the `gh` publish workflow with its hard limits. Read when writing a release note or publishing a release.
