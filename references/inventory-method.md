# Sweeping for the other copies

When one duplicated value is found, the copy that was reported is rarely the only one and
rarely the worst. This is how to find the rest without turning it into a week.

## What to search for

Search for the *values*, not the names. A copy is usually spelled differently from its
original — that is why it was never found by searching for the constant's name.

Categories worth a pass every time:

- **Option lists rendered in a UI** that mirror a backend union: pickers, dropdowns, filter
  chips, sort orders, anything with a fixed set of choices.
- **Status / state / kind vocabularies**: session states, turn outcomes, event kinds,
  dispositions. These mirror across an API boundary as types and drift silently, because a
  missing member usually means "this case is ignored" rather than a crash.
- **Defaults that also appear as a list member**: the default model, the default effort. A
  default is a second copy of one element and goes stale on its own.
- **Limits, timeouts, intervals and thresholds** written in more than one layer — especially
  one in validation and one in the input's `maxLength`.
- **Anything with a version or generation in its name**, which is what changed last time.

## What to record for each finding

| Column | Why it matters |
| --- | --- |
| What the value is, in one line | So a reader can judge relevance without opening files |
| Every copy, with file and line | "It's in a couple of places" is how one gets missed |
| **Do the copies agree today, element by element** | This is the finding. Compare literally; do not infer from the fact that both lists "look current" |
| Which copy is authoritative | Usually the owner/backend one; name it so the fix has a direction |

Order the report with **actual disagreements first** — those are live bugs, not tidiness.
Agreeing-but-duplicated entries are real findings too, but they are debt rather than defects
and should not bury the ones that are already wrong.

## Deciding what to fix now

Fix in this change:
- Anything that **already disagrees**. It is a bug by definition.
- Anything where the duplicate **changes behaviour rather than display** — a default that
  selects a retired model is worse than a label showing an old name, and is invisible.

List, with a recommendation, rather than fixing now:
- Copies that agree and whose single-sourcing crosses an ownership or layering boundary you
  would have to redesign. Say what the fix is and what it costs.
- Cases where the duplication might be **deliberate** — two vocabularies that overlap but are
  genuinely different sets. Say so and ask, rather than "fixing" a distinction someone meant.

## Do not

- Do not deduplicate code that merely looks alike. DRY is about knowledge; merging two
  similar functions that answer to different owners makes the next change harder, not easier.
- Do not widen a lint's pattern until its exception list is empty. The exceptions are the
  documentation.
- Do not claim an enforcement works without having watched it reject something.
