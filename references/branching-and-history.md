# Branch work, history editing, and recovery

Mode 7. These are the operations an engineer runs on a working repository between commits. The mode is read-only by default: read the real state, say what the situation is, and present the exact commands. It executes only under the same explicit autonomous request that mode 4 needs.

Conflict resolution has its own file: `references/conflicts.md`.

## Contents

- Before an operation that can lose work
- Branches
- Integration: merge or rebase
- Updating a feature branch against a moved base
- History editing on unpublished work
- Undo
- Moving work between branches
- Recovery
- Inspection

## Before an operation that can lose work

State the recovery path before the operation runs. After it fails is too late.

- **Name the reflog entry or the branch that recovers the current state.** `git reflog -5` prints the entries. Give the user the one that gets them back.
- **NEVER discard uncommitted work without explicit confirmation.** This covers `reset --hard`, `checkout -- <path>`, `restore` without `--staged`, `clean`, and `stash drop`.
- **Uncommitted work is not in the reflog.** The reflog records commits. A tracked change that was never committed has no entry, so `git stash push` before a destructive operation is the only thing that makes it recoverable.

## Branches

- **Read the naming convention, never impose one.** `git branch -a --sort=-committerdate | head -20` shows what this repository actually uses. Match the prefix, the separator, and the ticket form you find.
- **Uncommitted work follows a switch.** `git switch <branch>` carries a modified file across when the target branch does not conflict on it. The change then belongs to the wrong branch and looks intentional. Commit or stash first, and say which you did.
- **`git branch -d` refuses to delete unmerged work. `-D` does not.** Use `-d`. When it refuses, the refusal is the finding: report the unmerged commits with `git log <base>..<branch> --oneline` and ask before `-D`.
- **`git fetch --prune` removes remote-tracking refs, never remote branches.** It is safe. `git push origin --delete <branch>` deletes the branch on the remote, and the local reflog does not recover it — recovery needs whoever still holds the commit.
- Set tracking with `git push -u origin <branch>` on the first push, or `git branch --set-upstream-to=origin/<branch>` on a branch that already exists on the remote.

## Integration: merge or rebase

Two questions decide it, in this order.

1. **Is the branch published, and might another person hold it?** Published and shared means merge. A rebase rewrites the commits, so every other holder gets a divergence they did not cause.
2. **What does the base branch already do?** `git log --graph --oneline -20 <base>` answers it. Merge commits in the history mean the project merges. A flat history means it rebases. Match what you find.

- **`git merge --ff-only <branch>` fails rather than creating a merge commit you did not intend.** Use it when a fast-forward is what you mean.
- **A squash-merge collapses every commit on the branch into one and keeps only the author of the branch.** When the branch carries commits from more than one author, transcribe each one as a `Co-authored-by:` trailer from the real commit metadata. This is the mode 6 exception in `references/pr-review.md`, and it is the only attribution the skill adds unasked.

## Updating a feature branch against a moved base

```bash
git fetch origin
git log --oneline --left-right origin/<base>...HEAD
```

Read the divergence before you choose.

- **Your own unpublished branch:** `git rebase origin/<base>`. The history stays flat.
- **A shared branch:** `git merge origin/<base>`. Never rebase it.
- **Your own branch that is already pushed:** rebase, then `git push --force-with-lease`. **NEVER `--force`.** `--force-with-lease` refuses the push when the remote branch moved since your last fetch, so it protects the case where someone else pushed to your branch while you were rebasing. A plain `--force` overwrites their commits and reports success.

## History editing on unpublished work

The gate is absolute. **NEVER rewrite published history on a shared branch.** Rebase, amend, squash, and reset apply to unpublished work, or to the user's own feature branch after the user says the branch is theirs alone. `git branch -r --contains <sha>` shows whether a commit is published.

```bash
git rebase -i <base>
```

