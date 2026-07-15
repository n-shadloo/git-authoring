# Scopes and repository conventions

Two related judgment calls: what scope to put in the parentheses, and whether this repo even uses Conventional Commits.

## Contents

- Choosing a scope
- Scope cues by ecosystem
- Detecting the repository's convention
- Non–Conventional-Commits conventions to recognise

## Choosing a scope

A scope names the part of the codebase the change affects: `feat(auth):`, `fix(parser):`. It's optional — omit it when no single area fits or when the change is repo-wide.

- **Derive it from what changed**, not from a fixed menu. The directory or module the diff touches is usually the scope: changes under `payments/` → `payments`.
- **One clear scope beats several.** If a change spans many areas, either it's genuinely cross-cutting (drop the scope) or it should be split.
- **Keep scopes stable and coarse.** Reuse the handful the repo already uses (visible in `git log`) rather than inventing a new one per commit. `auth` is a better scope than `login-form-validation`.
- **Match existing casing and vocabulary.** If the repo writes `feat(API)` or `feat(ui)`, follow suit.

## Scope cues by ecosystem

Starting points, not rules — always defer to what the repo already uses.

- **Django / DRF backend**: `models`, `migrations`, `views`, `serializers`, `urls`, `admin`, `auth`, `api`, `settings`, `tasks`, plus app names (`orders`, `payments`, `users`). Deployment-adjacent: `deploy`, `nginx`, `gunicorn`, `docker`, `ci`.
- **Frontend**: component or feature areas (`auth`, `dashboard`, `cart`), or `ui`, `styles`, `router`, `api-client`.
- **Libraries / tooling**: the public surface (`core`, `cli`, `config`, `parser`) or package names in a monorepo.
- **Infra / ops**: `ci`, `docker`, `k8s`, `nginx`, `systemd`, `terraform`.

## Detecting the repository's convention

Before imposing Conventional Commits, look at `git log --oneline -20`:

- If recent commits already look like `type(scope): subject`, you're in a Conventional Commits repo — carry on as normal.
- If they follow a *different* consistent pattern, match that pattern instead. A commit-message convention is a property of the repository; a lone Conventional Commit in a history that doesn't use them is the thing that looks out of place.
- If the history is inconsistent (no discernible convention), default to Conventional Commits — it's the strongest default and nudges the history toward structure.

When you deliberately follow a non-default convention, note it in one line ("matching this repo's `Fixed #nnnn` style") so the choice is visible.

## Non–Conventional-Commits conventions to recognise

**Django** uses its own format, not Conventional Commits:

```text
Fixed #31234 -- Added a system check for duplicated field names.
```

- The subject starts with `Fixed #<ticket>` (which closes the Trac ticket) or `Refs #<ticket>` (which references it), then ` -- `, then a **past-tense** description.
- Branch or backport prefix when relevant: `[5.0.x] Fixed #… -- …`.
- Credit reporters and reviewers in the body ("Thanks … for the review.") and use `Co-authored-by:`.

**Angular** uses Conventional Commits with a fixed scope list drawn from its packages (`common`, `compiler`, `router`, `forms`, …) and an imperative, lowercase, no-period subject under a 100-character header limit.

**Linux kernel** uses a plain imperative subject prefixed by subsystem (`net: `, `mm: `), a mandatory explanatory body, and a `Signed-off-by:` trailer (DCO). No `type():` prefix.

The common thread: read the history, then be consistent with it. Conventional Commits is the default this skill reaches for, not a rule to force onto a repo that has chosen otherwise.
