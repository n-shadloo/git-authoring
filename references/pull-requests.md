# Pull requests

Writing a pull request that a reviewer can act on: a title that says what the branch does, and a description that answers what changed, why, how it was tested, and what breaks. This is mode 3 and runs only when the user asks for PR content — it is never part of a plain commit request. Opening and merging the pull request run under mode 4 whenever the user asks for them. Everything here is grounded in the branch's real history and diff; nothing is invented.

## Contents

- Detecting the base branch
- Reading what the branch did
- The title
- The description
- PR types and where the emphasis goes
- Verifying from history and diff
- Attribution and opening the PR
- Opening and merging the pull request
- A worked example

## Detecting the base branch

Work out the base automatically, without asking, and state the base you settled on. Try these in order (all read-only):

```bash
# 1. The remote's declared default branch — most reliable when set.
git symbolic-ref --quiet refs/remotes/origin/HEAD    # e.g. refs/remotes/origin/main
# 2. Fall back to whichever conventional remote branch exists.
git rev-parse --verify --quiet origin/main
git rev-parse --verify --quiet origin/master
# 3. Last resort: the local branch.
git rev-parse --verify --quiet main
git rev-parse --verify --quiet master
```

Strip `refs/remotes/origin/` to get the branch name. If `origin/HEAD` isn't set and both `origin/main` and `origin/master` are absent, fall back to the local `main`/`master`; if none of these exist, say so and ask which branch to target rather than guessing.

## Reading what the branch did

Compare the current branch to the base, from the merge base, so you see only this branch's work — not everything that landed on the base since it forked:

```bash
git branch --show-current
git log <base>..HEAD --oneline          # the branch's commits
git diff <base>...HEAD --stat           # files touched (three-dot: from the merge base)
git diff <base>...HEAD                   # the full branch diff
```

The two-dot `git log <base>..HEAD` lists commits on HEAD not on the base. The three-dot `git diff <base>...HEAD` diffs against the merge base, which is what a reviewer sees on the PR. Read the actual diff, not just the commit subjects — the subjects tell you the intended story, the diff tells you what really changed.

## The title

One line, specific, in the imperative — the same discipline as a commit subject.

- **Single-purpose branch** — mirror the primary commit's subject. A branch that only adds idempotency keys becomes `feat(orders): add idempotency keys to checkout`.
- **Multi-commit branch** — summarise the theme, not every commit. Don't chain concerns with "and"; if the branch really does several unrelated things, that's worth flagging to the user (it may want splitting into more than one PR).
- **Match the repo's PR-title style.** Projects that squash-merge with Conventional Commit titles want `type(scope): …`; others want a plain sentence. Look at recent merged PRs or the squash history and follow suit.
- Keep it tight and drop the trailing period.

## The description

Plain Markdown, structured so a reviewer can scan it. Include only the sections that carry weight for this branch. When a section has nothing real to say, **drop the heading entirely** rather than filling it with "N/A", "None", or a paraphrase of the summary — a description with two substantive sections beats one with five padded ones. The discipline that governs a commit body governs every line here: if a reviewer can recover it from the diff or the Files-changed tab, it isn't worth their time.

**Summary** — why this branch exists, in a sentence or two. The problem, the missing capability, or the goal. Not a restatement of the title.

**What changed** — the substantive changes, grouped by intent, as a short bulleted list. Describe behaviour and decisions ("added an Idempotency-Key check that returns the original order on a repeat"), not a file inventory ("edited views.py, urls.py") — the Files-changed tab already lists files.

**Testing** — how the change was verified. Draw this from the branch itself: tests added or updated in the diff, CI that runs on the branch, and any manual steps the history implies. Be honest about coverage — if the branch adds no tests, say testing is unverified and suggest what a reviewer should exercise, rather than implying it passed.

**Breaking changes** — call out any incompatible change to an API, a config surface, a CLI, or a data shape, and give the migration. Mirror the commit's `BREAKING CHANGE:` notation where one exists. Always check for a break; when there is genuinely nothing to report, drop the section rather than writing "None". Keep it when you have something substantive to say about compatibility — that a new header is optional, that old clients keep working — because that's a real claim, not a placeholder.

**Reviewer / testing checklist** *(when it earns its place)* — a few `- [ ]` items for a reviewer or for pre-merge verification: run the migration, check the mobile client against the new response shape, confirm the feature flag default. Skip it for a one-line fix.

## PR types and where the emphasis goes

Tailor the weight of each section to what the branch is:

- **Feature** — lead with the capability and the user-facing behaviour; testing and any new config matter.
- **Bug fix** — state the symptom, the root cause, and the fix; a regression test is the strongest evidence, so surface it.
- **Refactor** — emphasise that behaviour is unchanged and how that was confirmed (existing tests still pass, no diff in output); call out any risky move.
- **Performance** — give the before/after numbers and how they were measured.
- **Docs / chore / build** — keep it short; a summary and a one-line testing note are usually enough.
- **Breaking** — put the migration front and centre and coordinate with any dependent release.

