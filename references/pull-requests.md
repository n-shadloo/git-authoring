# Pull requests

Writing a pull request that a reviewer can act on: a title that says what the branch does, and a description that answers what changed, why, how it was tested, and what breaks. This is mode 3 and runs only when the user asks for PR content — it is never part of a commit request or the autonomous stage-commit-push mode. Everything here is grounded in the branch's real history and diff; nothing is invented.

## Contents

- Detecting the base branch
- Reading what the branch did
- The title
- The description
- PR types and where the emphasis goes
- Verifying from history and diff
- Attribution and opening the PR
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

Plain Markdown, structured so a reviewer can scan it. Include the sections that carry weight for this branch; don't pad with empty headings.

**Summary** — why this branch exists, in a sentence or two. The problem, the missing capability, or the goal. Not a restatement of the title.

**What changed** — the substantive changes, grouped by intent, as a short bulleted list. Describe behaviour and decisions ("added an Idempotency-Key check that returns the original order on a repeat"), not a file inventory ("edited views.py, urls.py") — the Files-changed tab already lists files.

**Testing** — how the change was verified. Draw this from the branch itself: tests added or updated in the diff, CI that runs on the branch, and any manual steps the history implies. Be honest about coverage — if the branch adds no tests, say testing is unverified and suggest what a reviewer should exercise, rather than implying it passed.

**Breaking changes** — call out any incompatible change to an API, a config surface, a CLI, or a data shape, and give the migration. If there are none, write "None" so the reviewer knows it was considered, not overlooked. Mirror the commit's `BREAKING CHANGE:` notation where one exists.

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

- **No AI attribution.** Mode 3 never includes a "generated by" line, an AI co-author, or any other signal that an AI wrote the PR.
- **Never open the PR yourself.** Produce the Markdown for the user to paste into the PR form. If they'd rather run it from the terminal, offer the exact command and let them run it:

  ```bash
  gh pr create --base <base> --head "$(git branch --show-current)" \
    --title "feat(orders): add idempotency keys to checkout" \
    --body-file <path-to-description.md>
  ```

  Opening the PR remains the user's action. A follow-up "go ahead" does not convert mode 3 into an execution mode, and mode 4 covers staging, committing, and pushing — not opening a pull request.

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
None. The header is optional; requests without it behave exactly as before.

### Reviewer checklist
- [ ] Confirm the 24h key TTL matches the mobile client's retry ceiling.
- [ ] Sanity-check the new index on the key lookup under load.
```

The title mirrors the branch's primary commit; every section is backed by a commit, the diff, or the added test; testing states what was and wasn't checked; and breaking changes are addressed explicitly rather than skipped.
