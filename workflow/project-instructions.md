# Project instructions (paste into Claude Code Projects)

Paste everything inside the box below into Project settings > Memory > Project instructions. Replace the two placeholders first. Limit: 16,000 characters.

```
This project builds REPLACE_WITH_APP_NAME in the REPLACE_WITH_REPO repository. Mohannad is the architect and decides every design question. Every thread follows CLAUDE.md, docs/, and docs/decisions.md in the repository.

How we work
- Propose threads and wait for my go-ahead before starting any. Show each proposal as: title, scope, numbered acceptance criteria, files it touches, and what it depends on.
- Run at most 3 threads at a time unless I say otherwise. Threads that touch the same files run one after the other.
- Each feature has a milestone branch named milestone/<feature-slug>, which I create and which already holds the feature's docs. Branch from that milestone branch and open one pull request per thread into it. If the milestone branch doesn't exist yet, stop and ask me to create it.
- Open pull requests as normal pull requests, never as drafts.
- Put the thread's numbered acceptance criteria in the pull request description, with how each one is met.
- Before calling work done, run the test and lint commands in CLAUDE.md and paste the summary lines in your final message.
- Push once when the work is ready, not after every commit: each push to an open pull request starts a full re-review. When fixing review comments, make all the fixes, then push once.
- The PR reviewer may comment on your pull request. Address its numbered fixes; don't argue with it.

Never
- Branch from, target, push to, or merge into main. Only I touch main.
- Merge any pull request. Approved pull requests are merged automatically.
- Tell me a pull request is "ready for me to merge" because CI is green. Pull requests merge only through the reviewer-approved label and the merge workflow. The one exception is a pull request that changes .github/: say so plainly, because I merge those myself.
- Add or remove the reviewer-approved or changes-requested labels. Only the reviewer sets them.
- Force push.
- Change files in .github/, .claude/, workflow/ or docs/ unless I ask in that thread.
- Add Co-authored-by lines, "Generated with Claude Code" lines, or session links to commits or pull request descriptions.

When something is missing or unclear
- If a design choice isn't answered by docs/ or docs/decisions.md, don't guess. Open the pull request (not as a draft), describe the question with 2 or 3 options, add the needs-decision label, and wait for me. When I answer: build the answer, remove the needs-decision label, then push. Removing the label before pushing is what lets the reviewer review it.
- If you can't reach something you need (a repository, secret, API, or tool), say exactly what's missing in your first message and stop. Don't substitute, mock, or guess.
```

## Project settings to change after creating the project

- General > Thread model: Sonnet 5.5, Thread effort: Medium. (The default is Opus at high effort, which uses your plan's limits fastest, even on Max. Ask for Opus in a single task when it really needs it.)
- General > Coordinator model: keep the default, or Sonnet 5.5 to save more.
- Environment > Project repositories: only this app's repository.
