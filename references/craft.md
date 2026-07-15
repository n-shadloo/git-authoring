# Writing commits that communicate

The format is the skeleton; this is the craft. A message can be perfectly Conventional and still be useless ("fix(api): fix bug"). What makes a commit worth reading later is that it captures intent — the thing the diff can't tell you.

## Contents

- The subject line
- The imperative-mood test
- The body: why, not how
- The anti-pattern catalogue

## The subject line

The subject is read far more often than the body — in `git log`, in blame, in PR lists, in changelogs. Make it carry the change on its own.

- **Say what changed, specifically.** "fix(auth): prevent redirect loop on expired token" tells the reader what happened. "fix(auth): fix login" does not.
- **Imperative mood.** "add", "remove", "fix" — not "added", "adds", "adding". This matches the verbs git itself uses ("Merge…", "Revert…") and reads as an instruction the commit carries out.
- **Length.** Aim for ≤ 50 characters and treat 72 as the hard ceiling. The limit is a forcing function for precision, and tools truncate long subjects.
- **No trailing period**, lowercase first letter (the common linter default). Consistency within a repo matters more than the specific choice.

## The imperative-mood test

A well-formed subject completes this sentence:

> If applied, this commit will **\<subject\>**.

"If applied, this commit will *prevent redirect loop on expired token*" reads correctly. "If applied, this commit will *fixed the login bug*" doesn't. If it won't complete the sentence, rewrite it.

## The body: why, not how

The diff already shows *how* the code changed. The body's job is everything the diff can't show:

- **Why the change was needed** — the bug's symptom, the missing capability, the constraint.
- **Why this approach** — alternatives you rejected and the reason, trade-offs you accepted.
- **Consequences** — side effects, follow-ups, anything the next person should know.

This is the highest-leverage part of a commit. Months later, `git blame` lands someone on a line and the body is the only record of *why* it's like that. Code review benefits too: the reasoning sits right next to the change.

Keep it wrapped at ~72 characters, separated from the subject by a blank line. And omit it entirely when the change speaks for itself — a forced body that restates the subject is noise.

## The anti-pattern catalogue

Common ways commits go wrong, and the fix.

**Vague subject** — "update code", "fix stuff", "changes", "wip", "misc". These tell a future reader nothing. Name the actual change: what, and where.

**"how" in the subject instead of "what"** — "fix(api): change the loop to a comprehension" describes the edit, not the effect. Prefer the outcome: "perf(api): speed up the export by batching rows".

**Mixing unrelated changes** — a subject with "and" in it ("fix login and update deps") is usually two commits wearing one message. Split them (`references/examples.md`).

**Type that doesn't match the diff** — `fix` for a change that adds a feature, or `refactor` for a change that alters behaviour. The type feeds versioning and changelogs; a wrong type ships the wrong release. Pick the type the diff actually supports.

**Past tense or narrating yourself** — "added a test", "I refactored the parser". Use the imperative: "add a test", "refactor the parser". (A repo whose history is consistently past-tense, like Django's, is the exception — match it.)

**Body that repeats the subject** — if the body just restates the subject as a full sentence, drop it. Bodies are for the *why*.

**A period on the subject, or a paragraph as the subject** — the subject is one short line. Detail belongs in the body.

**Invented metadata** — a `Reviewed-by` for a review that didn't happen, a `Closes #NN` for an unrelated issue. Only include footers that are true.