- `reword` changes a message. `squash` merges a commit into the one above it and keeps both messages. `fixup` merges and discards the message. `drop` removes the commit. Reordering the lines reorders the commits.
- **Autosquash is the safer path to a fixup.** `git commit --fixup=<sha>` while the work is fresh, then `git rebase -i --autosquash <base>`. The marks are already in place, so there is no line to mis-edit.
- **Splitting one commit into several:** mark it `edit`, then `git reset HEAD^` to return its changes to the working tree, then stage and commit each piece. The mode 2 rules on grouping apply to the pieces.
- A rebase that stops on a conflict is a half-finished operation. See `references/conflicts.md`.

## Undo

| Situation | Command | What it keeps |
|---|---|---|
| The last commit is unpushed and its message or content is wrong | `git commit --amend` | everything; the SHA changes |
| The commit is published | `git revert <sha>` | the whole history; adds a new commit |
| Undo commits, keep the changes staged | `git reset --soft <ref>` | the index and the working tree |
| Undo commits, keep the changes unstaged | `git reset --mixed <ref>` | the working tree only |
| Undo commits and discard the changes | `git reset --hard <ref>` | nothing in the working tree |

- **`--hard` is the reflex and it is the one that loses work.** Ask which of the three the user means. Confirm before `--hard`, and stash first.
- **`revert` is the only correct undo for published history.** A reset on a pushed branch produces a force-push, and the rule above forbids it.
- Restore one file's content from another point: `git restore --source=<rev> -- <path>`. Unstage without touching the file: `git restore --staged <path>`.

## Moving work between branches

- **Cherry-pick:** `git cherry-pick -x <sha>` records the source commit in the message. Use `-x` whenever the commit exists on another branch that stays.
- **Stash:** `git stash push -m "<what it is>"` — always with a message, because `git stash list` is unreadable without one. **`pop` deletes the stash on success and keeps it on conflict.** After a conflicted `pop`, read `git stash list` before assuming the entry is gone. `apply` never deletes, so prefer it when the work matters.
- **Work committed on the wrong branch:** create the correct branch at the current commit with `git branch <correct>`, then move the wrong branch back. The reset is destructive, so name the recovery SHA and confirm first.
- **Two branches at once:** `git worktree add ../<dir> <branch>` checks a second branch out in its own directory. Use it instead of repeated stash-and-switch when both branches must be present, for example to run one branch's tests against the other's output. `git worktree remove <dir>` ends it.

## Recovery

**`git reflog` is the first move after any loss.** Do not attempt a repair before reading it.

```bash
git reflog -20
```

| Lost | Recovery |
|---|---|
| A commit dropped by a rebase | find the SHA in `git reflog`, then `git branch rescue <sha>` |
| A deleted branch | `git reflog` for the last checkout of it, then `git branch <name> <sha>` |
| A bad reset | `git reset --hard HEAD@{1}` — destructive in turn, so confirm and stash first |
| A stash that was dropped | `git fsck --unreachable \| grep commit`, then `git stash apply <sha>` |

- **Recover to a new branch, never over the current one.** `git branch rescue <sha>` costs nothing and cannot make the loss worse.
- `git fsck --lost-found` is the last resort. It writes every unreachable object to `.git/lost-found`, and reading it is slow. Reach for it only when the reflog has no entry.

## Inspection

- **Find when a string entered or left the code:** `git log -S'<text>' -- <path>`. `-G'<regex>'` matches a pattern instead of a literal.
- **Follow a file across renames:** `git log --follow -- <path>`.
- **Read a range:** `<a>..<b>` is what `b` has and `a` does not. `<a>...<b>` from `git log` is what either has and both do not, and `--left-right` marks which side each commit is on.
- **Blame:** `git blame -L <start>,<end> -w -C -- <path>`. `-w` ignores whitespace changes and `-C` follows lines moved between files, so both cut through a reformat that would otherwise name the wrong commit.
- **Bisect:** `git bisect start`, `git bisect bad`, `git bisect good <sha>`, then `git bisect run <command>` when a command can decide. **`git bisect reset` ends it.** A repository left in a bisect is a half-finished operation, and the user must be told the command that ends it.
