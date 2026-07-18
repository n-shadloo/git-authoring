# Conventional Commits reference

The precise rules behind the format. The `SKILL.md` body covers the common case; this file is for when you need the exact grammar, a less common type, breaking-change details, or trailer syntax.

## Contents

- Grammar
- Type taxonomy
- Choosing between close types
- Breaking changes
- Footers and trailers
- Reverts
- Merges
- SemVer mapping

## Grammar

```text
<type>[optional scope][!]: <description>

[optional body]

[optional footer(s)]
```

Rules, from the Conventional Commits 1.0.0 spec:

1. A commit is prefixed with a **type** (a noun such as `feat` or `fix`), then an optional scope, an optional `!`, and a required `:` and space.
2. `feat` must be used for a new feature; `fix` must be used for a bug fix.
3. A **scope** may follow the type in parentheses, naming a section of the codebase: `feat(parser):`.
4. A **description** immediately follows the colon and space.
5. A longer **body** may follow one blank line after the description. It is free-form and may span multiple paragraphs.
6. One or more **footers** may follow one blank line after the body. Each footer is a token, then `: ` or ` #`, then a value. Tokens use `-` instead of spaces (`Reviewed-by`), except `BREAKING CHANGE`, which is allowed as a two-word token.
7. A **breaking change** is signalled by `!` before the `:` in the prefix, or by a `BREAKING CHANGE:` footer, or both.
8. Everything except the literal `BREAKING CHANGE` is case-insensitive to tools; `BREAKING CHANGE` must be uppercase. `BREAKING-CHANGE` is a synonym.

## Type taxonomy

The two types the spec mandates:

- **feat** — introduces a new feature. → SemVer MINOR.
- **fix** — patches a bug. → SemVer PATCH.

The conventional set beyond those (from the Angular convention and `@commitlint/config-conventional`), none of which affect SemVer on their own:

- **docs** — documentation only: READMEs, docstrings, comments, guides.
- **style** — changes that don't affect meaning: whitespace, formatting, semicolons, import order. Not "CSS/visual styling".
- **refactor** — a code change that neither fixes a bug nor adds a feature: renames, extractions, restructuring with identical behaviour.
- **perf** — a change that improves performance. → SemVer PATCH.
- **test** — adds missing tests or corrects existing ones.
- **build** — the build system or external dependencies: bundler config, `pyproject.toml`, lockfiles. Dependency bumps are conventionally `build(deps):` — or `chore(deps):` in many repos, so match the repo.
- **ci** — continuous-integration config and scripts: GitHub Actions, GitLab CI, pipelines.
- **chore** — routine maintenance touching neither source nor tests: `.gitignore`, tooling config, housekeeping.
- **revert** — reverts a previous commit (see Reverts below).

## Choosing between close types

- **fix vs refactor** — did behaviour change for the user? A corrected defect is `fix`; code reshaped with identical behaviour is `refactor`.
- **feat vs fix** — is the capability new, or was it meant to work already? A newly added ability is `feat`; making an existing-but-broken ability work is `fix`.
- **refactor vs perf** — same output faster is `perf`; same output cleaner is `refactor`.
- **style vs refactor** — `style` never changes the parsed or compiled meaning (pure formatting). Renaming a variable is `refactor`, not `style`.
- **chore vs build** — dependency and build-tool changes are `build`; unrelated housekeeping is `chore`.

When two types could fit because the commit really does two things, that's usually a signal to split it (`references/examples.md`).

## Breaking changes

A breaking change correlates with a SemVer MAJOR bump and can accompany any type. Signal it in either or both of:

- `!` before the colon: `feat(api)!: …` or `refactor!: …`.
- A footer: `BREAKING CHANGE: <what breaks and how to migrate>`.

If you use `!` without a footer, the subject itself must describe the break. Prefer adding the footer whenever a migration note would help the reader.

```text
feat(config)!: read settings from environment first

BREAKING CHANGE: environment variables now take precedence over
config-file values. Deployments that relied on the file winning must
move those values into the environment.
```

## Footers and trailers

Footers follow the git trailer convention: `Token: value`, one per line, in a block after a blank line. Common tokens:

Issue and PR references

- `Closes #123`, `Fixes #123`, `Resolves #123` — GitHub and GitLab close the issue when the commit merges into the default branch. (`close/closes/closed`, `fix/fixes/fixed`, and `resolve/resolves/resolved` all work; on a non-default branch the issue is referenced but not closed.)
- `Refs #123` — references without closing.

Attribution (recognised by GitHub and GitLab; co-authors appear in the contributors graph)

- `Co-authored-by: Full Name <email>` — a genuine collaborator on the change (pairing, mob, applying someone's patch).
- `Reported-by:` / `Suggested-by:` / `Helped-by:` — credit for the report, the idea, or debugging help.
- `Reviewed-by:` / `Acked-by:` / `Tested-by:` — review, agreement, and testing sign-off.

Sign-off

- `Signed-off-by: Full Name <email>` — a Developer Certificate of Origin sign-off, required by projects such as the Linux kernel. Add `-s` to the quoted-heredoc commit command: `git commit -s -F - <<'COMMIT_MSG'`.

Breaking change

- `BREAKING CHANGE: …` — uppercase, as above.

Only add trailers that are true. An invented reviewer or co-author is worse than none.

## Reverts

When reverting, Conventional Commits recommends the `revert` type with a footer identifying the reverted commit:

```text
revert: feat(orders): add idempotency keys to checkout

Refs: 676104e
```

`git revert` generates a default message ("Revert \"…\""). Reshape it to the convention, keeping the original subject and the reverted SHA.

## Merges

Merge commits are usually left in git's default form and are typically excluded from changelog tooling, so you don't normally write one by hand. Repos using a rebase or squash workflow won't have merge commits to write at all. If a merge is in progress and the user wants a message, keep git's default unless they ask otherwise.

## SemVer mapping

- `fix`, `perf` → PATCH (x.y.**z**)
- `feat` → MINOR (x.**y**.0)
- any `BREAKING CHANGE` (or `!`) → MAJOR (**x**.0.0)
- `docs`, `style`, `refactor`, `test`, `build`, `ci`, `chore` → no release on their own

Tools such as semantic-release and release-please read these to decide the version bump and to generate the changelog — which is why an accurate type is worth the thought.
