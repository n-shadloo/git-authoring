# Release notes

Writing the release note for a version from what actually landed in it. This is mode 5, and it runs only when the user asks for release-note content — it is never part of a commit or pull-request request. Every entry traces to a real commit or a real hunk in the range; nothing is invented, and nothing user-visible is left out.

Mode 5 is **read-only by default**: it prints the note as a Markdown block in the conversation and stops. Writing a file happens only on an explicit request. Publishing a GitHub release belongs to mode 4 and needs its own explicit request on top of that.

## Contents

- Precondition: the repo must already version itself
- Establishing the range
- Matching the repo's existing style
- The canonical structure
- Content rules
- Output targets
- Publishing the release (mode 4 only)

## Precondition: the repo must already version itself

Mode 5 runs only if the project already versions itself. Look for any one of:

```bash
git fetch --tags                       # first, so the local list isn't stale
git tag --list --sort=-v:refname       # version-shaped tags
gh release list                        # published releases, when gh is available
```

— or a version field in the project: `package.json`, `pyproject.toml`, `Cargo.toml`, a `VERSION` file, `version:` in skill frontmatter, whatever the ecosystem uses.

**If none of these exist, say so plainly and stop.** Do not invent a versioning scheme, and do not create the repository's first-ever tag unprompted. Whether a project uses SemVer or CalVer, whether tags carry a `v` prefix, and where the canonical number lives are all the maintainer's decisions — and a first tag silently settles every one of them. Offer an unversioned summary of what changed since a point the user names instead.

## Establishing the range

Accuracy is the whole point of a release note, and accuracy starts with the range. Settle it before writing a word.

**1. Refresh the tags first.**

```bash
git fetch --tags
```

A local tag list goes stale silently, and it goes stale most often in exactly the repos that use this workflow: `gh release create` makes the tag on the remote, so a machine that hasn't fetched since the last release will not have it. Skipping this produces a note for the wrong range that looks entirely correct.

**2. Find the last released version** — the most recent version tag reachable from `HEAD`:

```bash
git describe --tags --abbrev=0                  # nearest tag on HEAD's history
git tag --sort=-v:refname --merged HEAD | head -1
```

Reachability matters: a tag cut on another branch is not the base for this branch's range.

**3. Cross-check it against the project's version field.** If the tag and the field disagree, **report the discrepancy and state which one you used as the range base.** Never silently pick one. The usual causes, each with a different answer:

- **The field is ahead of the tags** — the version was bumped in the tree but not yet released. Normal, and usually means the field names the release you are about to cut. The *tag* is still the range base.
- **The local tag list is stale** — fetch (step 1) and re-check before reporting anything; this is the most common false alarm.
- **A tag is ahead of the field** — a release was cut without bumping the field, or from another branch. Worth surfacing on its own; it usually wants fixing before the next release.

**4. Enumerate everything in the range, then read the diff.**

```bash
git log <base>..HEAD --oneline          # the subjects
git diff <base>..HEAD --stat            # what was touched
git diff <base>..HEAD                   # what actually changed
```

Two-dot is correct here: the base tag is an ancestor of `HEAD`, so this is the diff from the release to the tip. **Subjects alone are not enough.** A commit message can understate what landed, describe an intent the diff outgrew, or bundle a second change its subject never mentions. The diff is the source of truth, exactly as it is for a commit message.

**5. Every entry traces to a commit or a hunk.** Nothing invented, nothing user-visible omitted. If the range contains something you can't explain from the evidence, say so rather than filling the gap.

**6. Group by user-visible effect, not by file and not by commit.** Several commits that together deliver one change get one entry. One commit that delivered two unrelated user-visible changes gets two.

**7. Exclude internal churn that changes no behaviour** — formatting passes, test refactors, CI tweaks — unless it affects how someone uses, installs, or upgrades the project.

## Matching the repo's existing style

**Prior releases define the house style.** The canonical structure below is the fallback for a repository with no release history — where the two conflict, the repo wins. Detect:

- **Tag style** — prefix or no prefix (`v2.1.0` vs `2.1.0`), from `git tag --sort=-v:refname`.
- **Release title style** — from `gh release list` and `gh release view <last>` when `gh` is available; otherwise from any `RELEASE*.md` or `CHANGELOG.md` in the repo. Some projects title a release with the bare tag, others with a headline.
- **Everything below the title** — heading levels, section names and their order, bullet versus prose, whether entries end in a period, whether commit hashes or PR links are included, whether the note ends with a compare link and in what form.

Where the detected style departs from the canonical template, follow the repo and **note the deviation once, briefly**. Reformatting a project's release history to match a template is not an improvement; it's an inconsistency you introduced.

## The canonical structure

The fallback for a repo with no release history. Fixed section order:

```markdown
## <title, matching repo style>

<lead: one or two sentences, only if the release has a genuine theme>

### Breaking changes
### Added
### Changed
### Fixed
### Removed
### Deprecated
### Security
### Upgrade notes
### Full changelog
```

**Omit any section that has no real content.** Do not emit "N/A", "None", or a restatement to fill it — a note with two real sections beats one with eight padded ones. This is the same discipline that governs a commit body and a PR description.

`Breaking changes` comes first when present, because it is the thing a reader must not miss.

`Full changelog` carries the compare link. **Emit it only if both tags actually resolve** — check with `git rev-parse --verify --quiet <tag>` or `gh release view <tag>`. When you are writing the note *before* the release is tagged, the new tag does not exist yet, so omit the link rather than shipping a dead one.

## Content rules

**The rules that govern a commit body govern every line here** (`references/craft.md`). Restated for this context:

- **One line per entry**, describing the user-visible effect — what someone using the project can now do, can no longer do, or must change.
- **Every line traces to evidence** — a commit or a hunk in the range. If you can't point at one, cut the line.
- **No narration of the diff.** "Updated `SKILL.md` and `AGENTS.md`" tells a reader nothing they want; what changed for *them* is the note's whole job. (A per-file section is the exception where the repo's own history already uses one — style detection wins.)
- **No generic value claims.** "Improves maintainability", "better developer experience", "for consistency" fit every release ever cut, which is what makes them worthless.
- **No invented rationale.** If the reason a change landed isn't recoverable from the range or from what the user said, state the change and stop.
- **Name the bump and why**, when the project's style does — "Minor release, because the new mode is additive" is a real claim a reader can check; it also catches a mislabelled version before it ships.

## Output targets

1. **A Markdown block in the conversation** — the default, and read-only. Emit the note as a single fenced block the user can copy whole.
2. **A file in the repo** — only on an explicit request. Match whatever convention is already there: `RELEASE_v<version>.md`, a new entry at the top of `CHANGELOG.md`, or the project's own scheme. Don't introduce a second convention alongside an existing one.
3. **A published GitHub release** — mode 4 only, and only on an explicit request to publish. See below.

## Publishing the release (mode 4 only)

Publishing requires **mode 4 *and* an explicit request to publish**. It is never inferred — not from a request for a release note, not from approval of the note you just produced, and not from the presence of `gh`.

**Check availability first:**

```bash
command -v gh
gh auth status
```

If either fails, **fall back to the Markdown block in the conversation and say which check failed.** An unavailable or unauthenticated `gh` is never a reason to stop — the note is the deliverable; publishing is a convenience on top of it.

**Then publish:**

```bash
git tag -a v<version> -m "v<version>"           # tag name matches the detected style
git push origin v<version>
gh release create v<version> \
  --title "<style-matched title>" \
  --notes-file <path-to-notes.md>
```

Tag first and push it, so `gh` attaches the release to the tag you wrote rather than creating one of its own. Match the repo's prior pattern for latest/prerelease flags (`--latest`, `--prerelease`) rather than choosing one — `gh release view <last> --json isLatest,isPrerelease` shows what the project has been doing.

**Hard limits:**

- **Never overwrite or move an existing tag or release.** If either already exists for this version, stop and report it. A moved tag breaks every checkout that already has the old one.
- **Never `--force`, never delete a tag or release.**
- Mode 4's existing constraints are unchanged: still never opens pull requests, still never force-pushes, still never `git add -A`.
- **Publishing is never inferred.** It takes mode 4 and an explicit request, together.
