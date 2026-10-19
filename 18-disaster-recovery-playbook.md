# Chapter 18 — Disaster Recovery Playbook

Every scenario below follows the same format: what happened, how panicked you should actually be, and the exact fix. Bookmark this chapter — it's meant to be searched during an actual emergency, not read start to finish. The recurring theme, per Chapter 11: **almost nothing here is truly unrecoverable**, because Git rarely deletes anything immediately.

> [!TIP]
> General first move for almost any "I think I lost work" moment: **stop, don't run anything else yet, and run `git reflog`.** It's the answer to more of these scenarios than any other single command.

---

### 1. I committed to the wrong branch
**Panic level: none.** Very common, trivially fixable.
```bash
git branch correct-branch          # create a new branch AT your current (wrong-branch) position
git reset --hard HEAD~1            # remove the commit from the wrong branch (safe — it lives on correct-branch now too)
git switch correct-branch          # continue your work in the right place
```
*Why it works:* `git branch` just creates a pointer to where you currently are — the commit now has two names for a moment, then you remove one of them.

---

### 2. I need to undo my last commit — haven't pushed yet
**Panic level: none.**
```bash
git reset --soft HEAD~1     # keeps changes staged, ready to re-commit differently
# or
git reset --hard HEAD~1     # discards the changes entirely, if you truly don't want them
```
Full detail in [Chapter 11.3](11-undoing-changes-time-travel.md#113-git-reset--rewinding-your-branch-pointer-with-three-levels-of-force).

---

### 3. I need to undo a commit that's already pushed and others may have pulled it
**Panic level: low.** Do **not** `reset` + force-push here.
```bash
git revert <bad-commit-sha>
git push
```
`revert` adds a new, opposite commit rather than rewriting history — completely safe on shared branches ([Chapter 11.4](11-undoing-changes-time-travel.md#114-git-revert--the-safe-way-to-undo-sharedpushed-commits)).

---

### 4. I accidentally committed a file that should've been in `.gitignore` (but it's not a secret — e.g., a build folder)
**Panic level: low.**
```bash
git rm -r --cached build/
echo "build/" >> .gitignore
git add .gitignore
git commit -m "Stop tracking build output"
git push
```
`--cached` removes it from Git's tracking **without deleting it from your disk**. If it's already in old history and you need it gone from *every* past commit too (bloat, not secrecy), see [Chapter 15.3](15-rewriting-history-safely.md#153-purging-a-file-from-your-entire-history).

---

### 5. I accidentally deleted a branch that had unmerged commits
**Panic level: low, if recent.**
```bash
git reflog | grep "checkout: moving from deleted-branch"
# or just scan: git reflog
git switch -c deleted-branch <sha-from-reflog>
```
The branch *name* is gone, but the commits it pointed to are still sitting in the object database until garbage collection — reflog remembers the last SHA the branch pointed to. Full mechanism: [Chapter 11.6](11-undoing-changes-time-travel.md#116-git-reflog--your-ultimate-safety-net).

---

### 6. I force-pushed and now my teammate's local branch conflicts with everything
**Panic level: medium.** This is a coordination problem, not just a Git problem.
On the **teammate's** machine, first make sure nothing of theirs is at risk:
```bash
git branch backup-just-in-case      # snapshot their current state before touching anything
git fetch origin
git reset --hard origin/feature-x   # now matches the force-pushed remote exactly
```
If they had genuinely new commits not reflected in the force-push, cherry-pick them back from `backup-just-in-case` afterward. **Prevention beats cure here:** this is exactly the scenario `--force-with-lease` (Chapter 15.4) exists to prevent in the first place — the person who force-pushed should have been using it.

---

### 7. A merge conflict is an absolute mess and I want to bail entirely
**Panic level: none.**
```bash
git merge --abort       # if you're mid-merge
git rebase --abort      # if you're mid-rebase
```
Both fully restore you to exactly where you were before you started — as if you'd never attempted it. Full conflict process: [Chapter 6](06-merge-conflicts-mastery.md).

---

### 8. I'm in detached HEAD and made commits I want to keep
**Panic level: none, but act before switching branches.**
```bash
git switch -c rescued-work
```
One command, done — full explanation in [Chapter 4.7](04-branching-deep-dive.md#47-detached-head--demystified-completely).

---

### 9. I committed (and maybe pushed) a secret — API key, password, token
**Panic level: high — but the fix is not "clean the Git history," it's this, in this exact order:**
1. **Rotate/revoke the credential at its source immediately** — this is the actual fix. A leaked-then-rotated key is safe; a "removed from history" but never-rotated key is still compromised.
2. Clean the history if you still want it gone (`git filter-repo` or BFG — [Chapter 15.5](15-rewriting-history-safely.md#155-what-to-actually-do-if-you-commit-a-secret-api-key-password-token)).
3. Force-push (`--force-with-lease`) and have every collaborator re-clone.

---

### 10. I ran `git add .` and staged files I didn't mean to
**Panic level: none.**
```bash
git restore --staged unwanted-file.js       # unstage one file
git restore --staged .                       # unstage everything, start over
```

---

### 11. I wrote a terrible commit message and already pushed
**Panic level: none, if you're the only one who's based work on it yet.**
```bash
git commit --amend -m "Actually clear message"
git push --force-with-lease
```
If someone may have already pulled it, prefer to just leave it — amending a pushed, possibly-pulled commit reopens the Golden Rule problem (Chapter 7.3) for a purely cosmetic gain that usually isn't worth it.

---

### 12. I need to combine my last 5 messy commits into 1 clean commit before opening a PR
**Panic level: none.**
```bash
git reset --soft HEAD~5
git commit -m "One clean, well-described commit"
```
Or, for more control over the final message/order, `git rebase -i HEAD~5` and mark four of the five as `squash`/`fixup` ([Chapter 7.5](07-rebase-mastery.md#75-interactive-rebase--the-real-power-tool)).

---

### 13. `fatal: refusing to merge unrelated histories`
**Panic level: none.** This happens when two repositories (or a repo re-initialized with `git init` and one already on GitHub) genuinely have no shared commit ancestor — common when a project was started both locally and on GitHub separately, with an auto-generated README on one side only.
```bash
git pull origin main --allow-unrelated-histories
```
Resolve any resulting conflicts exactly as in Chapter 6 (very likely just the README/`.gitignore`), then commit normally.

---

### 14. I accidentally committed directly to `main` when I meant to branch first
**Panic level: none, if not yet pushed.**
```bash
git branch feature/my-work          # save a pointer to this work under a proper name
git reset --hard origin/main        # rewind main to match the remote (removing your local-only commit from main)
git switch feature/my-work          # continue there instead
```

---

### 15. I ran `git checkout .` (or `git restore .`) and lost uncommitted edits
**Panic level: high, and this is one of the few genuinely hard cases.** Uncommitted changes were **never a Git object** — they exist only in your working directory and, unlike commits, `reflog` cannot resurrect what was never committed.
- If the file was open in an editor (VS Code, JetBrains, etc.), check the editor's **own local history/undo** — many editors (notably VS Code and JetBrains IDEs) keep independent local history snapshots of edited files, entirely separate from Git, specifically for this situation.
- If you'd run `git add` on it at any point before discarding (even once, even a while ago), the staged blob may still exist loose in the object database — try `git fsck --no-reflogs --unreachable` and grep through candidate blobs with `git cat-file -p <sha>`.
- Otherwise: this is the sharp edge of Git's design (Chapter 11.2) — it's why teams recommend committing early and often, even with imperfect messages, since a committed-but-messy state is fully recoverable and an uncommitted one isn't.

---

### 16. A huge binary file got committed and now the repo is slow to clone
**Panic level: medium, mostly about coordination effort, not risk.**
```bash
git filter-repo --path path/to/huge-file.zip --invert-paths
```
Then force-push and have everyone re-clone ([Chapter 15.3](15-rewriting-history-safely.md#153-purging-a-file-from-your-entire-history)). Going forward, track large files properly with **Git LFS** ([Chapter 14.5](14-advanced-git-toolkit.md#145-git-lfs-large-file-storage)) instead of committing them raw.

---

### 17. I need to change the author name/email on old commits
**Panic level: low, effort: medium.**
```bash
git filter-repo --mailmap ../mailmap.txt
```
where `mailmap.txt` contains lines like:
```
Correct Name <correct@email.com> <old-wrong@email.com>
```
Rewrites every matching commit's author identity across all of history. Force-push + re-clone required afterward, same as any `filter-repo` operation.

---

### 18. `git status` says my branch has "diverged" and I don't understand why
**Panic level: none — this is informational, not an error.**
```
Your branch and 'origin/main' have diverged,
and have 2 and 3 different commits each, respectively.
```
This just means: you made 2 local commits, and the remote gained 3 different commits, since you last synced — genuinely parallel, independent work on both sides. Fix exactly like a normal sync:
```bash
git pull      # merge or rebase, per your config (Chapter 8.2) — resolve any conflicts as usual
```

---

### 19. I dropped a stash and now I need it back
**Panic level: medium, time-sensitive.**
```bash
git fsck --unreachable | grep commit
git stash apply <sha-that-looks-right>
```
A dropped stash isn't in `reflog` by default (stashes have their own separate reflog: try `git fsck` output or check `.git/logs/refs/stash` directly if it still exists) — but the underlying commit object is still present until garbage collection. `git show <sha>` on each candidate to identify the right one before applying it.

---

### 20. Two people force-pushed to the same branch around the same time and stepped on each other
**Panic level: medium-high.** Same core fix as Scenario 7, but confirm with **both** people what work exists before resetting anyone's local branch:
```bash
git branch backup-mine <current-sha>     # each person snapshots their own current state FIRST
git fetch origin
git log origin/feature-x --oneline -10   # see what actually ended up on the remote
```
Compare the backup branches against what's now on the remote, cherry-pick back anything genuinely missing, then have everyone reset their local branch to match the remote going forward. This is fundamentally a "talk to your teammate" problem wearing a Git costume — the commands are secondary to agreeing on which work is the source of truth.

---

## Quick-reference decision table

| Symptom | Go to |
|---|---|
| "I think I lost commits" | `git reflog` first, always — [Chapter 11.6](11-undoing-changes-time-travel.md#116-git-reflog--your-ultimate-safety-net) |
| Merge/rebase conflict markers | [Chapter 6](06-merge-conflicts-mastery.md) |
| Need to undo something already pushed | `git revert`, not `git reset` — [Chapter 11.4](11-undoing-changes-time-travel.md#114-git-revert--the-safe-way-to-undo-sharedpushed-commits) |
| Need to force-push | `--force-with-lease`, never bare `--force` — [Chapter 15.4](15-rewriting-history-safely.md#154-force-push-safety---force-vs---force-with-lease-vs---force-if-includes) |
| Committed a secret | Rotate first, clean history second — [Chapter 15.5](15-rewriting-history-safely.md#155-what-to-actually-do-if-you-commit-a-secret-api-key-password-token) |

Next: [Chapter 19 — Real-World Scenario Simulator](19-real-world-scenario-simulator.md) — a full narrative walkthrough putting everything in this course together, end to end.
