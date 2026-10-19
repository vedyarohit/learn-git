# Chapter 17 — GitHub Actions & Automation

## 17.1 Why CI/CD exists, in real-life terms

Before automation, "did the tests pass" meant trusting that every individual developer remembered to run them locally before pushing — and inevitably, someone forgets, and a broken build reaches `main`. **Continuous Integration (CI)** is simply: a server automatically runs your tests/linters/build **every single time** anyone pushes or opens a PR, with zero reliance on human memory. **Continuous Deployment (CD)** extends this: if everything passes, automatically ship the result to real users, again with no manual step to forget. **GitHub Actions** is GitHub's built-in engine for both.

## 17.2 Anatomy of a workflow file

Workflows live in `.github/workflows/*.yml` and are picked up automatically — no separate registration step. Here's a complete, real example, annotated:

```yaml
name: Run Tests                    # Shows up as the workflow's name in the Actions tab

on:                                 # WHEN this workflow runs
  push:
    branches: [main]                #   on every push to main...
  pull_request:                     #   ...and on every PR targeting any branch
  workflow_dispatch:                #   ...and lets you trigger it manually, on demand, from the UI

jobs:                               # One or more independent units of work
  test:
    runs-on: ubuntu-latest           # The virtual machine image to run on
    steps:
      - uses: actions/checkout@v4    # "uses" = run a pre-built, reusable Action from the Marketplace
                                      # this one checks out your repo's code onto the runner
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm install              # "run" = execute a raw shell command
      - run: npm test
```

| Key | Meaning |
|---|---|
| `on` | The trigger(s) — `push`, `pull_request`, `schedule` (cron), `workflow_dispatch` (manual button), `release`, and many more |
| `jobs` | Each job runs on its own fresh virtual machine, in parallel by default (unless you declare dependencies with `needs:`) |
| `steps` | Run in order, within one job, sharing the same filesystem |
| `uses` | Pull in a reusable, versioned Action someone already built (your own, a teammate's, or the public Marketplace) |
| `run` | Execute a literal shell command |

## 17.3 Tying CI into your branch protection

Recall [Chapter 10.6](10-pull-requests-and-code-review-workflow.md#106-branch-protection-rules): a branch protection rule can require specific **status checks** to pass before the merge button unlocks. The job name (`test`, above) is exactly what shows up as a checkable status check in that settings screen — this is the literal mechanism connecting "I wrote a YAML file" to "nobody can merge broken code into main."

## 17.4 Secrets — never hardcode credentials in a workflow

Workflows frequently need credentials (a deploy key, an API token) — these go in **repo Settings → Secrets and variables → Actions**, encrypted at rest, and are referenced (never displayed in logs, automatically redacted even if accidentally printed) via:

```yaml
      - run: ./deploy.sh
        env:
          DEPLOY_TOKEN: ${{ secrets.DEPLOY_TOKEN }}
```

## 17.5 Matrix builds — testing many configurations at once

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18, 20, 22]
    steps:
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm test
```
This single block runs the **entire job three times in parallel**, once per Node version listed — the standard way to confirm your code genuinely works across every version/OS combination you claim to support, without hand-writing three near-identical job definitions.

## 17.6 A realistic deploy-on-merge example

```yaml
name: Deploy
on:
  push:
    branches: [main]     # only runs when code actually lands ON main — i.e., after a PR merges
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm install && npm run build
      - name: Deploy to production
        run: ./scripts/deploy.sh
        env:
          DEPLOY_TOKEN: ${{ secrets.DEPLOY_TOKEN }}
```
This is the literal, complete mechanism behind "GitHub Flow" from Chapter 16.2's promise that merging to `main` means deploying — there's no separate manual deploy step at all; the merge *is* the trigger.

## 17.7 Checking workflow status from the terminal

```bash
gh run list                  # recent workflow runs and their pass/fail status
gh run watch                  # live-follow the currently running workflow
gh run view --log-failed      # jump straight to the logs of whatever failed
gh workflow run deploy.yml    # manually trigger a workflow_dispatch-enabled workflow
```

You now understand the mechanism that turns "someone reviewed and merged my code" into "the tests ran, and the app deployed" — with no human required to remember any individual step. Next: [Chapter 18 — Disaster Recovery Playbook](18-disaster-recovery-playbook.md), a cookbook for the moments things go wrong despite all of this.
