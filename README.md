# git-commit-writer

A commit-message skill for AI coding agents — built for Claude, and portable to Codex, Cursor, and Gemini CLI. It reads the staged diff, works out what the change is and why it was made, and produces a properly structured commit — type, scope, subject, and, when the change needs it, a body and footer trailers.

## Why

Most commit-message helpers stop at the format: they slap a `type:` on the front and call it done. The format is the easy part. The hard part is choosing the right type, writing a subject that actually says what changed, and using the body to record *why* — the context the diff can't show and that you'll want back six months later when `git blame` drops you on a line. This skill treats the format as table stakes and spends its effort on intent.

It defaults to Conventional Commits (imperative subject, `type(scope): …`, breaking-change notation, issue and co-author trailers) and enforces that discipline consistently. It also reads the repository's history first, so if a project uses a different convention it matches that instead of imposing one.

## What it does

- Reads the staged diff — not just filenames — to infer the change and its intent.
- Picks an accurate Conventional Commit type and a sensible scope.
- Writes an imperative subject within length limits, and a body explaining the *why* when the change isn't self-evident.
- Notates breaking changes with `!` and a `BREAKING CHANGE:` footer.
- Adds trailers: `Closes #123`, `Co-authored-by:`, `Signed-off-by:`, and the rest.
- Tells you when staged changes should be split into separate commits, and gives you the commands to do it.
- Commits when you ask — never before, and never by staging things you didn't stage.

## Works with

Claude, Codex, and Cursor all read this as an Agent Skill: `SKILL.md` plus the
`references/` folder, loaded on demand so deeper guidance is pulled in only when a
commit needs it. The skill format is a shared standard across those three tools, so
the same folder works in each without changes.

Gemini CLI doesn't read skills — it reads `GEMINI.md` at the repo root, which carries
the same conventions. A self-contained `AGENTS.md` at the root is also provided as
always-on project context for any agent that reads it.

## Install

The repo is a folder of plain-Markdown instruction files. Claude, Codex, and Cursor read it as a skill — `SKILL.md` plus the `references/` folder, loaded on demand. Gemini reads a single always-on file instead. Nothing to build; no dependencies beyond `git`.

### Claude

One project:

```bash
git clone https://github.com/n-shadloo/git-commit-writer.git \
  .claude/skills/git-commit-writer
```

All your projects:

```bash
git clone https://github.com/n-shadloo/git-commit-writer.git \
  ~/.claude/skills/git-commit-writer
```

For claude.ai or the API, upload the folder as a custom skill in settings.

### Codex

Codex reads the same `SKILL.md` skill format. Copy the folder into its skills directory — the `references/` come along and load on demand.

One project:

```bash
git clone https://github.com/n-shadloo/git-commit-writer.git \
  .codex/skills/git-commit-writer
```

All your projects:

```bash
git clone https://github.com/n-shadloo/git-commit-writer.git \
  ~/.codex/skills/git-commit-writer
```

Codex loads it automatically when a commit task matches, or invoke it explicitly with `/skills` or `$git-commit-writer`. (Some Codex versions also scan `.agents/skills/` from the working directory up to the repo root — either location works.)

For always-on context instead of, or alongside, the skill, drop `AGENTS.md` in your repo root, or `~/.codex/AGENTS.md` to apply the conventions to every project.

### Cursor

Cursor also reads the `SKILL.md` skill format, so the same clone works:

```bash
git clone https://github.com/n-shadloo/git-commit-writer.git \
  .cursor/skills/git-commit-writer
```

Cursor additionally reads `AGENTS.md` at the repo root and the bundled `.cursor/rules/git-commits.mdc` rule — both point at the same guidance, so having the repo checked out is enough.

### Gemini CLI

Gemini doesn't read skills; it reads `GEMINI.md`.

- **Per project:** copy `GEMINI.md` into the repo root.
- **All projects:** copy it to `~/.gemini/GEMINI.md`.

`GEMINI.md` carries the same conventions and points at `AGENTS.md` for the full detail.

The only requirement anywhere is `git` on your path and a repository to run in.

## Use

Stage what you want to commit, then ask:

```
> write a commit message for my staged changes
```

or just tell Claude you're about to commit. A worked example:

```
$ git add users/serializers.py users/tests/test_serializers.py
> commit this

fix(serializers): handle null profile in user payload

A user without a profile row raised AttributeError during
serialization. Return an empty profile object so the endpoint responds
200 with nulls instead of 500.

Closes #517
```

If the staged changes turn out to cover more than one thing, it says so and hands you a sequence of `git add` / `git commit` commands to split them instead of writing one muddled message.

The same conventions apply in Codex, Cursor, and Gemini. Codex and Cursor read this as a skill, just like Claude; Gemini reads `GEMINI.md`. Once the files are in place, stage your changes and ask your agent to commit — it follows the same rules.

## Layout

```
git-commit-writer/
├── SKILL.md                          # skill: workflow and the core format
├── AGENTS.md                         # always-on layer; source for the pointers below
├── GEMINI.md                         # Gemini CLI
├── .cursor/
│   └── rules/
│       └── git-commits.mdc           # Cursor rule (points to AGENTS.md)
├── references/
│   ├── conventional-commits.md       # full grammar, types, trailers
│   ├── examples.md                   # worked commits, incl. splits
│   ├── craft.md                      # writing subjects and bodies well
│   └── scopes-and-repos.md           # scopes; matching a repo's style
├── README.md
└── LICENSE
```

Claude, Codex, and Cursor load `SKILL.md` and pull in the reference files only when a change calls for them; `GEMINI.md` and `AGENTS.md` carry the same guidance for agents that read a single always-on file.

## License

MIT. See [LICENSE](LICENSE).
