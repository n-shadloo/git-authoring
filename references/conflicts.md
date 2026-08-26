# Resolving a merge, rebase, or cherry-pick conflict

Mode 7. A conflict stops a sequence and leaves the repository in a named state. The work is to read both sides, decide from the intent of each change, prove the decision, and then end the sequence. Everything else in mode 7 is in `references/branching-and-history.md`.

## Contents

- The rule that governs the whole file
- Reading the state
- `ours` and `theirs`, and why a rebase inverts them
- Resolving
- Proving the resolution
- Ending the sequence
- rerere
- Never leave it half-finished

## The rule that governs the whole file

**NEVER resolve a conflict by choosing a side to make the sequence continue.** Both sides were written on purpose. Read each one, decide from what each change was for, and state what you decided and why. A resolution that was chosen to clear a blocker is a silent revert of somebody's work.

## Reading the state

```bash
git status
git diff --name-only --diff-filter=U
git diff
```

`git status` names the operation in progress and lists the unmerged paths. `git diff` on a conflicted file shows the combined diff of both sides.

A conflicted region has three parts:

```text
<<<<<<< HEAD
the content on the side named "ours"
=======
the content on the side named "theirs"
>>>>>>> <ref>
```

Read the `<ref>` on the closing marker. It names what "theirs" actually is, and it is the reliable answer when the sequence is a rebase.

## `ours` and `theirs`, and why a rebase inverts them

**The two labels swap between a merge and a rebase.** This is the most common wrong assumption in conflict work.

| Operation | `ours` is | `theirs` is |
|---|---|---|
| `git merge <branch>` | the branch you are on | `<branch>`, the one being merged in |
| `git rebase <base>` | `<base>`, the commits already replayed | your commit, the one being replayed |
| `git cherry-pick <sha>` | the branch you are on | the commit being picked |

A rebase replays your commits onto the base, so at each step the base is the side that is already in place. Your own work arrives as "theirs". **Never assume `ours` is your work.** Read the closing marker, or run `git status` and read which operation is in progress.

## Resolving

- **Edit the files.** Remove all three markers and leave the resolved content. `git add <path>` marks a path resolved.
- **Do not reach for a merge tool the repository does not have.** `git config merge.tool` reports whether one is configured. When it prints nothing, `git mergetool` opens whatever the machine happens to have, which is not reproducible. Resolve in the files.
- **`git checkout --ours -- <path>` and `--theirs` are the "pick a side" move.** They are correct only where one side is authoritative by nature: a binary file, a generated lockfile, a build artifact. Use them nowhere else, and say which side you took and why.
- **A conflict inside a generated file is a signal to regenerate it, not to merge it.** Resolve the source, then run the command that produces the file.
- When both sides changed the same logic for different reasons, the resolution is usually neither side verbatim. Write the version that satisfies both intents, and say that you did.

## Proving the resolution

**Build or test before the sequence completes.** A resolution that compiles is not a resolution that is correct, and a rebase that finishes on a broken commit buries the break in the middle of the history where bisect will find it later.

Run the repository's own check — the test command in its CI workflow, its build, or its type check — against the resolved tree, and read the output. Report what you ran. When no check exists, say that the resolution is unverified rather than implying it passed.

## Ending the sequence

| In progress | Continue | Abandon | Drop the current commit |
|---|---|---|---|
| merge | `git merge --continue` | `git merge --abort` | — |
| rebase | `git rebase --continue` | `git rebase --abort` | `git rebase --skip` |
| cherry-pick | `git cherry-pick --continue` | `git cherry-pick --abort` | `git cherry-pick --skip` |

- **`--abort` returns the repository to the state before the sequence started.** It is the safe exit, and it is the correct answer when the conflict turns out to need a decision the user has to make.
- **`--skip` discards the commit being replayed.** That is work loss, not a way past a hard conflict. Never run it to clear a blocker. Confirm first, and name the SHA that recovers the commit.

## rerere

**Use it only where the repository already enables it.** `git config rerere.enabled` reports whether it does. Never turn it on for the user.

Where it is on, git records each resolution and replays it the next time the same conflict appears. **A replayed resolution is applied without being shown.** A resolution that was wrong the first time is reapplied silently, so read the resolved file after rerere fires rather than trusting that the conflict was handled. `git rerere forget <path>` discards a recorded resolution for that path.

## Never leave it half-finished

**NEVER leave a rebase, merge, cherry-pick, or bisect part-way through without telling the user the exact state and the command that ends it.** `git status` names the operation. Report it, report the unmerged paths, and give the one command that continues, aborts, or resets.
