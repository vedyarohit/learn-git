# Chapter 21 — Cheat Sheet, Glossary & Mastery Checklist

## 21.1 The complete command cheat sheet

### Setup
```bash
git config --global user.name "Name"
git config --global user.email "email"
git config --global init.defaultBranch main
git init
git clone <url>
```

### Daily workflow
```bash
git status
git add <file>            git add -p            git add .
git commit -m "msg"        git commit --amend
git diff                    git diff --staged
git log --oneline --graph --all
```

### Branching
```bash
git branch                        git branch -a
git switch <branch>                git switch -c <branch>
git branch -d <branch>             git branch -D <branch>
git branch -m <old> <new>
```

### Merging & rebasing
```bash
git merge <branch>                 git merge --no-ff <branch>
git merge --squash <branch>        git merge --abort
git rebase <branch>                git rebase -i HEAD~N
git rebase --continue              git rebase --abort         git rebase --skip
git rebase --onto <new> <old> <branch>
```

### Conflicts
```bash
git status                                    # see which files conflict
git checkout --ours -- <file>                 git checkout --theirs -- <file>
git add <resolved-file>
git merge --continue    # (i.e., git commit)   git rebase --continue
git config --global merge.conflictstyle zdiff3
```

### Remotes
```bash
git remote -v                      git remote add <name> <url>
git fetch <remote>                 git pull                    git push
git push -u origin <branch>        git push --force-with-lease
```

### Undo / recovery
```bash
git restore <file>                 git restore --staged <file>
git reset --soft HEAD~1            git reset HEAD~1             git reset --hard HEAD~1
git revert <sha>
git reflog                         git switch -c saved <sha-from-reflog>
```

### Stash / cherry-pick / bisect
```bash
git stash push -m "msg"            git stash pop                git stash list
git cherry-pick <sha>              git cherry-pick -x <sha>
git bisect start                   git bisect good / bad        git bisect run <cmd>
```

### Tags & releases
```bash
git tag -a v1.0.0 -m "msg"         git push origin v1.0.0        git push origin --tags
gh release create v1.0.0
```

### Inspecting internals
```bash
git cat-file -p <sha>              git hash-object <file>
git ls-tree HEAD                   git fsck --unreachable
```

### GitHub CLI
```bash
gh repo create / clone / view
gh issue create / list
gh pr create / list / checkout <n> / view
gh run list / watch
```

---

## 21.2 Glossary

| Term | Definition |
|---|---|
| **Blob** | A Git object storing raw file content, with no filename attached |
| **Tree** | A Git object listing filenames and permissions, pointing to blobs/other trees — Git's version of a folder |
| **Commit** | A snapshot: pointer to one tree, plus parent(s), author, and message |
| **SHA / hash** | The unique fingerprint identifying every Git object, derived from its content |
| **HEAD** | A pointer to whatever you currently have checked out — usually a branch, sometimes a raw commit ("detached") |
| **Branch** | A movable named pointer to a commit |
| **Tag** | An unmoving named pointer to a commit, typically marking a release |
| **Remote** | A named URL referencing another copy of the repository (e.g., `origin`) |
| **Remote-tracking branch** | A local, read-only bookmark (e.g., `origin/main`) recording where a branch was on the remote as of your last fetch |
| **Upstream** | (1) The branch a local branch is set to push/pull with; (2) in a fork workflow, the original repo you forked from |
| **Working directory** | The actual files on disk that you edit |
| **Staging area / index** | The holding area where you assemble exactly what your next commit will contain |
| **Fast-forward merge** | A merge where the target branch simply moves forward to match, with no new commit created |
| **Three-way merge** | A merge combining two divergent branches via their common ancestor, producing a new merge commit |
| **Merge commit** | A commit with two (or more) parents, representing the join of divergent history |
| **Merge conflict** | A state where Git cannot auto-combine overlapping changes and needs a human decision |
| **Rebase** | Replaying commits onto a new base commit, producing new SHAs for each |
| **Interactive rebase** | A rebase where you manually edit the sequence of commits (squash, reorder, drop, reword, etc.) |
| **Squash** | Combining multiple commits' changes into one |
| **Cherry-pick** | Applying one specific commit's changes onto a different branch |
| **Stash** | A temporary, uncommitted snapshot of your working directory, set aside for later |
| **Bisect** | A binary-search process for finding which commit introduced a bug |
| **Reflog** | A local log of every position HEAD has held — Git's safety net for recovering "lost" work |
| **Force push** | Overwriting a remote branch's history unconditionally (`--force`) or safely, checked against your last known state (`--force-with-lease`) |
| **Fork** | A personal, GitHub-hosted copy of someone else's repository |
| **Pull Request (PR)** | A GitHub feature requesting that one branch be merged into another, with review and discussion attached |
| **CODEOWNERS** | A file mapping paths to people/teams, used to auto-request reviewers |
| **Branch protection rule** | A GitHub setting enforcing requirements (reviews, CI, no force-push) before a branch can be merged into or changed |
| **CI/CD** | Continuous Integration / Continuous Deployment — automatically testing and shipping code on every change |
| **GitHub Actions** | GitHub's built-in automation/workflow engine, defined in `.github/workflows/*.yml` |
| **Submodule** | A pinned reference to another Git repository, embedded inside your own |
| **Worktree** | A second working directory attached to the same repository, checked out to a different branch simultaneously |
| **SemVer** | Semantic Versioning — MAJOR.MINOR.PATCH, where each position signals a specific kind of change |
| **DAG** | Directed Acyclic Graph — the structural shape of Git's commit history |
| **Porcelain / Plumbing** | User-facing commands (`commit`, `status`) vs. low-level internal commands (`cat-file`, `hash-object`) |

