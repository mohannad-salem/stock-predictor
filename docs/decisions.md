# Decisions log

Every agent reads this before making a design choice. Newest first. Only the design partner edits this file, and only to record a decision Mohannad made.

Format for each entry:

```
## D-001: Short title (YYYY-MM-DD)
Question: what was asked, and where it came from (issue or PR number)
Options considered: A, B, C
Decision: what Mohannad chose
Why: his reason, in a sentence or two
Applies to: which parts of the app this rule covers
```

## D-000: Workflow adopted (YYYY-MM-DD)
Question: how agents and Mohannad work together
Decision: design partner writes the docs; a Claude Code Project plans and runs threads that open pull requests into milestone branches; two reviewer routines label them; a GitHub workflow merges approved pull requests once CI passes. Mohannad approves plans and milestones, merges milestones into main, and approves any deploy if the app has one. See workflow/README.md.
Applies to: everything
