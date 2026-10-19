# Chapter 19 — Real-World Scenario Simulator

Every previous chapter taught you one tool in isolation. Real work never comes in isolation — it's a messy week where five of these tools get used back to back, under time pressure, with other humans involved. This chapter is one continuous story, Monday to Friday, following a developer (call them Alex) on a small team building a task-management app. Every command shown is real and would actually run as written. Follow along in your own playground repo if you want the full effect.

---

## Monday — Starting a feature

Alex is assigned: "Add the ability to archive a task." First step, always: branch off a fresh `main`.

```bash
git switch main
git pull
git switch -c feature/archive-task
```

A few hours of work later, a natural checkpoint:
```bash
git status
git add src/models/task.js src/api/archive.js
git commit -m "Add archive endpoint and task model field"
```

By end of day, the UI button is stubbed in too:
```bash
git add src/components/TaskCard.jsx
git commit -m "Add archive button to task card (not yet wired up)"
git push -u origin feature/archive-task
```

---

## Tuesday — A conflict shows up

Alex continues, wiring the button to the API. But first, good habit: sync with `main` in case anything changed.

```bash
git fetch origin
git log --oneline main..origin/main
```
```
3f9a2c1 Rename Task.status to Task.state (breaking change across the model)
```
A teammate renamed a field Alex's branch also touches. Rather than wait for this to become a bigger problem at PR time, Alex rebases now, while the divergence is still small (exactly the "pull/rebase frequently" advice from [Chapter 6.10](06-merge-conflicts-mastery.md#610-preventing-conflicts-in-the-first-place)):

```bash
git rebase origin/main
```
```
CONFLICT (content): Merge conflict in src/models/task.js
error: could not apply 7e1a2b3... Add archive endpoint and task model field
```
Opening the file:
```javascript
<<<<<<< HEAD
  status: "active",
=======
  archived: false,
  status: "active",
>>>>>>> 7e1a2b3 (Add archive endpoint and task model field)
```
Wait — `HEAD` here, mid-rebase, actually refers to `main`'s incoming state (recall [Chapter 6.5](06-merge-conflicts-mastery.md#65---ours-and---theirs-the-fast-escape-hatch-for-whole-files)'s warning that "ours"/"theirs" flip meaning during a rebase). Alex reads both sides carefully rather than guessing, and hand-merges:
```javascript
  archived: false,
  state: "active",
```
```bash
git add src/models/task.js
git rebase --continue
```
One more small conflict appears in the second commit (same root cause), resolved the same way. Then:
```bash
git push --force-with-lease
```
Force-with-lease, not bare `--force` — [Chapter 15.4](15-rewriting-history-safely.md#154-force-push-safety---force-vs---force-with-lease-vs---force-if-includes) — because the rebase rewrote SHAs on a branch that's already on the remote, and Alex is the only one who should be touching it, but the safety check costs nothing.

---

## Wednesday — Code review feedback

Alex opens the PR:
```bash
gh pr create --title "Add task archiving" --body "Closes #91"
```
CI runs automatically ([Chapter 17](17-github-actions-and-automation.md)) and passes. A reviewer leaves two inline comments: one **suggested change** (a one-line naming fix, applied with a single click, which lands as a new commit on Alex's branch automatically) and one **"Request changes"**: *"This should be a soft-delete, not a hard state flip — can you add an `archivedAt` timestamp too?"*

Alex agrees, makes the change locally:
```bash
git pull                      # picks up the suggested-change commit GitHub just added
# ... adds archivedAt field ...
git add src/models/task.js
git commit -m "Add archivedAt timestamp per review feedback"
git push
```
The reviewer approves. Because this team's repo policy (Chapter 10.5) is **squash merge only**, none of this branch's slightly messy 5-commit history will show up on `main` — it'll flatten into one clean commit at merge time, so Alex doesn't even bother cleaning it up further first.

---

## Wednesday afternoon — Interrupted by a production incident

Mid-review-wait, a Slack message: *"Production is failing to load any task older than 30 days — critical, need a fix NOW."* Alex's working directory has no uncommitted mess right now (everything from the archive feature is already committed and pushed), so this is the easy case:

```bash
git switch main
git pull
git switch -c hotfix/old-task-loading-crash
```
Quick investigation points to a specific bad commit from last week. Rather than dig through the diff by eye, Alex reaches for `bisect`:
```bash
git bisect start
git bisect bad HEAD
git bisect good v2.3.0
```
```
Bisecting: 34 revisions left to test after this (roughly 5 steps)
```
Five quick manual tests later:
```
9c4d1a0 is the first bad commit
```
It's an off-by-one in a date-filtering function — an easy fix once located.
```bash
git bisect reset
# ...apply the actual fix...
git add src/api/tasks.js
git commit -m "Fix off-by-one in 30-day task filter (regression from 9c4d1a0)"
git push -u origin hotfix/old-task-loading-crash
gh pr create --title "Hotfix: fix crash loading tasks older than 30 days" --body "Fixes #104"
```
Given the severity, a teammate reviews within minutes and it's merged straight to `main`, triggering an automatic deploy ([Chapter 17.6](17-github-actions-and-automation.md#176-a-realistic-deploy-on-merge-example)).

---

## Thursday — Back to the archive feature, plus a tag

The archive PR gets its final approval and is squash-merged. Alex deletes the now-fully-merged branch:
```bash
git switch main
git pull
git branch -d feature/archive-task     # -d (lowercase) is happy to delete, since it IS fully merged
```
This happens to be the team's Thursday release day. Someone tags the release:
```bash
git tag -a v2.4.0 -m "Add task archiving, fix 30-day loading crash"
git push origin v2.4.0
gh release create v2.4.0 --generate-notes
```

---

## Friday — Cleaning up, and one near-miss

Alex, doing some end-of-week tidying, runs:
```bash
git branch --merged main
```
to find local branches safe to delete, and among them notices `experiment/websocket-sync` — a throwaway spike from three weeks ago. Alex deletes it:
```bash
git branch -D experiment/websocket-sync
```
...and immediately remembers one useful snippet from that spike was never actually merged anywhere. A small moment of panic, resolved in five seconds thanks to Chapter 11:
```bash
git reflog | grep websocket
7f3e9a1 HEAD@{23}: checkout: moving from experiment/websocket-sync to main
git switch -c experiment/websocket-sync-recovered 7f3e9a1
```
Fully recovered. Alex copies the one useful function into the real codebase properly (with a real commit and PR) and this time deletes the experimental branch for good, without regret.

---

## What this week actually demonstrated

| Day | Tools used | Chapter |
|---|---|---|
| Monday | branching, staging, committing | 3, 4 |
| Tuesday | fetch, rebase, conflict resolution, force-with-lease | 6, 7, 8, 15 |
| Wednesday | PR review cycle, squash merge | 10 |
| Wednesday (incident) | bisect, hotfix branching | 12, 16 |
| Thursday | branch cleanup, tags, releases | 4, 13 |
| Friday | reflog recovery | 11 |

Nothing in this week was exotic — this is genuinely ordinary professional work. What made Alex look competent wasn't knowing rare tricks; it was **never once panicking**, because every situation mapped cleanly onto a tool covered earlier in this course. That's the actual target this whole course has been aiming at.

Next: [Chapter 20 — Best Practices & Etiquette](20-best-practices-and-etiquette.md).
