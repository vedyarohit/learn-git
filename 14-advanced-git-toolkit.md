# Chapter 14 — Advanced Git Toolkit

This chapter is a tour of the tools that most reliably separate "comfortable daily Git user" from "the person the team asks when something truly weird happens." None of these are exotic in the sense of being rarely used industry-wide — hooks and worktrees especially are genuinely common — they're just rarely *taught*.

## 14.1 Git Hooks — running code automatically at key moments

A hook is a script Git runs automatically at a specific point in its workflow — living in `.git/hooks/` as executable files named after the exact moment they fire (`pre-commit`, `commit-msg`, `pre-push`, etc.). They're **local to a single clone** by default (not version-controlled, not shared automatically), which is exactly the limitation that shareable-hook tools (below) exist to solve.

| Hook | Fires when | Common use |
|---|---|---|
| `pre-commit` | Before a commit message editor even opens | Run linters/formatters; abort the commit if code doesn't pass |
| `commit-msg` | After the message is written, before the commit is finalized | Enforce a message format (e.g., Conventional Commits, Chapter 20) |
| `pre-push` | Before anything is sent to a remote | Run the full test suite; block the push on failure |
| `post-checkout` | After switching branches | Auto-install dependencies if a lockfile changed |
| `post-merge` | After a merge completes | Same, after pulling in teammates' changes |

**A minimal `pre-commit` hook** (`.git/hooks/pre-commit`, made executable with `chmod +x`):
```bash
#!/bin/sh
npm run lint
if [ $? -ne 0 ]; then
  echo "Linting failed — commit aborted."
  exit 1
fi
```
Any non-zero exit code from a hook script aborts the Git operation it's attached to.

### Shareable hooks, since raw `.git/hooks` isn't version-controlled

Because `.git/` itself is never pushed or cloned, raw hooks placed there don't travel with the repo — a real limitation for team-wide enforcement. Two common solutions:
- **Husky** (Node.js ecosystem) — stores hook scripts in a tracked folder (e.g., `.husky/`) and configures Git to point at it, so `npm install` sets hooks up for every contributor automatically.
- **pre-commit framework** (language-agnostic, Python-based) — a `.pre-commit-config.yaml` in the repo declares hooks from a shared registry; contributors run `pre-commit install` once after cloning.
- Native option: `git config core.hooksPath .githooks` — point Git at a tracked folder directly, no extra tooling needed at all.

> [!NOTE]
> Hooks are a **client-side** safety net — a contributor can always bypass them with `git commit --no-verify`, or simply not install them. For anything that truly must be enforced (not just encouraged), pair hooks with **server-side enforcement** — GitHub's required status checks and branch protection rules (Chapter 10.6) — since those can't be skipped by an individual contributor's local setup.

## 14.2 Submodules — a repository inside a repository

A submodule embeds one Git repository inside another **as a pinned reference to one specific commit** — useful when your project genuinely depends on another whole separate project's source (a shared internal library, a vendored dependency you need to patch).

```bash
git submodule add git@github.com:org/shared-lib.git libs/shared-lib
git commit -m "Add shared-lib as a submodule"
```
This doesn't copy the code in — it records, in a tracked `.gitmodules` file, the submodule's URL and the **exact commit SHA** it's pinned to. Anyone cloning your repo afterward needs an extra step to actually get that code:
```bash
git clone --recurse-submodules git@github.com:you/main-project.git
# — or, if you already cloned without that flag: —
git submodule update --init --recursive
```

**Updating a submodule to a newer commit of the inner repo:**
```bash
cd libs/shared-lib
git pull origin main            # move the submodule's own checkout forward
cd ../..
git add libs/shared-lib          # stage the UPDATED POINTER (not the submodule's files themselves)
git commit -m "Bump shared-lib to latest"
```