## Verifying from history and diff

The PR must reflect what the branch actually contains. Every claim traces to a commit, a diff hunk, a test, or a CI configuration you can see. If something can't be verified from the branch — whether a manual QA pass happened, whether a downstream client was updated — say it's unverified or leave it to the reviewer rather than asserting it. A confident but wrong PR description wastes review time and erodes trust in the next one.

## Attribution and opening the PR

- **No attribution by default.** A PR description carries no "generated by" line, no AI co-author, and no other attribution — the same default that governs commits. Include attribution only on an explicit request from the user or a standing instruction in the repo's agent context file.
- **Open the PR when the user asks you to.** By default, produce the Markdown for the user to paste into the PR form, or the exact command:

  ```bash
  gh pr create --base <base> --head "$(git branch --show-current)" \
    --title "feat(orders): add idempotency keys to checkout" \
    --body-file <path-to-description.md>
  ```

  When the user asks the agent to open the pull request — "open the PR", "create it" — open it yourself, as the next section describes.

## Opening and merging the pull request

Mode 4 opens and merges pull requests whenever the user asks for either, in the first request ("commit, push, open the PR, and merge it once CI passes") or in a later message. Neither needs a further confirmation, and both apply to the user's own pull request and to someone else's.

Check `command -v gh` and `gh auth status` first. If either fails, say which check failed and hand over the exact command instead.

**Opening.**

1. Push the branch first, as mode 4 does. Find the base as in "Detecting the base branch".
2. Write the title and the description as this file describes, and run the attribution check of `SKILL.md`, "Proof of completion", over both; it must print nothing. Write the body to a file, or pass it in a quoted heredoc, so the shell expands nothing in it.
3. Run `gh pr create --base <base> --head <branch> --title "<title>" --body-file <file>`, or the exact command the user gave. Report the URL `gh` prints.

**Merging.**

1. Read the state: `gh pr checks <number>` and `gh pr view <number> --json state,mergeable,mergeStateStatus,reviewDecision`. Empty check output means the checks have not run, not that they passed.
2. When the user asked to merge after CI, wait for the checks. Use the harness's own CI notification where it has one; otherwise `gh pr checks <number> --watch --fail-fast` waits in one command.
3. Merge only while every required check passes and GitHub reports the pull request mergeable. A failing check, a conflict, or a blocked state stops the merge with the handoff of `SKILL.md`, "Stop conditions". Where the user asked for a working merge, fix the cause on the branch — merge the base in for a conflict, never rebase a pushed branch — push, and wait again.
4. Merge with the method the user named — `gh pr merge <number> --merge`, `--squash`, or `--rebase` — or else with the method the repository's history and settings use. Pass `--delete-branch` when the user asked for it or the repository always deletes merged branches. On a squash, write the message as `references/pr-review.md` describes.
5. After the merge, update the local base when the user asked for it: `git checkout <base>`, then `git pull --ff-only`.
6. Prove it: `gh pr view <number> --json url,state,mergeCommit` reports `MERGED` and the merge commit SHA.

**Hard limits.**

- Never pass `--admin` to merge past a failing check or a branch protection rule, and never enable auto-merge with `--auto`, unless the user asks for exactly that.
- Never close, reopen, or edit a pull request, or change its base, unless the user asks.
- Never merge while a required check fails or has not finished, when the user asked to merge after CI.

## A worked example

Branch `feat/idempotent-checkout` off `main`, three commits adding an idempotency guard plus its test and a short doc note. The generated PR:

```markdown
## feat(orders): add idempotency keys to checkout

### Summary
Retries from the mobile client on a flaky connection were submitting checkout
twice and creating duplicate orders. This branch makes checkout idempotent so a
repeated request returns the original order instead of creating another.

### What changed
- Checkout now accepts an `Idempotency-Key` header; a repeated key returns the
  stored result rather than creating a new order.
- Keys are persisted for 24h, covering realistic retry windows without growing
  the table unbounded.
- Documented the header and its retry semantics in the API reference.

### Testing
- Added `test_checkout_idempotency` covering a first request, a duplicate key,
  and an expired key; passes locally and in CI.
- Manually verified a double-submit against the staging app returns one order.

### Breaking changes
The `Idempotency-Key` header is optional; requests without it behave exactly as
before, so no client has to change.

### Reviewer checklist
- [ ] Confirm the 24h key TTL matches the mobile client's retry ceiling.
- [ ] Sanity-check the new index on the key lookup under load.
```

The title mirrors the branch's primary commit; every section is backed by a commit, the diff, or the added test; testing states what was and wasn't checked; and breaking changes are addressed explicitly rather than skipped.
