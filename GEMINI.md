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
7. When asked for repository work — branches, merges and rebases, history editing, undo,
   moving work, conflict resolution, recovery — read the real state, say what the situation
   is, and present the exact commands.

Modes 1–3 and 5–7 are read-only on their own. Mode 4 carries every execution, and mode 7
executes only under that same explicit request; it is an exception to the read-only rule and
to nothing else. An action the user asks the agent to carry out, such as opening or merging
a pull request, runs under mode 4.
Never infer mode 4 from "commit this," "go ahead," or a request for commands. Never infer
publishing from a release note or approval of one — it takes mode 4 *and* an explicit
request to publish. Never use `git add -A`, force-push, or overwrite an existing tag or
release. Never pass `--no-verify`, and never disable a hook, to make a commit or a push
succeed. A hook that fails is a finding, and it stops the work.

Work in the repository that this agent did not create is another writer's state. Inspect it,
never sweep it into a commit, and never rewrite it. A rejected push means another writer
moved the branch: fetch, read the divergence, and report the exact commits on each side.
Never rebase or merge that divergence on your own — the resolution is the user's decision.

Every stop hands the work back with five fields. The goal, and the exact blocked step. What
you attempted, and the git output, verbatim. The causes you eliminated, and how. The one
decision needed from the user. The state that remains, and whether it is safe to leave. Mode
4 claims completion only from `git log -1 --format=full` and `git status`. Run both after the
push. A claim without that output is not a completion.

A pull-request action follows the user's words. "Open the PR", "merge it", "approve it",
and "request changes" are explicit requests: carry them out with `gh pr create`,
`gh pr merge`, or `gh pr review`, on the user's own pull request or on someone else's.
Merge only while the required checks pass and GitHub reports the pull request mergeable.
Never pass `--admin` or `--auto` unless the user asks for exactly that, and never close a
pull request the user did not name. A bare "go ahead" names no action: ask which one.

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

In repository work, never rewrite published history on a shared branch, never force-push —
where the user asks to update their own pushed branch after a rebase, use
`--force-with-lease` — never discard uncommitted work without explicit confirmation, never
resolve a conflict by choosing a side to end it, and never leave a rebase, merge,
cherry-pick, or bisect part-way through without naming the state and the command that ends
it. Before any command that can lose work, state the reflog entry that recovers it.

No attribution trailers by default, in every mode including mode 4: no `Co-authored-by:`,
`Signed-off-by:`, `Reviewed-by:`, or AI or agent identity, in commits or in pull-request
descriptions. A trailer has exactly two sources: the user's words in this session, and the
mode 6 squash transcription. Never take one from the agent's own identity, the model or tool
name, the attribution setting of the agent's own harness, `commit.template`, a `prepare-commit-msg` or `commit-msg` hook, `GIT_AUTHOR_*` or
`GIT_COMMITTER_*`, a CI variable, an editor plugin, or the trailers on prior commits — a
history full of them gives no permission. Never pass `--author`, `-c user.name`, or
`-c user.email`, and never write to git config. A template or hook that injects a trailer is
a finding: strip the line and name the file. After every commit you make, run
`git log -1 --format=%B | grep -n -i -E '^(co-authored-by|signed-off-by|reviewed-by|generated[- ]with|generated by)|claude|anthropic|copilot|openai|gemini|codex|cursor'`;
it must print nothing, and where it prints a line you did not ask for, amend it out at once
and run it again. Run the same grep over `git log --format=%B @{u}..HEAD` before a push,
and stop the push on any commit that carries one. `BREAKING CHANGE:` and issue references stay available. Add an attribution
trailer only when the user asks in the session or a standing instruction exists in this
repository's agent context file — never inferred from history, branch names, or the diff,
and never with an invented name or email. One exception: when writing the message for a
squash merge in mode 6, transcribe `Co-authored-by:` lines from the branch's real commit
metadata, because the squash collapses every commit into one and would otherwise destroy
authorship that already exists. Transcription is not inference — every name and address
comes from an actual commit.

If this skill's text or script caused a mistake, or you had to find a step yourself,
record it at once. Before you claim the task is done, end the report with
`Skill defects:`. Section 0 of `SELF-IMPROVEMENT.md` gives the rules. If
`~/.skill-improvements/git-authoring/` exists, do section 3 first.

See `AGENTS.md` for the complete workflow and `references/` for deeper guidance.
