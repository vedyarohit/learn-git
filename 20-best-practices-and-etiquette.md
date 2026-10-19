# Chapter 20 — Best Practices & Etiquette

Commands get you unstuck; habits are what make you good to work with. This chapter is the difference between "technically knows Git" and "the kind of collaborator every team wants."

## 20.1 Conventional Commits

A widely-adopted convention for structuring commit messages so they're both human-readable and machine-parseable (able to auto-generate changelogs, auto-determine the next semantic version — Chapter 13.5):

```
<type>(<optional scope>): <short summary>

<optional longer body>

<optional footer>
```

| Type | Used for |
|---|---|
| `feat` | A new feature |
| `fix` | A bug fix |
| `docs` | Documentation only |
| `refactor` | Code change that neither fixes a bug nor adds a feature |
| `test` | Adding or correcting tests |
| `chore` | Tooling, dependencies, build config |
| `perf` | A performance improvement |

**Examples:**
```
feat(auth): add password reset via email

fix(cart): prevent negative quantity when rapidly clicking decrement

refactor(api): extract shared validation logic into middleware
```
Appending `!` after the type/scope (`feat(api)!: change response shape`) or adding a `BREAKING CHANGE:` footer flags a breaking change — tooling built on this convention (like `semantic-release`) uses exactly this signal to know whether the next version bump should be MAJOR, MINOR, or PATCH ([Chapter 13.5](13-tags-releases-versioning.md#135-semantic-versioning-semver--the-near-universal-convention)), fully automatically.

## 20.2 The 50/72 rule and imperative mood

Even without adopting the full Conventional Commits prefix system, two habits from Git's own documented conventions matter:

- **Subject line ≤ ~50 characters**, body wrapped at ~72 characters — this isn't arbitrary; it's exactly what fits cleanly in `git log --oneline`, GitHub's PR list view, and a terminal without wrapping awkwardly.
- **Write the subject in the imperative mood** — "Add validation," not "Added validation" or "Adds validation." The test: it should complete the sentence *"If applied, this commit will \_\_\_."* This convention exists because Git's own auto-generated messages (merge commits, reverts) are written the same way — consistency makes `git log` read like a coherent set of instructions rather than a mixed-tense diary.

**Bad:** `fixed stuff`
**Good:** `Fix race condition in concurrent cart updates`

## 20.3 Atomic commits

One commit should represent **one logical change** — not "everything I did today." This isn't pedantry; it has direct practical payoffs covered earlier in this course: `git revert` only cleanly undoes a self-contained change ([11.4](11-undoing-changes-time-travel.md#114-git-revert--the-safe-way-to-undo-sharedpushed-commits)), `git bisect` only pinpoints a bug precisely if each commit is one coherent, testable change ([12.3](12-stash-cherrypick-bisect.md#123-git-bisect--finding-a-bug-with-binary-search-instead-of-guessing)), and `git cherry-pick` only ports cleanly if a fix isn't tangled up with unrelated formatting changes ([12.2](12-stash-cherrypick-bisect.md#122-git-cherry-pick--taking-one-specific-commit-anywhere-you-want-it)). Use `git add -p` (Chapter 3.3) to split up a working directory that accumulated more than one concern before you commit.

## 20.4 Branch naming, one more time, as policy

Consistent, greppable branch names make `git branch` and GitHub's branch list actually useful at a glance:
```
feature/short-description
fix/short-description
hotfix/short-description
chore/short-description
```
Avoid names tied to a person (`johns-branch`) or vague enough to be meaningless a week later (`updates`, `stuff`, `test2`).

## 20.5 `.gitignore` hygiene

Never commit, ever, regardless of how "temporary" it feels in the moment:
- Dependency folders (`node_modules/`, `vendor/`, `venv/`) — regenerable from a lockfile, and enormous
- Build output (`dist/`, `build/`, `*.class`, `__pycache__/`)
- **Anything with real credentials** (`.env`, `*.pem`, `credentials.json`) — see [Chapter 18, Scenario 9](18-disaster-recovery-playbook.md#9-i-committed-and-maybe-pushed-a-secret--api-key-password-token) if this already happened
- Editor/OS artifacts (`.DS_Store`, `.idea/`, `*.swp`)

Set these up in `.gitignore` **before your first commit** on a new project — much easier than untangling already-tracked junk later ([Chapter 3.7](03-core-workflow-basics.md#37-gitignore--telling-git-what-to-never-track)).

## 20.6 Code review etiquette — as the author

- Keep PRs small enough that a reviewer can hold the whole thing in their head — [Chapter 10.8](10-pull-requests-and-code-review-workflow.md#108-pr-etiquette-that-actually-matters) covered why.
- Write a description that answers *what* changed, *why*, and *how you verified it* — not just a restatement of the diff.
- Respond to every single comment thread, even briefly — silence reads as "I ignored this."
- Don't take "Request changes" as a judgment of you personally; it's the process working as intended.

## 20.7 Code review etiquette — as the reviewer

- Distinguish, explicitly in your wording, between *"this is broken"*, *"this is a strong suggestion,"* and *"this is just a personal preference, feel free to ignore."* Unlabeled nitpicks read as blocking issues and cause unnecessary back-and-forth.
- Review promptly — a PR sitting unreviewed for days blocks someone else's work and encourages larger, harder-to-review batches next time as people wait longer between requests.
- Ask questions instead of asserting when you're genuinely unsure ("Is there a reason this isn't memoized?" vs. "This should be memoized.") — you might be missing context, and phrasing it as a question invites the author to explain rather than defend.

## 20.8 When to squash vs. preserve history

Revisit [Chapter 5.5](05-merging-explained.md#55-squash-merge--a-third-option-technically-not-a-merge-at-all) and [10.5](10-pull-requests-and-code-review-workflow.md#105-the-three-merge-methods--know-these-cold): squash when your commits-in-progress are messy and only the final PR-level result matters; preserve individual commits when each one is independently meaningful and you'd want to `git blame`, revert, or bisect at that granularity later. Most teams standardize on one default (squash is more common) specifically so this isn't a per-PR debate.

## 20.9 Security basics, restated as habits

- **Enable Two-Factor Authentication on your GitHub account.** GitHub has required 2FA for all contributors to any repository since 2023 — but confirm it's actually configured with a real second factor, not left half-set-up.
- **Never commit secrets** — use `.env` files (git-ignored) locally, and a real secrets manager (GitHub Actions Secrets, AWS Secrets Manager, Vault, etc.) in deployed environments.
- **Consider signed commits** ([Chapter 14.4](14-advanced-git-toolkit.md#144-signing-commits)) for any repository where verifying authorship genuinely matters — security-sensitive open source, regulated industries.
- **Review branch protection settings** ([Chapter 10.6](10-pull-requests-and-code-review-workflow.md#106-branch-protection-rules)) on any repository you administer — the default, unprotected state allows direct pushes and force-pushes to `main` from anyone with write access, which is rarely what you actually want.

You now know not just *how* to run every command in this course, but *when*, and *why a professional team would care*. Last stop: [Chapter 21 — Cheat Sheet, Glossary & Mastery Checklist](21-cheat-sheet-glossary-and-mastery-checklist.md).
