# Copilot instructions

## Commit messages

When asked to write or suggest a git commit message, follow the conventions in
`AGENTS.md` at the repository root — accurate Conventional Commit `type(scope): subject`,
imperative subject, a body only when it explains *why* the change was made, correct
breaking-change notation (`!` and/or a `BREAKING CHANGE:` footer), and issue/co-author
trailers. Read the staged diff (`git diff --staged`) as the source of truth. If nothing
is staged, say so — do not stage everything. Commit only when explicitly asked.
See `AGENTS.md` for the full workflow.