---

## 21.3 The "Are you a hero yet?" self-assessment

Go through honestly. Anything you can't confidently do, that chapter number is your next stop.

**Foundations**
- [ ] I can explain, without notes, why Git stores snapshots instead of diffs — [Ch. 1](01-git-mental-model-and-internals.md)
- [ ] I know what a blob, tree, and commit each are, and can use `git cat-file -p` to look at one — [Ch. 1](01-git-mental-model-and-internals.md)
- [ ] My global Git config, SSH keys, and `gh` CLI are all properly set up — [Ch. 2](02-installation-and-setup.md)

**Core skills**
- [ ] I understand the difference between working directory, staging area, and repository, and use `git add -p` when it's useful — [Ch. 3](03-core-workflow-basics.md)
- [ ] I can explain what a branch physically is, and I'm not afraid of detached HEAD — [Ch. 4](04-branching-deep-dive.md)
- [ ] I can predict, before running it, whether a merge will fast-forward or create a merge commit — [Ch. 5](05-merging-explained.md)
- [ ] I can resolve any merge conflict calmly, including modify/delete and binary conflicts — [Ch. 6](06-merge-conflicts-mastery.md)
- [ ] I've used interactive rebase to squash/reorder/drop commits, and I know the Golden Rule — [Ch. 7](07-rebase-mastery.md)
- [ ] I know exactly what `fetch` does differently from `pull` — [Ch. 8](08-remotes-and-collaboration.md)

**GitHub & collaboration**
- [ ] I've run a fork → clone → PR → sync-with-upstream workflow start to finish — [Ch. 8](08-remotes-and-collaboration.md), [Ch. 9](09-github-platform-essentials.md)
- [ ] I know the difference between the three PR merge methods and when to use each — [Ch. 10](10-pull-requests-and-code-review-workflow.md)

**Undo & recovery**
- [ ] I know precisely when to use `restore` vs. `reset` vs. `revert` — [Ch. 11](11-undoing-changes-time-travel.md)
- [ ] I've personally used `git reflog` to recover something — [Ch. 11](11-undoing-changes-time-travel.md)
- [ ] I've used `stash`, `cherry-pick`, and `bisect` for real, not just read about them — [Ch. 12](12-stash-cherrypick-bisect.md)

**Advanced / master level**
- [ ] I understand annotated vs. lightweight tags and SemVer — [Ch. 13](13-tags-releases-versioning.md)
- [ ] I know what hooks, submodules, worktrees, and signed commits are for — [Ch. 14](14-advanced-git-toolkit.md)
- [ ] I know why `--force-with-lease` exists and use it by default over `--force` — [Ch. 15](15-rewriting-history-safely.md)
- [ ] I can explain Git Flow vs. GitHub Flow vs. Trunk-Based Development and argue for one — [Ch. 16](16-branching-strategies-and-workflows.md)
- [ ] I can read and write a basic GitHub Actions workflow file — [Ch. 17](17-github-actions-and-automation.md)

**Real world**
- [ ] I could handle at least 15 of the 20 disaster scenarios without looking anything up — [Ch. 18](18-disaster-recovery-playbook.md)
- [ ] I write commit messages a stranger could understand in a year — [Ch. 20](20-best-practices-and-etiquette.md)

If you checked every box: genuinely, you now understand Git and GitHub more thoroughly than the majority of people who use it professionally every day — not because the material was secret, but because almost nobody sits down and learns the *model*, only the commands they've personally needed so far.

## 21.4 Where to go from here

- **Practice deliberately.** Create a playground repo and intentionally cause a conflict, then a bad rebase, then a "lost" commit — and recover from each without looking at this course, only reaching back to it when truly stuck.
- **Read real history.** Run `git log --graph --oneline --all` on a large, real open-source project (clone one — e.g., a project you already use) and try to narrate the story it tells.
- **Contribute to something.** Fork a small open-source project, fix a typo or a "good first issue," and run the full fork → branch → PR → review cycle for real, with real strangers reviewing your work.
- **Revisit this course as reference**, not just once. Chapters 11, 15, and 18 in particular are worth re-reading a month from now, after you've hit a couple of real situations of your own — they'll land completely differently the second time.

That's the whole course. Go break things in a playground repo — you now know how to put every single piece back together.
