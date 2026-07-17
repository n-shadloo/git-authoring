# Commit examples

A gallery of commits worth imitating, each with the situation that produced it and a note on why it's shaped the way it is. Match the closest pattern; don't copy the wording literally.

## Contents

- Single-line commits
- Commits with a body
- Breaking changes
- Trailers and attribution
- Backend / API-contract changes
- Splitting a diff into several commits
- Choosing which files to stage

## Single-line commits

When the change is self-explanatory, one line is the whole message. A body would just restate the diff.

```text
fix(auth): prevent redirect loop on expired token
```

```text
docs(readme): document the ZARINPAL_SANDBOX env var
```

```text
chore(deps): bump djangorestframework to 3.15.2
```

```text
test(orders): cover zero-quantity checkout
```

## Commits with a body

Add a body when the *why* isn't visible in the diff. The subject says what; the body says why, and what you weighed.

Situation: retries from the mobile client were creating duplicate orders.

```text
feat(orders): add idempotency keys to checkout

Client retries on a flaky connection were submitting checkout twice and
creating duplicate orders. Callers now send an Idempotency-Key header;
a repeated key returns the original order instead of creating a new one.

Keys are stored for 24h, which covers realistic retry windows without
growing the table unbounded.

Closes #482
```

Situation: a query was slow because it wasn't using an index.

```text
perf(reports): add composite index for the monthly summary query

The summary joined orders and line_items and filtered on (tenant_id,
created), which had no matching index and forced a sequential scan on
the largest table. The composite index drops the query from ~2.4s to
~40ms on production-sized data.
```

Note how the body captures the reasoning a future reader — or `git blame` — can't reconstruct from the code: the numbers, the cause, the trade-off.

## Breaking changes

`!` flags the break at a glance; the footer explains the migration. Use both for anything a consumer must react to.

```text
feat(api)!: return ISO 8601 timestamps

BREAKING CHANGE: `created` and `modified` are now ISO 8601 strings
rather than Unix-epoch integers. Clients parsing them as numbers must
switch to date parsing.
```

A breaking change can ride on any type:

```text
refactor(client)!: drop the deprecated v1 request signer

BREAKING CHANGE: v1 request signing was removed. Callers still on v1
must move to `sign_v2`, which has been available since 2.3.
```

## Trailers and attribution

Credit collaborators and link issues with footers:

```text
fix(serializers): handle null profile in user payload

A user without a profile row raised AttributeError during
serialization. Fall back to an empty profile object so the endpoint
returns 200 with nulls instead of 500.

Co-authored-by: Sara Ahmadi <sara@example.com>
Reviewed-by: Nima Shadloo <nima@example.com>
Closes #517
```

## Backend / API-contract changes

When your API has a consumer — a Flutter or web client — the commit is where the contract change is recorded. Make the impact explicit, and mark it breaking if the shape changes incompatibly.

Additive, non-breaking:

```text
feat(api): include `display_name` in the user endpoint

Additive only — existing fields are unchanged, so current clients keep
working. The mobile client will use `display_name` for the profile
header.
```

Breaking contract change:

```text
feat(api)!: paginate the transactions endpoint

BREAKING CHANGE: /transactions now returns a paginated envelope
({count, next, previous, results}) instead of a bare array. Clients must
read `results` and follow `next`. Coordinated with the app release that
adds pagination handling.
```

## Splitting a diff into several commits

When staged changes cover unrelated concerns, don't force one message. Propose the split and give the commands, smallest logical unit first.

Situation: the working tree holds a bug fix in the auth serializer, an unrelated new endpoint, and a dependency bump — all unstaged. Stage and commit them separately:

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

Each commit is now a single, revertible, reviewable idea, and the history reads as a sequence of decisions rather than one undifferentiated "update".

If one file holds two unrelated changes, stage selectively with `git add -p` and commit the hunks separately.

## Choosing which files to stage

When the user asks the agent to *pick* what to stage — rather than committing what's already staged — the same grouping judgment applies, but you start from an empty index and select the one coherent set yourself. The rule is: related changes only, not everything, not an arbitrary mix.

Situation: the user says "choose the files that belong together and commit them." `git status --short` shows a mix — a fix to the auth serializer with its test, an unrelated new endpoint, and a stray formatting change in a settings file. The auth fix and its test are the tightest, most coherent unit, so stage exactly those and commit them first:

```bash
# Chosen set: the null-profile fix and the test that covers it — nothing else.
git add users/serializers.py users/tests/test_serializers.py
git commit -m "fix(serializers): handle null profile in user payload"
```

Then say what was deliberately left out and why, so the user can decide what comes next:

> Staged and committed the null-profile fix with its test. Left unstaged: the new
> `transactions` endpoint (a separate `feat`) and a formatting-only change in
> `settings.py` (a `style` commit, or drop it). Want either of those next?

The point is the *selection*: you didn't `git add -A`, and you didn't fold three unrelated concerns into one commit. You picked the set a careful developer would, proposed it, and — as always — staged and committed only once the user was on board.
