# Reviewing an incoming pull request

Mode 6. This is written for the maintainer on the receiving end — the person deciding what to do with a branch someone else wrote. It is not for authoring your own PR; that is mode 3 and `references/pull-requests.md`.

The mode is **read-only**. It reads the pull request through `gh`, works the decision through with the maintainer, and produces one Markdown block for them to paste into GitHub. The agent never approves, requests changes, comments, merges, or closes. The button is always the human's.

## Gathering the pull request

All read-only:

```bash
gh pr view <n>
gh pr diff <n>
gh pr checks <n>
gh pr view <n> --json commits,author,isCrossRepository,baseRefName,closingIssuesReferences,reviews
gh repo view --json mergeCommitAllowed,squashMergeAllowed,rebaseMergeAllowed
```

Also read `CONTRIBUTING.md` if it exists — many projects state their commit convention, DCO or sign-off requirement, or changelog rules there, and a maintainer checking a PR would want them.

**Read the existing review threads before forming an opinion.** Someone may have already raised your finding, or the contributor may have already answered it. Repeating a settled point wastes everyone's time and reads as inattentive.

**Fork PRs need care with CI.** When `isCrossRepository` is true, workflows frequently require maintainer approval before they run at all. Empty or missing output from `gh pr checks` therefore means *not run* — it does not mean passing. Report which one it is; never let silence read as green.

**Large diffs.** Read `gh pr diff <n> --name-only` first, decide which files carry the actual change, then pull those. A 60-file diff read in full will crowd out the reasoning you need the context for.

**Check the base is current.** If the base branch has moved substantially since the PR was opened, the diff you are reading may not be the diff that will land. Say so rather than reviewing a stale picture.

## What actually blocks a merge

Two buckets. Only two.

**Blocking** — merging leaves the project worse off than not merging:

- It breaks the build, or breaks an existing test.
- It loses or corrupts data, or a migration is unsafe or irreversible.
- It opens a security hole — injection, missing authorization, a leaked secret, a dependency with a known advisory.
- It breaks a documented contract (an API response shape, a public function signature) without notating the break.
- It does not do what the PR says it does.

**Suggestion** — everything else. Naming, file organisation, a nicer abstraction, a test that could be broader, a comment that could be clearer, a pattern you would have written differently. These are worth saying. None of them is on its own a reason to withhold a merge.

The test: *would you rather have this branch in the tree, or not have it?* If the answer is "in the tree, with a note," it is a suggestion.

## Calibrating to the project

The blocking bar is not the same everywhere, and applying a large project's bar to a small one is the most common way this mode goes wrong.

**Small projects** — few contributors, no stated convention, little or no CI. The bar is high. Most findings are suggestions the maintainer is free to take or leave, and a contributor who showed up with a working fix has already done the hard part. Missing tests in a repo that has no test suite is not a blocker. A commit message that ignores a convention the project never wrote down is not a blocker.

**Larger projects** — stated conventions, CI gates, multiple reviewers. Documented requirements are worth checking, and a project that treats its own rules as gates is entitled to have them enforced. But a convention miss is still usually a suggestion unless the project genuinely gates on it.

Two rules hold at any size:

- **Never manufacture a blocker.** Reaching for something to object to, so that the review looks thorough, produces exactly the reviews that make contributors stop contributing.
- **Lead with the verdict.** If nothing blocks, say so in one plain line before anything else. The maintainer's first question is "can I merge this," and burying the answer under six paragraphs of observation answers it badly.

## Working the decision through

Present, in this order: what the PR does, what blocks it if anything, what is merely suggested. Then ask what the user wants to do. Do not draft any block before they answer.

- When they ask what you would do, say so directly.
- When a finding is taste rather than defect, label it as taste. Maintainers make merge decisions with context you do not have — a deadline, a relationship with the contributor, a plan to refactor the area next week.
- When they decide to merge over a finding you raised, that is a legitimate call. Note it once if it is genuinely risky; do not relitigate.
- When they are unsure, help them weigh it rather than deciding for them.

