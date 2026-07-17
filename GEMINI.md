# Git authoring conventions

This repository's commit-message and pull-request rules live in `AGENTS.md` at the
repository root. Follow it in full when writing any git commit or opening a pull
request here: pick an accurate Conventional Commit type and scope, write an imperative
subject, add a body only when it explains *why*, notate breaking changes, and — when
asked — choose which unstaged files belong together as one coherent commit, or write a
complete PR title and description from the branch's diff against its base. Never mutate
git on your own: propose the exact `git add` / `git commit` / PR commands and run them
only when explicitly asked — never via `git add -A`. Never add AI attribution to a
commit or PR unless the user explicitly asks. See `AGENTS.md` for the complete workflow
and `references/` for deeper guidance.
