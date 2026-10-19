# Chapter 9 — GitHub Platform Essentials

Recall from Chapter 1: **Git is the version control engine; GitHub is a website built around it.** This chapter is entirely about the features GitHub adds on top — none of what follows is a "Git command," it's "how the GitHub product works."

## 9.1 Creating a repository

**On the website:** github.com → **New repository** → name it, choose public/private, optionally add a README, `.gitignore` template, and a license.

**From the command line, with the GitHub CLI (Chapter 2):**
```bash
gh repo create my-project --public --clone
# creates the repo on GitHub AND clones it locally in one step
```

**Turning an existing local project into a GitHub repo:**
```bash
cd existing-project
git init
git add .
git commit -m "Initial commit"
gh repo create my-project --public --source=. --push
# — or, without gh, manually: —
git remote add origin git@github.com:you/my-project.git
git branch -M main
git push -u origin main
```

## 9.2 Anatomy of a repository page

| Element | Purpose |
|---|---|
| **README.md** | Rendered automatically on the repo's front page — your project's first impression. Supports full GitHub-flavored Markdown (see 9.6). |
| **LICENSE** | Legally defines how others may use your code. GitHub offers a picker for common licenses (MIT, Apache 2.0, GPL, etc.) when creating this file. |
| **.gitignore** | Covered in Chapter 3. |
| **About panel** | Short description, website link, and **topics** (tags) — how people discover your repo via search/browsing. |
| **Insights tab** | Contributor stats, commit activity graphs, dependency graph. |

## 9.3 Issues — tracking work and bugs

Issues are GitHub's built-in ticket system — not a Git concept at all, purely a GitHub (and GitLab/similar) feature.

- **Labels** — categorize (`bug`, `enhancement`, `good first issue`, `wontfix`).
- **Assignees** — who's responsible.
- **Milestones** — group issues toward a release or deadline.
- **Linking issues to code:** referencing `#42` anywhere in a commit message or PR description auto-links to issue #42. Using specific keywords in a PR description or commit message — `Closes #42`, `Fixes #42`, `Resolves #42` — **automatically closes that issue the moment the PR is merged.** This is genuinely one of the most useful small habits you can build: it means your issue tracker maintains itself.

```bash
git commit -m "Fix null pointer on logout

Fixes #42"
```

## 9.4 Forking vs. cloning — the distinction, stated plainly

| | Clone | Fork |
|---|---|---|
| What it is | A Git operation — copies a repo to your local machine | A GitHub operation — copies a repo to **your own GitHub account** |
| Where the copy lives | Your local disk | GitHub's servers, under your username |
| Push access | Only if you're a collaborator on the original | Always — it's entirely your own copy |

You typically **fork** a repo you don't have write access to, then **clone your fork** locally to actually work on it (full workflow in [Chapter 8, Section 8.5](08-remotes-and-collaboration.md#85-the-fork-workflow)).

## 9.5 The GitHub CLI (`gh`) — day-to-day commands

```bash
gh repo clone owner/repo         # clone by owner/name, no need to remember the full URL
gh repo view --web               # open the current repo's page in your browser
gh issue create                  # interactive prompt to file a new issue
gh issue list --label bug
gh pr create                     # open a pull request (full detail in Chapter 10)
gh pr list
gh pr checkout 42                 # check out PR #42 locally, as a real branch, to test it
gh pr view 42 --web
gh pr status                      # PRs you've opened, and PRs waiting on your review
gh run list                       # recent GitHub Actions workflow runs (Chapter 17)
gh run watch                      # live-follow a running workflow
```

`gh` is worth building as a habit specifically because it removes constant browser-switching from your workflow — you can file an issue, open a PR, and check whether CI passed without ever leaving the terminal.

## 9.6 GitHub-flavored Markdown — the features beyond plain Markdown

Since your READMEs, issues, PRs, and comments all render this, and since these very documentation files use several of these features, it's worth knowing what's available beyond standard Markdown:

```markdown
- [x] Completed task
- [ ] Pending task              <- interactive checkboxes, clickable directly in the rendered page

@username                        <- mentions, triggers a notification

#42                              <- auto-links to issue/PR #42 in the same repo
owner/repo#42                    <- auto-links across repos

`inline code`
​```python
def hello(): ...
​```                              <- fenced code blocks with syntax highlighting

> [!NOTE]
> Renders as a colored callout box (also: TIP, IMPORTANT, WARNING, CAUTION) — used throughout this course.
```

Pro tip: click any line number in a file on GitHub, then shift-click a range, and copy the URL — you get a **permalink to those exact lines**, perfect for pointing a teammate at specific code in a chat message or issue.

## 9.7 Notifications and watching

**Watch** a repo (bell icon, top of any repo page) to get notified of new issues/PRs/discussions. Options range from "All Activity" to "Custom" (e.g., only when you're mentioned) to fully muted. Your notification inbox lives at **github.com/notifications** — worth checking rather than relying purely on email, especially on active repos.

You now know your way around the GitHub product itself. Next: [Chapter 10 — Pull Requests & Code Review](10-pull-requests-and-code-review-workflow.md) — where Git and GitHub combine into the actual day-to-day loop of professional software development.
