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

**Read the diff before writing a body.** Every claim has to trace to a specific hunk, to an issue the user referenced, or to something the user said in the session. A body assembled from filenames is guesswork, and guesswork in the permanent record is worse than silence.

**Most commits should carry no body at all.** A self-explanatory change gets a subject line and nothing else. There is no minimum length, no bullet quota, and no obligation to fill the space — omitting the body is a correct and common output, not a gap. If you cannot state a real reason from the evidence, omit it. Never invent rationale, motivation, or impact in order to have something to write: an invented reason doesn't just waste the reader's time, it misleads.

The cost of padding compounds. A body that restates the diff teaches the reader that bodies in this repository carry nothing — so they start skipping them, and the one commit that genuinely needed explaining gets skipped along with the rest.

**A body earns its place only when it adds one of these:**

- the problem or trigger that made the change necessary
- a non-obvious decision, plus the alternative you rejected and why
- a consequence a reader would not predict from reading the diff
- migration, operational, or compatibility impact
- a reference the diff cannot carry — an issue ID, a spec link, an incident

**The test to apply to every line before keeping it:** *could a reviewer recover this from `git show` alone?* If yes, cut it. What survives is the commit's actual contribution to the record.

Prefer fewer, denser lines over more, thinner ones. Keep it wrapped at ~72 characters, separated from the subject by a blank line.

## The anti-pattern catalogue

Common ways commits go wrong, and the fix.

**Vague subject** — "update code", "fix stuff", "changes", "wip", "misc". These tell a future reader nothing. Name the actual change: what, and where.

**"how" in the subject instead of "what"** — "fix(api): change the loop to a comprehension" describes the edit, not the effect. Prefer the outcome: "perf(api): speed up the export by batching rows".

**Mixing unrelated changes** — a subject with "and" in it ("fix login and update deps") is usually two commits wearing one message. Split them (`references/examples.md`).

**Type that doesn't match the diff** — `fix` for a change that adds a feature, or `refactor` for a change that alters behaviour. The type feeds versioning and changelogs; a wrong type ships the wrong release. Pick the type the diff actually supports.

**Past tense or narrating yourself** — "added a test", "I refactored the parser". Use the imperative: "add a test", "refactor the parser". (A repo whose history is consistently past-tense, like Django's, is the exception — match it.)

**Body that repeats the subject** — "fix(auth): prevent redirect loop on expired token" followed by "This fixes a redirect loop that occurred when the token had expired." The reader just read that sentence. Drop the body, or replace it with the *why*.

**Body that narrates the diff** — "Updated the serializer, then the view, then added a test", or a list of the files touched. `git show` and the Files-changed tab already carry that, more precisely than prose can. Say why the change happened, or say nothing.

**Generic value claims** — "improves maintainability", "enhances readability", "better developer experience", "for consistency", "for clarity". They're unfalsifiable and they fit every commit ever written, which is exactly what makes them worthless. If the change really did remove an inconsistency, name the inconsistency; if you can't name it, cut the line.

**Preambles** — "This commit…", "In this change…", "This PR…". The reader knows what they're looking at. Open with the substance.

**Boilerplate section headers on a short commit** — "Summary", "Changes", "Testing" bolted onto a three-line message. Structure that carries nothing is worse than no structure; save headings for a body long enough to need navigating.

**A period on the subject, or a paragraph as the subject** — the subject is one short line. Detail belongs in the body.

**Invented metadata** — a `Reviewed-by` for a review that didn't happen, a `Closes #NN` for an unrelated issue. Only include footers that are true.
