# Decisions log

Every agent reads this before making a design choice. Only the design partner edits this file, and only to record a decision Mohannad made.

It stays short on purpose, because every thread and every review reads it:
- **Rules in force:** one line per decision that still applies, newest first. This is what to follow.
- **Full entries for the current milestone:** the question, options and reasoning for decisions made since the last milestone was merged.

When a milestone is finished, the design partner's "milestone close" session moves its full entries to `docs/decisions-archive/<slug>.md` and keeps their one-line rules here. D-numbers never change. Open an archive file only when you need an old decision's reasoning or the options that were turned down.

## Rules in force

- D-000: Agents build in threads that open pull requests into milestone branches; only Mohannad merges into main. See workflow/README.md. (setup)

## Full entries for the current milestone

Format for each entry, newest first:

```
## D-001: Short title (YYYY-MM-DD)
Question: what was asked, and where it came from (issue or PR number)
Options considered: A, B, C
Decision: what Mohannad chose
Why: his reason, in a sentence or two
Applies to: which parts of the app this rule covers
```
