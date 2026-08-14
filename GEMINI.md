# Git authoring conventions

This repository's commit-message and pull-request rules live in `AGENTS.md` at the
repository root. Follow it in full and choose the mode from the user's request:

1. For already-staged changes, present the exact quoted-heredoc commit command.
2. When asked to choose files, present specific staging commands plus the heredoc
   commit command.
3. When asked for PR content, write the title and structured Markdown description.
4. Only when unmistakably told to do it all, stage, commit, and push autonomously — and,
   on a further explicit request, tag and publish a release.
5. When asked for release notes, establish the range since the last release, read what
   actually landed, and write the note as a Markdown block.
6. When asked to review an incoming pull request, read it through `gh`, work the decision
   through with the user, and write the review comment or the merge-commit message as a
   Markdown block for them to paste. Never approve, request changes, comment, or merge.

Modes 1–3, 5, and 6 are always read-only, even after a follow-up confirmation. Mode 4 is
the sole execution exception; it is an exception to the read-only rule and to nothing else.
Never infer mode 4 from "commit this," "go ahead," or a request for commands. Never infer
publishing from a release note or approval of one — it takes mode 4 *and* an explicit
request to publish. Never use `git add -A`, force-push, or overwrite an existing tag or
release.

Mode 4 does not extend to mode 6. Deciding to approve, reject, or merge someone else's pull
request always produces text the user pastes, never an action the agent takes — "approve
it" or "yes, merge" tells you what the block should say, not to run it. Never run
`gh pr review`, `gh pr comment`, `gh pr merge`, or `gh pr close`.

When reviewing a pull request, separate what actually blocks the merge — breaks the build
or an existing test, loses or corrupts data, opens a security hole, breaks a documented
contract without notating it, or does not do what the PR claims — from what is only a
suggestion, and calibrate the bar to the project: a small repo with no stated convention
and no CI is not a large one. Never manufacture a blocker, and when nothing blocks, say so
in one plain line before anything else. On a fork pull request, empty `gh pr checks` output
means the workflows have not run, not that they passed.

For a release note, fetch tags first, base the range on the last version tag reachable
from `HEAD`, and report any disagreement with the project's version field rather than
silently picking one. If the repository has no versioning at all, say so and stop rather
than creating its first tag. Match the style of prior releases over any template.

No attribution trailers by default, in every mode including mode 4: no `Co-authored-by:`,
`Signed-off-by:`, `Reviewed-by:`, or AI or agent identity, in commits or in pull-request
descriptions. `BREAKING CHANGE:` and issue references stay available. Add an attribution
trailer only when the user asks in the session or a standing instruction exists in this
repository's agent context file — never inferred from history, branch names, or the diff,
and never with an invented name or email. One exception: when writing the message for a
squash merge in mode 6, transcribe `Co-authored-by:` lines from the branch's real commit
metadata, because the squash collapses every commit into one and would otherwise destroy
authorship that already exists. Transcription is not inference — every name and address
comes from an actual commit.

See `AGENTS.md` for the complete workflow and `references/` for deeper guidance.
