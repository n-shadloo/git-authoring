# git-commit-writer

A Claude skill that writes Conventional Commits messages from your staged changes. It reads the staged diff, works out what the change is and why it was made, and produces a properly structured commit — type, scope, subject, and, when the change needs it, a body and footer trailers.

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

## Install

The skill is a folder with a `SKILL.md`. Put it where your tool looks for skills.

Claude Code, one project:

```bash
git clone https://github.com/n-shadloo/git-commit-writer.git \
  .claude/skills/git-commit-writer
```

Claude Code, all your projects:

```bash
git clone https://github.com/n-shadloo/git-commit-writer.git \
  ~/.claude/skills/git-commit-writer
```

For claude.ai or the API, upload the folder as a custom skill in settings. The skill needs `git` on the path and a repository to run in; it has no other dependencies.

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

## Layout

```
git-commit-writer/
├── SKILL.md                          # workflow and the core format
├── references/
│   ├── conventional-commits.md       # full grammar, types, trailers
│   ├── examples.md                   # worked commits, incl. splits
│   ├── craft.md                      # writing subjects and bodies well
│   └── scopes-and-repos.md           # scopes; matching a repo's style
├── README.md
└── LICENSE
```

`SKILL.md` handles routine commits on its own; the reference files load only when a change calls for them.

## License

MIT. See [LICENSE](LICENSE).
