# git-authoring

A commit- and pull-request authoring skill for AI coding agents — built for Claude, and portable to Codex, Cursor, and Gemini CLI. It has four request-selected modes: generate an exact commit command for staged changes; choose one coherent set of files and generate staging plus commit commands; write a pull-request title and description; or, only when explicitly told to do it all, stage, commit, and push autonomously.

## Why

Most commit-message helpers stop at the format: they slap a `type:` on the front and call it done. The format is the easy part. The hard part is choosing the right type, writing a subject that actually says what changed, and using the body to record *why* — the context the diff can't show and that you'll want back six months later when `git blame` drops you on a line. This skill treats the format as table stakes and spends its effort on intent — for pull requests as much as commits.

It defaults to Conventional Commits (imperative subject, `type(scope): …`, breaking-change notation, issue and co-author trailers) and enforces that discipline consistently. It also reads the repository's history first, so if a project uses a different convention it matches that instead of imposing one.

## What it does

- Reads the staged diff — not just filenames — to infer the change and its intent.
- Picks an accurate Conventional Commit type and a sensible scope.
- Writes an imperative subject within length limits, and a body explaining the *why* when the change isn't self-evident.
- Notates breaking changes with `!` and a `BREAKING CHANGE:` footer.
- Adds trailers: `Closes #123`, `Co-authored-by:`, `Signed-off-by:`, and the rest.
- Tells you when staged changes should be split into separate commits, and gives you the commands to do it.
- On request, chooses which unstaged files belong together as one coherent commit and gives you the exact staging and commit commands.
- On request, writes a complete pull-request title and description — summary, what changed, testing, breaking changes — from the branch's diff against its base.
- Keeps the first three modes read-only: it never stages, commits, pushes, or opens a PR, and never adds AI attribution.
- Runs staging, committing, and pushing itself only when you explicitly ask for the autonomous mode.

## Works with

Claude, Codex, and Cursor all read this as an Agent Skill: `SKILL.md` plus the
`references/` folder, loaded on demand so deeper guidance is pulled in only when a
commit or pull request needs it. The skill format is a shared standard across those
three tools, so the same folder works in each without changes.

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

The only requirement is `git` on your path and a Git repository to run in.

## Use

The skill has one default and three on-request modes. The first three are read-only: the agent inspects git and gives you commands or Markdown, but you run any operation yourself. Only the fourth mode lets the agent mutate git.

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

AI attribution is prohibited in modes 1–3. It is permitted but optional in mode 4; human co-author, reviewer, sign-off, and other trailers must always be true.

The autonomous mode is never inferred. Asking for a message or commands, saying "commit this," or approving commands with "go ahead" keeps the interaction read-only. Ask the agent explicitly to stage, commit, and push when you intend mode 4.

The same conventions apply in Codex, Cursor, and Gemini. Codex and Cursor read this as a skill, just like Claude; Gemini reads `GEMINI.md`. Once the files are in place, ask for a commit command or explicitly request the autonomous workflow — the same mode boundary applies everywhere.

## Layout

```
git-authoring/
├── SKILL.md                          # skill: workflow, commit + PR authoring
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
│   └── pull-requests.md              # PR titles and descriptions
├── README.md
└── LICENSE
```

Claude, Codex, and Cursor load `SKILL.md` and pull in the reference files only when a change calls for them; `GEMINI.md` and `AGENTS.md` carry the same guidance for agents that read a single always-on file.

## License

MIT. See [LICENSE](LICENSE).
