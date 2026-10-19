# Chapter 2 — Installation & Configuration

## 2.1 Installing Git

**Windows**
Download and run the installer from [git-scm.com](https://git-scm.com/download/win). During setup, the defaults are fine for 95% of people, but two screens are worth choosing deliberately:
- **"Adjusting your PATH environment"** → choose *"Git from the command line and also from 3rd-party software"*.
- **"Choosing the default editor"** → pick VS Code if you have it installed (`Use Visual Studio Code as Git's default editor`), otherwise Notepad temporarily — you can change this later (2.4).

This also installs **Git Bash**, a Unix-style terminal — many tutorials online assume Bash syntax, so having it available on Windows is genuinely useful even if you mostly use PowerShell.

**macOS**
```bash
brew install git
# or simply run `git` in Terminal once — macOS offers to install Xcode Command Line Tools, which includes Git
```

**Linux**
```bash
sudo apt install git      # Debian/Ubuntu
sudo dnf install git      # Fedora
sudo pacman -S git        # Arch
```

**Verify it worked, anywhere:**
```bash
$ git --version
git version 2.43.0
```

## 2.2 The three levels of configuration

Git configuration is layered — a setting can be defined at three levels, and a more specific level always wins:

| Level | Flag | Stored in | Applies to |
|---|---|---|---|
| System | `--system` | Once per machine (all users) | Everyone on this computer |
| Global | `--global` | `~/.gitconfig` (your user folder) | Every repo *you* work in |
| Local | (no flag, default) | `.git/config` inside one repo | Just that one repository |

You'll use `--global` almost always, and drop to local config only for repo-specific overrides (e.g., a work project needing a different commit email than your personal projects).

## 2.3 The two settings you cannot skip

Git refuses to let you commit meaningfully without these — every commit permanently embeds this name and email:

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

> [!IMPORTANT]
> Use the **same email** here that's registered on your GitHub account (or added as a secondary email there) — otherwise GitHub won't link your commits to your profile, and your contribution graph won't credit you. If you want to keep your real email private, GitHub provides a `username@users.noreply.github.com` address in **Settings → Emails** made exactly for this.

## 2.4 Quality-of-life settings worth setting immediately

```bash
# Use the modern default branch name for new repos (industry standard since 2020)
git config --global init.defaultBranch main

# Make `git pull` explicit about HOW it should reconcile histories (see Chapter 8) —
# this avoids Git nagging you about it on every pull
git config --global pull.rebase false     # merge (default; safest starting point)

# Only push the current branch, not every branch that happens to share a name (modern default anyway, but explicit is good)
git config --global push.default simple

# Turn on helpful color output in terminal
git config --global color.ui auto

# Set VS Code as your editor for commit messages and merge conflicts
git config --global core.editor "code --wait"

# Reuse recorded conflict resolutions automatically when the same conflict recurs (huge time-saver during rebases — see Chapter 7)
git config --global rerere.enabled true
```

Check anything, anytime, with:
```bash
git config --list                 # everything currently in effect, most specific wins
git config --list --show-origin   # ...and which file each setting came from
git config user.name              # just one value
```

## 2.5 Aliases: shortcuts a 20-year veteran actually uses daily

Aliases are custom shortcuts for commands you type constantly. These four are close to universal among experienced Git users:

```bash
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.st status
git config --global alias.cm "commit -m"

# The single most valuable alias: a compact, graphical log
git config --global alias.lg "log --oneline --graph --decorate --all"
```

Now `git lg` gives you an instant visual of your whole commit graph:
```
* a1b2c3d (HEAD -> main, origin/main) Add login validation
| * 9f8e7d6 (feature/signup) WIP signup form
|/
* 5c4b3a2 Initial commit
```

## 2.6 Authenticating with GitHub: SSH vs. HTTPS

You have two ways to prove to GitHub's servers that you're allowed to push to a repo. Pick **one** and set it up properly now — most early frustration ("it keeps asking for my password and rejecting it!") comes from GitHub having removed plain password authentication for Git operations back in 2021.

### Option A — SSH keys (recommended; set up once, never type a credential again)

```bash
# 1. Generate a key pair (press Enter through the prompts for defaults; a passphrase is optional but recommended)
ssh-keygen -t ed25519 -C "you@example.com"

# 2. Start the ssh-agent and add your new key to it
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# 3. Copy your PUBLIC key (never share the private one!) to the clipboard
cat ~/.ssh/id_ed25519.pub
```

Paste that public key into **GitHub → Settings → SSH and GPG keys → New SSH key**. Then test it:

```bash
$ ssh -T git@github.com
Hi <your-username>! You've successfully authenticated, but GitHub does not provide shell access.
```

From now on, clone repos using the SSH URL (`git@github.com:user/repo.git`) rather than the HTTPS one, and pushes/pulls will authenticate silently via your key.

### Option B — HTTPS + a Personal Access Token (PAT)

GitHub no longer accepts your account password for Git operations over HTTPS — you generate a **token** that acts as a password instead:

1. **GitHub → Settings → Developer settings → Personal access tokens → Fine-grained tokens → Generate new token.**
2. Scope it to the specific repos and permissions you need, set an expiry.
3. When Git prompts for a password over HTTPS, paste the token instead.

Install a **credential helper** so you don't re-enter it constantly:
```bash
git config --global credential.helper manager   # Windows (Git Credential Manager, bundled with Git for Windows)
git config --global credential.helper osxkeychain  # macOS
```

> [!NOTE]
> SSH vs. HTTPS is a one-time infrastructure decision, not a daily one — most professional developers set up SSH once and forget it exists. If you're ever on a locked-down corporate network that blocks SSH's port (22), HTTPS + PAT is the fallback.

## 2.7 The GitHub CLI (`gh`) — worth installing now

`gh` is GitHub's official command-line tool. It lets you create repos, open pull requests, review code, and check CI status without leaving the terminal — you'll use it heavily from Chapter 9 onward.

```bash
winget install --id GitHub.cli      # Windows
brew install gh                     # macOS

gh auth login       # interactive one-time browser-based login
gh auth status       # verify you're logged in
```

## 2.8 A sane starter `.gitconfig`

After the steps above, your `~/.gitconfig` will look roughly like this — feel free to copy this as a reference (yours will differ slightly based on choices made):

```ini
[user]
    name = Your Name
    email = you@example.com
[init]
    defaultBranch = main
[core]
    editor = code --wait
[color]
    ui = auto
[pull]
    rebase = false
[push]
    default = simple
[rerere]
    enabled = true
[alias]
    co = checkout
    br = branch
    st = status
    cm = commit -m
    lg = log --oneline --graph --decorate --all
```

You're fully set up. Next: [Chapter 3 — Core Workflow Basics](03-core-workflow-basics.md), where you'll make your first real commits and actually see the three-trees model from Chapter 1 in action.
