# Chapter 12 — Stash, Cherry-pick & Bisect

Three tools that solve three very specific, very common real-world interruptions: needing to switch context suddenly, needing one specific commit somewhere else, and needing to find which of hundreds of commits broke something.

## 12.1 `git stash` — the junk drawer for work in progress

**Real-life scenario:** you're mid-way through a messy refactor — nothing compiles, nothing's ready to commit — when your manager says "drop everything, there's a critical bug on `main`, fix it now." You can't commit garbage, and you don't want to lose an hour of work either. `stash` takes your entire uncommitted state (staged and unstaged changes) and tucks it away, leaving your working directory clean, as if you'd just checked out a fresh copy of the last commit.

```bash
git stash                    # stash everything (tracked, modified files)
git stash push -m "wip: refactor auth"   # stash WITH a descriptive label — do this, always
git stash -u                 # also include untracked (new) files
git stash -a                 # also include ignored files (rare, but exists)
```
```bash
$ git stash list
stash@{0}: On feature/refactor: wip: refactor auth
stash@{1}: On main: wip: half-finished CSS
```

Now you're free:
```bash
git switch main
git switch -c hotfix/critical-bug
# ... fix the bug, commit, push, open PR ...
git switch feature/refactor
git stash pop                # re-apply the most recent stash AND remove it from the stash list
# — or —
git stash apply              # re-apply WITHOUT removing it from the list (in case you want it again elsewhere)
git stash drop stash@{0}     # manually delete a stash you no longer need
git stash clear              # delete ALL stashes
```

Other stash tricks worth knowing:
```bash
git stash show -p stash@{0}         # view a stash's contents as a diff, without applying it
git stash branch new-branch-name    # create a new branch AND apply the stash onto it in one step — great when you realize mid-stash that this work deserves its own branch
git stash push -p                   # interactively choose WHICH changes to stash (patch mode, same UI as `git add -p`)
```

> [!NOTE]
> A stash is stored as a special kind of commit object internally (with up to two parents — one for the working tree, one for the index) — which is exactly why it can hold arbitrarily complex changes, and why `git stash show` can display it like a diff. It's not "part of history" in the normal sense (no branch points to it), but it lives in `.git` and is even recoverable via `git fsck` if you accidentally `drop` it.

## 12.2 `git cherry-pick` — taking one specific commit, anywhere you want it

**Real-life scenario:** you fixed a critical security bug on your `feature/big-redesign` branch, but that branch is weeks away from being merged, and the fix needs to go out on `main` *today*, and also on your `release/v2.3` branch for a customer still on that version.

```bash
git switch main
git cherry-pick 7b2e4d1
```
This takes the exact changes introduced by commit `7b2e4d1` and re-applies them as a **new commit** on your current branch — same content, same message by default, but a new SHA and a new parent, exactly like a rebase replay (Chapter 7.1) but for one hand-picked commit instead of a whole range.

```bash
git cherry-pick 7b2e4d1 9a1f3c2       # cherry-pick multiple, specific commits
git cherry-pick 7b2e4d1^..9a1f3c2     # cherry-pick an entire RANGE (note the ^ to make the range inclusive of the first commit)
git cherry-pick -n 7b2e4d1            # apply the changes but don't auto-commit (lets you tweak first, or combine several picks into one commit)
git cherry-pick -x 7b2e4d1            # append "(cherry picked from commit 7b2e4d1)" to the message — good practice for traceability across branches
```

Conflicts during a cherry-pick look and resolve exactly like any other conflict (Chapter 6), and follow the same continue/abort pattern as rebase:
```bash
git add resolved-file.js
git cherry-pick --continue
# or: git cherry-pick --abort
```

## 12.3 `git bisect` — finding a bug with binary search instead of guessing

**Real-life scenario:** users report the checkout page has been broken "for a while," nobody's sure exactly when, and there have been 300 commits since the last known-good release. Reading 300 diffs by eye is a terrible plan. `bisect` finds the exact breaking commit in about **log₂(300) ≈ 9 steps**, by repeatedly checking out the midpoint of the remaining range and asking you (or a script) "good or bad?"

```bash
git bisect start
git bisect bad                    # the CURRENT commit (e.g., HEAD) is known broken
git bisect good v2.4.0            # this old tag/commit is known to have worked fine
```
Git checks out the commit exactly halfway between them (detached HEAD — Chapter 4.7 — this is a perfectly normal use of it) and waits:
```
Bisecting: 149 revisions left to test after this (roughly 8 steps)
```
You test it (run the app, run a specific test, whatever "broken" means here), then tell Git the verdict:
```bash
git bisect good     # if this midpoint commit is fine
# — or —
git bisect bad       # if this midpoint commit already shows the bug
```
Repeat — each answer halves the remaining search space — until Git announces:
```
7b2e4d1 is the first bad commit
```
Then clean up:
```bash
git bisect reset     # return to whatever branch/commit you were on before starting
```

### Automating it completely with `bisect run`

If "broken" can be checked by a script (a specific test passing/failing, a build succeeding, a curl request returning the right status), skip the manual back-and-forth entirely:
```bash
git bisect start HEAD v2.4.0
git bisect run npm test -- --grep "checkout flow"
```
Git runs your command at every candidate commit automatically, interprets a non-zero exit code as "bad" and zero as "good," and lands directly on the culprit commit with zero manual steps. This is precisely why Chapter 7's advice to keep every individual commit independently buildable/testable (via `rebase -i --exec`) pays off — `bisect run` is only fully automatable if every commit in the range can actually be built and tested on its own.

## 12.4 Quick reference

```bash
git stash push -m "message"       # save WIP work, clean the working directory
git stash pop                     # restore the most recent stash and remove it
git stash list                    # see all stashes
git cherry-pick <sha>             # copy one commit's changes onto your current branch
git cherry-pick -x <sha>          # ...and note where it came from in the message
git bisect start / good / bad     # manually binary-search for a breaking commit
git bisect run <command>          # fully automate that search
```

Next: [Chapter 13 — Tags, Releases & Versioning](13-tags-releases-versioning.md), where we mark specific points in history as meaningful milestones.
