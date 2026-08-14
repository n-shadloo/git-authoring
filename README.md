# git-authoring

A commit-, pull-request-, release-note-, and PR-review authoring skill for AI coding agents — built for Claude, and portable to Codex, Cursor, and Gemini CLI. It has six request-selected modes: generate an exact commit command for staged changes; choose one coherent set of files and generate staging plus commit commands; write a pull-request title and description; stage, commit, and push autonomously when explicitly told to do it all; write the release note for a version from what actually landed since the last release; or review an incoming pull request and hand you the review comment or the squash-merge message to paste.

## Why

Most commit-message helpers stop at the format: they slap a `type:` on the front and call it done. The format is the easy part. The hard part is choosing the right type, writing a subject that actually says what changed, and using the body to record *why* — the context the diff can't show and that you'll want back six months later when `git blame` drops you on a line. This skill treats the format as table stakes and spends its effort on intent — for pull requests and release notes as much as commits.

It defaults to Conventional Commits (imperative subject, `type(scope): …`, breaking-change notation, issue references) and enforces that discipline consistently. It also reads the repository's history first, so if a project uses a different convention it matches that instead of imposing one.

## What it does

- Reads the staged diff — not just filenames — to infer the change and its intent.
- Picks an accurate Conventional Commit type and a sensible scope.
- Writes an imperative subject within length limits, and a body explaining the *why* when the change isn't self-evident.
- Notates breaking changes with `!` and a `BREAKING CHANGE:` footer.
- Adds issue references (`Closes #123`) — and, on request only, `Co-authored-by:`, `Signed-off-by:`, and the rest.
- Tells you when staged changes should be split into separate commits, and gives you the commands to do it.
- On request, chooses which unstaged files belong together as one coherent commit and gives you the exact staging and commit commands.
- On request, writes a complete pull-request title and description — summary, what changed, testing, breaking changes — from the branch's diff against its base.
- On request, writes the release note for a version from the real range since the last release, in the style your project's previous releases already use.
- On request, reviews an incoming pull request with you — reads its diff, checks, and existing comments, separates what actually blocks a merge from what's only a suggestion, and hands you the review comment or the squash-merge message to paste.
- Keeps every mode except the autonomous one read-only: it never stages, commits, pushes, opens a PR, tags, publishes a release, or approves, rejects, or merges a pull request.
- Adds no attribution trailers in any mode unless you ask for them — see [Attribution and trailers](#attribution-and-trailers).
- Runs staging, committing, and pushing itself only when you explicitly ask for the autonomous mode — and tags and publishes a release only on a further explicit ask.

## Works with

Claude, Codex, and Cursor all read this as an Agent Skill: `SKILL.md` plus the
`references/` folder, loaded on demand so deeper guidance is pulled in only when a
commit, pull request, or release note needs it. The skill format is a shared standard
across those three tools, so the same folder works in each without changes.

Gemini CLI doesn't read skills — it reads `GEMINI.md` at the repo root, which carries
the same conventions. A self-contained `AGENTS.md` at the root is also provided as
always-on project context for any agent that reads it.

## Install

The repo is a folder of plain-Markdown instruction files. Claude reads it as an
Agent Skill (`SKILL.md` plus the `references/` folder, loaded on demand).
OpenAI Codex CLI and Cursor reuse the same canonical content through their
native discovery mechanisms, while Gemini CLI reads a single `GEMINI.md`
context file. Nothing to build; no dependencies beyond `git`.

### Claude

One project:

```bash
git clone https://github.com/n-shadloo/git-authoring.git \
  .claude/skills/git-authoring
```

All your projects:

```bash
git clone https://github.com/n-shadloo/git-authoring.git \
  ~/.claude/skills/git-authoring
```

For claude.ai or the API, upload the folder as a custom skill in Settings.

### Codex CLI

Codex CLI discovers Agent Skills from the `.agents/skills/` directory and reads
`AGENTS.md` for always-on project context.

One project:

```bash
git clone https://github.com/n-shadloo/git-authoring.git \
  .agents/skills/git-authoring
```

All your projects:

```bash
git clone https://github.com/n-shadloo/git-authoring.git \
  ~/.agents/skills/git-authoring
```

Codex automatically discovers the skill when appropriate. `AGENTS.md` provides
project-wide conventions alongside the skill.

### Cursor

Cursor also supports Agent Skills, so the same clone works:

```bash
git clone https://github.com/n-shadloo/git-authoring.git \
  .cursor/skills/git-authoring
```

Cursor additionally reads `AGENTS.md` at the repository root and the bundled
`.cursor/rules/git-authoring.mdc` rule, which reinforces the same guidance.

### Gemini CLI

Gemini CLI doesn't read Agent Skills directly; it reads `GEMINI.md`.

- **Per project:** copy `GEMINI.md` into the repository root.
- **All projects:** copy it to `~/.gemini/GEMINI.md`.

`GEMINI.md` carries the same conventions and points at `AGENTS.md` for the full
detail.

The only requirement is `git` on your path and a Git repository to run in. [GitHub CLI](https://cli.github.com) (`gh`) is optional: release-note mode uses it to read your previous releases and match their style, and the autonomous mode needs it to publish one. Without it, the skill falls back to any `CHANGELOG.md` / `RELEASE*.md` in the repo for style and hands you the note as Markdown, telling you which check failed.

## Use

The skill has one default and five on-request modes. All of them are read-only except the autonomous one: the agent inspects git and gives you commands or Markdown, but you run any operation yourself. Only mode 4 lets the agent mutate git — and it never extends to GitHub review or merge actions.

### 1. Get a commit command for staged changes — the default

Stage what you want to commit, then ask — or just tell the agent you're about to commit:

```
> write a commit message for my staged changes
```

It reads the staged diff and returns the exact command for you to run:

```bash
git commit -F - <<'COMMIT_MSG'
fix(serializers): handle null profile in user payload

A user without a profile row raised AttributeError during serialization.
Return an empty profile object so the endpoint responds 200 with nulls
instead of 500.

Closes #517
COMMIT_MSG
```

If the staged changes cover more than one thing, it says so and hands you a sequence of specific staging and heredoc commit commands to split them instead of writing one muddled message. It never runs those commands for you in this mode.

### 2. Ask it to choose files and prepare the commands — on request

When you have a pile of unstaged work and want the agent to decide what belongs together, ask for it explicitly:

```
> choose the files that belong in one commit and give me the commands
```

It reads the staged and unstaged changes, selects the files that form one coherent change — related work only, not everything, not an arbitrary mix — and returns commands like these:

```bash
git add users/serializers.py users/tests/test_serializers.py
git commit -F - <<'COMMIT_MSG'
fix(serializers): handle null profile in user payload
COMMIT_MSG
```

If several unrelated changes are in flight, it selects the strongest coherent group and points out what it deliberately left out. You run both commands; the agent never stages or commits in this mode.

### 3. Ask for a pull-request title and description — on request

When the branch is ready to open, ask for PR content explicitly:

```
> write a PR title and description for this branch
```

It detects the base branch (main/master), reads the branch's commits and its diff against that base, and writes a complete PR as plain Markdown: a strong title and a description with a summary, what changed, testing, and any breaking changes — plus a reviewer/testing checklist where it helps. It verifies what it can from the history and diff, and never opens the PR for you: you get the Markdown to paste, or a `gh pr create …` command to run yourself.

### 4. Ask it to stage, commit, and push — explicit autonomous mode

Use unambiguous wording when you want the agent to perform the operations itself:

```
> stage, commit, and push this for me — do it all yourself
```

The agent inspects the staged and unstaged hunks, selects one coherent change, stages only its specific paths, verifies the staged diff, commits with the quoted-heredoc form, and pushes the current branch to its upstream. It never uses `git add -A` or force-pushes. If the remote is ambiguous, existing staged work conflicts with a safe grouping, or the push is rejected, it stops and reports the exact state instead of guessing.

On a further explicit ask, mode 4 will also tag and publish a GitHub release. It checks that `gh` is installed and authenticated first, and falls back to handing you the note as Markdown if either check fails. It never overwrites or moves an existing tag or release, never uses `--force`, and never deletes either — if the version is already tagged or released, it stops and tells you.

Mode 4 is an exception to the read-only rule and to nothing else. In particular it adds no attribution of its own — see [Attribution and trailers](#attribution-and-trailers).

The autonomous mode is never inferred. Asking for a message or commands, saying "commit this," or approving commands with "go ahead" keeps the interaction read-only. Ask the agent explicitly to stage, commit, and push when you intend mode 4.

### 5. Ask for the release note for a version — on request

When you're cutting a release, ask for the note:

```
> write the release notes for 2.3.0
```

It fetches tags, works out the range from the last version tag reachable from `HEAD`, reads the actual diff over that range — not just the commit subjects — and returns the note as a Markdown block you can paste:

```markdown
# v2.3.0

Minor release. Adds a fifth mode: the skill now writes the release note for a
version from what actually landed since the last release.

## What changed

**Release notes are grounded in the range, not the subjects.** The note is built
from `git diff <last-tag>..HEAD`, so a commit whose message understated what it
shipped doesn't understate it twice.

## Upgrade notes

Drop-in replacement for 2.2.0. The four existing modes are unchanged.
```

It matches your project's own release style — tag prefix, title shape, headings, section order — over any built-in template, and says so when the two differ. Sections with nothing real in them are dropped rather than filled with "None".

Two things it won't do. If the repository has no versioning at all — no version tags, no version field, no releases — it says so and stops rather than creating your first tag and silently deciding your scheme for you. And if the last tag disagrees with the version field in your project, it reports the discrepancy and tells you which one it used as the range base instead of quietly picking one.

Publishing is separate: the note lands in the conversation, and writing it to a file or cutting the actual GitHub release each take their own explicit ask (the latter via mode 4).

### 6. Ask it to review an incoming pull request — on request

This one is for the other side of the table: you're the maintainer, and someone else's branch is waiting on you.

```
> review PR 412 with me
```

It reads the PR through `gh` — the diff, the CI checks, the existing review threads, the linked issue, the commits and their authors, and your repo's merge method — then tells you what the branch does and, more usefully, sorts what it found into two buckets. **Blocking** means merging leaves you worse off than not merging: a broken build or test, data loss, a security hole, a broken contract, or a change that doesn't do what the PR says. **Everything else is a suggestion**, and a suggestion is never on its own a reason to hold up a merge.

That distinction is calibrated to your project. A small repo with no stated convention and no CI has a high bar for blocking — missing tests in a project with no test suite isn't a blocker, and neither is ignoring a rule you never wrote down. It won't manufacture an objection to make the review look thorough, and when nothing blocks, that's the first line you read.

Then you decide, and it writes exactly one Markdown block for the decision you made:

- **Something blocks it** — a review comment, verdict first, blocking items separated from optional ones, pointing at code and locations rather than at the contributor.
- **You're accepting** — the merge-commit message. On a squash merge this matters more than it looks: GitHub prefills the body with every branch commit concatenated together, and that's the commit that lands on your default branch and lives in `git log` forever. It gets written fresh from the diff instead, and `Co-authored-by:` lines are carried across from the branch's real commits so a squash doesn't erase a second contributor's credit.

You paste it into GitHub and click the button. The agent never approves, requests changes, comments, or merges — and "approve it" or "yes, merge" tells it what the block should say, not to run it. The autonomous mode doesn't extend here: mode 4 stages, commits, and pushes *your* work, never lands someone else's.

The same conventions apply in Codex, Cursor, and Gemini. Codex and Cursor read this as a skill, just like Claude; Gemini reads `GEMINI.md`. Once the files are in place, ask for a commit command, a release note, or explicitly request the autonomous workflow — the same mode boundary applies everywhere.

## Attribution and trailers

**The default is no attribution trailers at all.** No `Co-authored-by:`, no `Signed-off-by:`, no `Reviewed-by:`, and no AI or agent identity of any kind — not in commit messages, not in pull-request descriptions. A commit reads as your own work, authored by whatever `git config user.name` / `user.email` resolves to.

The reason is that a trailer is an assertion: that a particular person collaborated on the change, or reviewed it, or is certifying its origin under a DCO. That claim belongs to you, not to a tool guessing on your behalf. A wrong one is worse than none — it puts a name in permanent history that doesn't belong there.

**This applies to mode 4 too.** There is no autonomous-mode exception. Asking the agent to stage, commit, and push is a request to do the work, not a request to be credited for it, so an autonomous commit carries exactly the same trailers a mode-1 commit would.

**One exception, and only one.** When you squash-merge someone's pull request in mode 6, `Co-authored-by:` lines are carried across from the branch's real commits without you asking. A squash collapses every commit into one and credits only the PR author, so transcribing them preserves authorship that already exists rather than asserting anything new — every name and address comes from an actual commit.

**Never inferred.** The skill does not inspect history, branch names, or the diff to decide that someone should be credited — there is no contributor-detection step. A repository whose every commit carries `Signed-off-by` still gets no sign-off: convention detection matches subject shape, scope vocabulary, and tense, and stops there. And a named human only ever comes from a value you supply — the skill will not invent a name or an email address.

**Still included by default,** because neither is attribution:

- `BREAKING CHANGE:` — required by the Conventional Commits grammar.
- Issue references (`Refs: #123`, `Closes #123`) when you supply the issue, or when it's unambiguous from the branch name. It never guesses an issue number.

### Asking for a trailer on one commit

Say so in the session, in whatever words are natural:

```
> write the commit message and sign it off
> commit this and add Sara Ahmadi <sara@example.com> as co-author
> credit the agent on this one
```

You get the same message with the trailer block filled in:

```bash
git commit -F - <<'COMMIT_MSG'
fix(serializers): handle null profile in user payload

Co-authored-by: Sara Ahmadi <sara@example.com>
Closes #517
COMMIT_MSG
```

The request applies to that session's work, not to the repository — the next session starts from the default again.

### Making it persistent

For a standing preference, put it in the consuming repository's own agent context file — `AGENTS.md`, `CLAUDE.md`, or whatever your agent reads. The skill honours an instruction it finds there. One line is enough:

```markdown
Sign off every commit with `Signed-off-by:` using my git user.name and user.email.
```

That's the whole mechanism: an explicit ask in the session, or a standing line in your own context file. There is no config file and no flag to set.

### Supported on request

- `Co-authored-by:`
- `Signed-off-by:`
- `Reviewed-by:` (and the neighbouring `Acked-by:` / `Tested-by:`, `Reported-by:` / `Suggested-by:` / `Helped-by:`)
- Issue-reference footers
- AI or agent attribution, in whatever form you ask for

## Layout

```
git-authoring/
├── SKILL.md                          # skill: commit, PR, release, and review authoring
├── AGENTS.md                         # always-on layer; source for the pointers below
├── GEMINI.md                         # Gemini CLI
├── .cursor/
│   └── rules/
│       └── git-authoring.mdc         # Cursor rule (points to AGENTS.md)
├── references/
│   ├── conventional-commits.md       # full grammar, types, trailers
│   ├── examples.md                   # worked commits, incl. splits and staging
│   ├── craft.md                      # writing subjects and bodies well
│   ├── scopes-and-repos.md           # scopes; matching a repo's style
│   ├── pull-requests.md              # PR titles and descriptions
│   ├── release-notes.md              # release notes; tagging and publishing
│   └── pr-review.md                  # reviewing and landing incoming PRs
├── README.md
└── LICENSE
```

Claude, Codex, and Cursor load `SKILL.md` and pull in the reference files only when a change calls for them; `GEMINI.md` and `AGENTS.md` carry the same guidance for agents that read a single always-on file.

## License

MIT. See [LICENSE](LICENSE).
