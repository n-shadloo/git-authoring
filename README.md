# git-authoring

A commit- and pull-request authoring skill for AI coding agents — built for Claude, and portable to Codex, Cursor, and Gemini CLI. By default it reads the staged diff, works out what the change is and why it was made, and produces a properly structured commit — type, scope, subject, and, when the change needs it, a body and footer trailers. When you ask, it also picks which unstaged files belong together as one coherent commit, and writes a complete pull-request title and description from what changed on the branch.

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
- On request, chooses which unstaged files belong together as one coherent commit before committing.
- On request, writes a complete pull-request title and description — summary, what changed, testing, breaking changes — from the branch's diff against its base.
- Never adds AI attribution, and never stages, commits, or opens a PR without your go-ahead — never by staging things you didn't stage.

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

The skill has one default and two on-request modes.

### Write a commit for staged changes — the default

Stage what you want to commit, then ask — or just tell the agent you're about to commit:

```
> write a commit message for my staged changes
```

A worked example:

```
$ git add users/serializers.py users/tests/test_serializers.py
> commit this

fix(serializers): handle null profile in user payload

A user without a profile row raised AttributeError during
serialization. Return an empty profile object so the endpoint responds
200 with nulls instead of 500.

Closes #517
```

If the staged changes turn out to cover more than one thing, it says so and hands you a sequence of `git add` / `git commit` commands to split them instead of writing one muddled message. It commits only when you ask, and only the set you staged.

### Ask it to choose which files to stage, then commit — on request

When you have a pile of unstaged work and want the agent to decide what belongs together, ask for it explicitly:

```
> choose the files that belong in one commit and commit them
```

It reads the unstaged changes and selects the files that form one coherent change — related work only, not everything, not an arbitrary mix — then proposes the exact `git add …` plus the commit. If several unrelated changes are in flight, it stages the most coherent group first and points out the rest. Nothing is staged or committed until you confirm.

### Ask for a pull-request title and description — on request

When the branch is ready to open, ask for PR content explicitly:

```
> write a PR title and description for this branch
```

It detects the base branch (main/master), reads the branch's commits and its diff against that base, and writes a complete PR as plain Markdown: a strong title and a description with a summary, what changed, testing, and any breaking changes — plus a reviewer/testing checklist where it helps. It verifies what it can from the history and diff, and never opens the PR for you: you get the Markdown to paste, or a `gh pr create …` command to run yourself.

The file-selection and pull-request modes run only when you ask for them. If you never mention them, the default commit flow behaves exactly as before.

The same conventions apply in Codex, Cursor, and Gemini. Codex and Cursor read this as a skill, just like Claude; Gemini reads `GEMINI.md`. Once the files are in place, stage your changes and ask your agent to commit — it follows the same rules.

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
