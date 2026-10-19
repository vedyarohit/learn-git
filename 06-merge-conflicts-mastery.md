# Chapter 6 — Merge Conflicts Mastery

This is the chapter that terrifies beginners and, honestly, still rattles plenty of "20-year experts" who never sat down and learned the mechanism. By the end of this chapter, a conflict will be a five-minute, mechanical, boring task instead of a moment of panic. That transformation — panic to boredom — is the entire goal.

## 6.1 Why conflicts happen at all

Recall from Chapter 5: a merge does a **three-way comparison** (common ancestor vs. your side vs. their side) and auto-combines non-overlapping changes. A conflict happens only when Git genuinely **cannot decide which version is correct**, because both sides changed the *same region* of a file in *different* ways since the common ancestor. Git will never silently guess and possibly destroy someone's work — it stops and asks you, a human, to decide. That's a safety feature, not a failure.

Conflicts are not a sign you did something wrong. They're a sign that two people had different ideas about the same few lines of code at the same time — which happens constantly on any real team, and is completely normal.

## 6.2 Anatomy of a conflict marker, dissected line by line

When Git hits a conflict, it doesn't fail the merge — it pauses mid-merge and writes **both versions directly into the file**, wrapped in markers, so you (or your editor) can see and resolve it:

```
function greet(name) {
<<<<<<< HEAD
  return `Hello, ${name}! Welcome back.`;
=======
  return `Hi there, ${name.toUpperCase()}!`;
>>>>>>> feature/friendly-greeting
}
```

| Marker | Meaning |
|---|---|
| `<<<<<<< HEAD` | Everything below this, down to the `=======`, is **your current branch's** version (the one you ran `git merge` *from* — i.e., what you had checked out) |
| `=======` | The dividing line between the two versions |
| `>>>>>>> feature/friendly-greeting` | Everything above this, up from `=======`, is **the other branch's** version (the one you're merging *in*) — its name is printed right on the marker |

**Nothing is "correct" or "wrong" here** — these are just labeled excerpts of two different pieces of work. Your job is to edit this block into what the code *should* actually say — which might be one side, the other side, a hand-merged combination of both, or something else entirely — and then delete all three marker lines. There is no command that does this final judgment call for you; that's the one truly human step in the entire process.

**Resolved version, for example:**
```javascript
function greet(name) {
  return `Hello, ${name.toUpperCase()}! Welcome back.`;
}
```

## 6.3 The complete, repeatable resolution workflow

This exact sequence works for every conflict you'll ever hit, regardless of size or cause:

```bash
git merge feature/friendly-greeting
# CONFLICT (content): Merge conflict in greet.js
# Automatic merge failed; fix conflicts and then commit the result.

git status
# both modified:   greet.js       <-- Git tells you EXACTLY which files need attention
```

1. **Open every file `git status` lists as "both modified" (or similar).**
2. **Find each `<<<<<<<` block** (a file can have several, for different regions of the same file).
3. **Edit each block** into what the file should actually contain; delete the `<<<<<<<`, `=======`, `>>>>>>>` lines completely — Git does not remove them for you.
4. **Stage the resolved file:** `git add greet.js` — this is how you tell Git "I've resolved this one, it's ready."
5. **Repeat for every conflicted file**, then check `git status` again — it moves fully-resolved files into "Changes to be committed."
6. **Finish the merge:** `git commit` (no `-m` needed — Git pre-fills a sensible merge commit message; just save and close, or add context about *how* you resolved it if it was non-obvious).

```bash
git add greet.js
git status    # confirm: "All conflicts fixed but you are still merging."
git commit    # completes the merge commit
```

> [!TIP]
> `git diff` while mid-conflict shows you a special combined diff format highlighting exactly which hunks are still unresolved — genuinely useful on files with many conflict blocks. And `git diff --staged` after `git add` shows you the *final resolved* version as a clean diff against the common ancestor, letting you sanity-check your resolution before committing.

## 6.4 Every type of conflict you'll actually encounter

Not all conflicts look like Section 6.2. Here's the full taxonomy:

### Type 1 — Both sides edited the same lines (the classic case, shown above)
Covered fully in 6.2–6.3.

### Type 2 — One side deleted a file, the other side modified it
```
CONFLICT (modify/delete): config.yml deleted in feature/cleanup and modified in HEAD.
Version HEAD of config.yml left in tree.
```
Git can't know if the file should still exist. `git status` shows this file as `deleted by us` or `deleted by them`. Decide, then act:
```bash
git add config.yml       # "keep it" (with whatever edits are currently in the working tree)
# — or —
git rm config.yml         # "confirm the deletion"
```

### Type 3 — Both sides added a file with the same name, different content
Treated like Type 1, but for the whole file — Git shows the conflict markers wrapping both entire versions. Resolve, `git add`, done.

### Type 4 — Rename conflicts
One side renamed `utils.js` → `helpers.js`, the other side kept editing the original `utils.js`. Git's rename detection (heuristic, based on content similarity) tries to follow this automatically and usually succeeds silently — but when it can't confidently tell, it reports a rename conflict and shows both candidate paths for you to reconcile manually.

### Type 5 — Binary file conflicts
Git cannot show inline markers inside a binary file (an image, a compiled asset, a `.docx`) — there's no meaningful line-by-line diff. It reports the conflict and lets you pick a whole side:
```bash
git checkout --ours  -- logo.png     # keep YOUR version entirely
git checkout --theirs -- logo.png    # keep THEIR version entirely
git add logo.png
```

### Type 6 — Whitespace-only conflicts
Often just noise from different editors/line-ending settings. Consider re-running the merge with a whitespace-tolerant option *before* resolving by hand:
```bash
git merge -X ignore-space-change feature
```

## 6.5 `--ours` and `--theirs`: the fast escape hatch for whole files

