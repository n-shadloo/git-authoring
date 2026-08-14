# Git authoring conventions

This repository's commit-message and pull-request rules live in `AGENTS.md` at the
repository root. Follow it in full and choose the mode from the user's request:

1. For already-staged changes, present the exact quoted-heredoc commit command.
2. When asked to choose files, present specific staging commands plus the heredoc
   commit command.
3. When asked for PR content, write the title and structured Markdown description.
4. Only when unmistakably told to do it all, stage, commit, and push autonomously.

Modes 1–3 are always read-only, even after a follow-up confirmation. Mode 4 is the sole
execution exception; it is an exception to the read-only rule and to nothing else. Never
infer mode 4 from "commit this," "go ahead," or a request for commands. Never use
`git add -A` or force-push.

No attribution trailers by default, in every mode including mode 4: no `Co-authored-by:`,
`Signed-off-by:`, `Reviewed-by:`, or AI or agent identity, in commits or in pull-request
descriptions. `BREAKING CHANGE:` and issue references stay available. Add an attribution
trailer only when the user asks in the session or a standing instruction exists in this
repository's agent context file — never inferred from history, branch names, or the diff,
and never with an invented name or email. See `AGENTS.md` for the complete workflow and
`references/` for deeper guidance.
