# Chapter 13 — Tags, Releases & Versioning

## 13.1 What a tag is

Like a branch, a tag is a named pointer to a specific commit — but unlike a branch, **a tag never moves.** Once `v1.0.0` points at commit `a1b2c3d`, it points there forever (unless someone deliberately force-recreates it, which is unusual and generally discouraged). Tags exist to mark a specific, permanent, meaningful point in history — almost always a release.

## 13.2 Lightweight vs. annotated tags

```bash
git tag v1.0.0-lw                 # lightweight — literally just a name pointing at a commit, nothing more
git tag -a v1.0.0 -m "First stable release"   # annotated — a real, full Git OBJECT with its own metadata
```

| | Lightweight | Annotated |
|---|---|---|
| What it is | A bare pointer (like a branch, but that never moves) | A full Git object: tagger name/email, date, message, and a pointer to the commit |
| Can be GPG/SSH-signed | No | Yes (`git tag -s`) |
| Shows up in `git describe` | No | Yes |
| Recommended for | Quick, personal, throwaway markers | **Anything you'll ever publish or rely on — i.e., basically all real releases** |

**Default to annotated tags for anything real.** The extra metadata (who tagged it, when, why) is exactly the kind of context you'll want later and can never reconstruct after the fact from a lightweight tag.

## 13.3 Working with tags

```bash
git tag                          # list all tags
git tag -l "v1.1.*"              # list tags matching a pattern
git show v1.0.0                  # see the tag's metadata AND the commit/diff it points to
git tag -a v1.0.1 a1b2c3d -m "Patch release"   # tag a SPECIFIC (e.g., older) commit, not just HEAD
git tag -d v1.0.0-lw              # delete a tag locally
```

> [!WARNING]
> **Tags do not push automatically** — `git push` alone never sends your tags to the remote. This surprises almost everyone the first time.
> ```bash
> git push origin v1.0.0            # push one specific tag
> git push origin --tags            # push every tag you have locally that the remote doesn't
> git push --delete origin v1.0.0   # delete a tag from the REMOTE (deleting locally, per above, doesn't touch the remote either)
> ```

## 13.4 Checking out a tag

```bash
git checkout v1.0.0
```
This puts you in **detached HEAD** (Chapter 4.7) — entirely expected and safe, since a tag is a fixed point, not something you're meant to commit new work directly onto. If you do need to build on top of a historical tag (a rare hotfix-on-an-old-release scenario):
```bash
git switch -c hotfix/v1.0.1 v1.0.0    # create a proper branch FROM the tag, so you have somewhere safe to commit
```

## 13.5 Semantic Versioning (SemVer) — the near-universal convention

Version numbers in the format **MAJOR.MINOR.PATCH** (e.g., `2.4.1`), where each position means something specific and machine-interpretable — this is what lets tools automatically know whether upgrading a dependency is likely to be safe:

| Position | Bump when... | Example |
|---|---|---|
| **MAJOR** | You make incompatible/breaking API changes | `1.9.0` → `2.0.0` |
| **MINOR** | You add functionality in a backward-compatible way | `2.0.0` → `2.1.0` |
| **PATCH** | You make backward-compatible bug fixes only | `2.1.0` → `2.1.1` |

Pre-release and build metadata extend this: `2.1.0-beta.1`, `2.1.0-rc.2`, `2.1.0+build.20240115`. The full formal spec lives at [semver.org](https://semver.org) — worth reading once, since so much of the package-manager ecosystem (npm's `^`/`~` version ranges, for instance) is built directly on these rules.

## 13.6 GitHub Releases — turning a tag into something users actually see

A **Release** on GitHub wraps a tag with a human-facing page: formatted release notes, a title, and optionally attached binary **assets** (a compiled `.exe`, a `.zip`, a `.whl` — anything you want people to download directly, distinct from just browsing source code).

```bash
gh release create v1.0.0 --title "v1.0.0 — First stable release" --notes "See CHANGELOG.md for details"
gh release create v1.0.0 ./dist/app.zip --title "v1.0.0"   # attach a binary asset
gh release list
```
Or via the web UI: **repo page → Releases → Draft a new release**, which can create the underlying tag for you at the same time if it doesn't already exist.

### Auto-generated release notes

GitHub can generate a changelog automatically from merged PR titles since the last release (**"Generate release notes"** button) — this works far better, with much less manual effort, when your team follows the commit-message conventions covered in [Chapter 20](20-best-practices-and-etiquette.md#201-conventional-commits).

## 13.7 Quick reference

```bash
git tag -a v1.2.0 -m "message"     # create an annotated tag (default choice)
git push origin v1.2.0             # tags need an explicit push!
git push origin --tags             # push all local tags at once
git checkout v1.2.0                # inspect a tagged commit (detached HEAD, expected)
gh release create v1.2.0           # turn a tag into a real, user-facing GitHub Release
```

Next: [Chapter 14 — Advanced Git Toolkit](14-advanced-git-toolkit.md) — hooks, submodules, worktrees, and signed commits: the tools that most separate casual users from people who've genuinely gone deep.