When you already know, with certainty, that one entire side should simply win for a specific file (not a hand-merge, just "take this whole file as-is"):

```bash
git checkout --ours  -- path/to/file    # take YOUR side, ignore theirs, for this file only
git checkout --theirs -- path/to/file   # take THEIR side, ignore yours, for this file only
git add path/to/file
```

> [!CAUTION]
> **"Ours" and "theirs" flip meaning depending on the operation** — this catches even experienced people off guard. During a plain `git merge`, "ours" = your current branch, "theirs" = the branch being merged in (exactly as described in 6.2). During a `git rebase`, it's **reversed**: because rebase works by replaying *your* commits on top of the *other* branch, "ours" temporarily means the branch you're rebasing onto, and "theirs" means your own commit being replayed. When in doubt, don't guess — open the file and read the actual content inside the markers.

## 6.6 Configuring better tools for the job

### A real merge tool instead of raw markers

Reading three-way conflicts as colored side-by-side panes is dramatically easier than raw text markers:

```bash
git config --global merge.tool vscode
git config --global mergetool.vscode.cmd 'code --wait $MERGED'
git mergetool     # launches your configured tool on every conflicted file, one by one
```

VS Code's built-in 3-way merge editor (opens automatically on a conflicted file if VS Code is your editor) shows "Current Change," "Incoming Change," and the result pane side by side with one-click "Accept Current / Accept Incoming / Accept Both" buttons — genuinely the easiest way to resolve conflicts as a beginner.

### See the common ancestor too — `diff3` style

By default, conflict markers show only *your* version and *their* version — you have to remember or guess what the original looked like. Turning on `diff3` (or the newer, smarter `zdiff3`) style adds a **third section showing the original common-ancestor text**, which makes it far easier to tell what each side actually *changed* rather than just what they currently *contain*:

```bash
git config --global merge.conflictstyle zdiff3
```
```
<<<<<<< HEAD
  return `Hello, ${name}! Welcome back.`;
||||||| e4a2b91 (merge base)
  return `Hi ${name}`;
=======
  return `Hi there, ${name.toUpperCase()}!`;
>>>>>>> feature/friendly-greeting
```
Now you can see plainly: the base said `Hi ${name}`, one side added a friendlier greeting, the other side added uppercasing. That's a much easier merge to reason about than seeing only the two end results. **This single config line is one of the most underused hero-level tricks in this entire course** — almost nobody turns it on by default, and it should be on by default for everyone.

## 6.7 Conflicts during rebase and cherry-pick look the same, but the commands differ

The markers themselves are identical, but which command you use to move forward differs by operation — mixing these up is a very common mistake:

| Operation | To continue after resolving | To bail out completely |
|---|---|---|
| `git merge` | `git commit` | `git merge --abort` |
| `git rebase` | `git rebase --continue` | `git rebase --abort` |
| `git cherry-pick` | `git cherry-pick --continue` | `git cherry-pick --abort` |

Also universally available: `--skip` (on rebase/cherry-pick only) to discard just the current commit entirely and move to the next one — useful when a specific commit's change is no longer relevant.

## 6.8 `rerere` — never resolve the same conflict twice

If you've enabled `rerere.enabled true` (Chapter 2), Git silently **rec**ords **re**solutions and **re**plays them automatically the next time it sees the *identical* conflicting hunk — extremely common during a long-running branch that you repeatedly rebase onto a fast-moving `main`. The first time you resolve a recurring conflict, Git remembers your exact resolution; every subsequent identical conflict is fixed instantly, with no prompt at all. Check `git rerere status` / `git rerere diff` to see what it has recorded.

## 6.9 A full real-world walkthrough

Two developers, Priya and Sam, both branched from the same commit and both edited `pricing.js`.

```bash
$ git switch main
$ git pull
$ git merge sam/discount-logic
Auto-merging pricing.js
CONFLICT (content): Merge conflict in pricing.js
Automatic merge failed; fix conflicts and then commit the result.

$ git status
On branch main
You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Unmerged paths:
  (use "git add <file>..." to mark resolution)
        both modified:   pricing.js

$ code pricing.js
```
Inside the editor, Priya sees:
```javascript
function applyDiscount(total, user) {
<<<<<<< HEAD
  if (user.isPremium) return total * 0.9;
  return total;
=======
  if (user.couponCode) return total * 0.85;
  return total;
>>>>>>> sam/discount-logic
}
```
Both changes are actually wanted — a premium discount AND a coupon discount, not an either/or. Priya hand-merges them:
```javascript
function applyDiscount(total, user) {
  if (user.isPremium) return total * 0.9;
  if (user.couponCode) return total * 0.85;
  return total;
}
```
```bash
$ git add pricing.js
$ git status
# All conflicts fixed but you are still merging.
$ git commit
# opens editor with a pre-filled "Merge branch 'sam/discount-logic'" message — saved as-is
$ git push
```

That's it. One conflict, one file, five minutes, zero drama — exactly the outcome this chapter promised at the top.

## 6.10 Preventing conflicts in the first place

- **Pull/rebase from `main` frequently** — the longer a branch diverges, the more surface area for conflicts.
- **Keep pull requests small** — a 40-line PR rarely conflicts; a 4,000-line PR touching four unrelated modules conflicts constantly.
- **Communicate** — if you know a teammate is deep in `pricing.js`, say something before you also start refactoring it.
- **Agree on formatting tooling (Prettier, Black, etc.)** so conflicts aren't manufactured by whitespace/quote-style differences that have nothing to do with actual logic changes.

You now have a complete, calm process for any conflict Git can throw at you. Next: [Chapter 7 — Rebase Mastery](07-rebase-mastery.md), the other major way to combine divergent history — and the one with a Golden Rule you must not break.