> [!CAUTION]
> The most common submodule mistake: a contributor makes changes **inside** the submodule folder itself (it's a real, independent Git repo — you genuinely can commit there), but forgets to `push` from inside the submodule before pushing the outer project. The outer repo then points at a commit that only exists on that one contributor's machine — nobody else can retrieve it. **Rule of thumb: always push inside the submodule first, then commit/push the pointer update in the outer repo.**

### Subtrees — the common alternative

`git subtree` solves a similar problem (vendoring another project's code) with a different tradeoff: it **copies the actual files directly into your repo's history**, with no separate `.gitmodules` reference and no extra clone step required for contributors. Simpler for consumers of your repo; more history bloat and a slightly trickier process for pushing changes back upstream.

| | Submodules | Subtrees |
|---|---|---|
| Contributor experience | Extra clone/init step required | Works with a totally normal `git clone` |
| Repo size impact | Minimal (just a pointer) | The dependency's full content is copied in |
| Pinning to an exact commit | Native and explicit | Possible, but less first-class |
| Best for | Actively co-developed internal libraries | Vendoring external code you rarely modify |

## 14.3 Worktrees — multiple branches checked out simultaneously

Normally, one clone = one checked-out branch at a time; switching branches means stashing or committing whatever's in progress first. **Worktrees let you check out several branches at once, into separate folders, all sharing the exact same underlying `.git` history** — no duplicate clone, no stashing required.

**Real-life scenario:** you're mid-feature on `feature/big-refactor` (half-broken, mid-edit, definitely not committable) when a critical production bug demands immediate attention. Instead of stashing:
```bash
git worktree add ../hotfix-workspace main
cd ../hotfix-workspace
# a completely separate, clean working directory, checked out to `main`,
# while ../original-project stays exactly as you left it, untouched
git switch -c hotfix/urgent-fix
# ...fix, commit, push, PR, merge...
cd ../original-project
git worktree remove ../hotfix-workspace
```
```bash
git worktree list      # see all active worktrees for this repository
```
This is dramatically faster than a second full `git clone` (no re-downloading history — it's the same object database, just a second working directory pointed at a different branch) and avoids ever disturbing your in-progress, uncommitted refactor.

## 14.4 Signing commits

Anyone can set `git config user.name "Linus Torvalds"` — Git's author field is just plain text, entirely unverified by default. **Signed commits** cryptographically prove a commit genuinely came from the holder of a specific private key, and GitHub displays a **"Verified"** badge next to any commit/tag it can confirm against a public key you've registered.

```bash
# Using an SSH key you likely already have (Chapter 2) — the modern, simplest option:
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519.pub
git config --global commit.gpgsign true    # sign EVERY commit automatically, going forward

git commit -S -m "Signed commit"           # sign just this one, if not auto-signing
```
Register the matching public key at **GitHub → Settings → SSH and GPG keys → New SSH key**, specifically under the **"Signing Key"** usage type (the same key can also be a regular authentication key, or you can use separate keys for each purpose). Security-sensitive open-source projects and regulated environments frequently require signed commits via branch protection (Chapter 10.6) — it's a genuinely strong guarantee, not just cosmetic.

## 14.5 Git LFS (Large File Storage)

Git's design (Chapter 1) assumes text-like content that diffs and compresses well — it handles huge binary files (video, large datasets, design files) very poorly, since every version of a binary blob is stored close to in full, forever, bloating clone times permanently. **Git LFS** replaces large tracked files with small text pointers inside your normal Git history, while the actual binary content lives in separate LFS storage, fetched on demand.

```bash
git lfs install                     # one-time setup per machine
git lfs track "*.psd"               # tell LFS which file patterns to intercept
git add .gitattributes              # this file records the tracking rule — must be committed
git add design.psd
git commit -m "Add design file (via LFS)"
git push
```
From here, `design.psd` behaves like any normal tracked file in your day-to-day workflow — `git lfs` transparently intercepts it under the hood.

## 14.6 Sparse checkout & partial clone — for genuinely huge repositories

On monorepos with millions of files, cloning "everything" can be impractically slow. Two complementary features address this:

```bash
git clone --filter=blob:none git@github.com:org/huge-monorepo.git
# "partial clone" — downloads all commit/tree metadata upfront, but fetches file CONTENTS
# lazily, only as you actually check them out

cd huge-monorepo
git sparse-checkout init --cone
git sparse-checkout set services/billing services/auth
# your WORKING DIRECTORY now only shows these two folders,
# even though the full repo's history is technically available
```
Most individual contributors will never need this — but knowing it exists (and why) matters the moment you join a company running a single enormous monorepo, where a naive `git clone` might otherwise take hours.

You've now covered the toolkit that experienced engineers reach for once or twice a month rather than daily — knowing it exists is most of the battle. Next: [Chapter 15 — Rewriting History Safely](15-rewriting-history-safely.md), including exactly how to handle the single most panic-inducing mistake: committing a secret.