## Output 1: the review comment

For blocking findings, or a comment the maintainer wants to leave. Plain Markdown, ready to paste into GitHub's review box.

- Open with the verdict in one line.
- Group blocking items first, suggestions after, and label which is which so the contributor knows what they must address versus what is optional.
- Point at code, not at the person. `file.py:120` and what is wrong there; never "you clearly didn't think about". Say what would resolve it, not just that it is wrong.
- Say something true about what is good in the branch when there is something true to say. Skip it rather than manufacturing praise.
- For an outside contributor, the register is different from a teammate's: they gave you unpaid work with no context on your standards. Be specific and actionable, and put anything they could not have known — an unwritten convention, a plan they were not party to — on the project, not on them.
- Keep it as short as the findings allow.

## Output 2: the merge-commit message

Only when the maintainer has decided to accept. What to write depends on the repo's merge method, already read in gathering.

**Squash merge.** GitHub prefills the subject from the PR title and the body from every branch commit concatenated together — usually noise, and it is the commit that lands on the default branch and stays in `git log` and the changelog forever. It is the most consequential message in the whole PR lifecycle and, by default, nobody writes it.

Write it fresh from the diff, with the ordinary discipline: accurate type, honest scope, imperative subject, and a body only when it adds the problem, a rejected alternative, an unpredictable consequence, migration impact, or a reference the diff cannot carry. The contributor's commit subjects are a hint at intent — they may be `wip`, `fix stuff`, and `fix stuff again` — not source text to tidy up.

**Merge commit.** GitHub's `Merge pull request #<n> from <user>/<branch>` default is fine and conventional. Leave it unless asked.

**Rebase merge.** No new message exists. The branch commits carry over as-is. Say so rather than producing a message that has nowhere to go.

### Trailers on a squash

**`Co-authored-by:` is transcribed, never invented.** A squash collapses every commit on the branch into one and credits only the PR author. When `gh pr view <n> --json commits` shows more than one distinct author, carry each across:

```text
Co-authored-by: Real Name <real@email>
```

copied exactly from the real commit metadata. This is the single place the skill emits an attribution trailer without being asked — and it is still not inference. It preserves authorship that already exists and that the squash would otherwise destroy.

Add `Closes #<n>` when `closingIssuesReferences` shows a genuine link, or when the maintainer supplies it. Add `Signed-off-by:` only if the project requires a DCO and the *maintainer* is signing off on their own merge.

Everything else about attribution is unchanged: no agent identity, no `Reviewed-by:` unless asked, and never a name or address that did not come from a real commit or from the user.

### Worked example

A three-commit fork PR adding rate limiting, squash-merged:

```text
feat(api): rate-limit the password reset endpoint

Reset requests were unthrottled, so the endpoint could be used to
enumerate registered addresses and to flood a known user's inbox.
Limits to 5 requests per hour per address, keyed in Redis.

Closes #318
Co-authored-by: Dana Okafor <dana@example.com>
```

The branch commits were `add throttle`, `fix test`, and `address review`. None of them says what shipped; the squash message does.

## Hard limits

- **Never run a write verb.** Not `gh pr review`, `gh pr comment`, `gh pr merge`, `gh pr close`, `gh pr edit`, `gh pr ready`, or any `gh api` call with a method other than GET.
- **A confirmation is about the text, not about execution.** "Approve it", "go ahead", "yes, merge" tell you what the block should say. They are never permission to carry it out, and no autonomous request extends to this mode — mode 4 covers staging, committing, and pushing your own work, never merging someone else's.
- **One block at a time**, for the decision actually made. Do not produce a review comment and a merge message together on the chance one is wanted.
- **Confirm before producing any block**, and state plainly at the end that the user runs the action themselves.
- **Never assert a check passed that you did not see pass.**
