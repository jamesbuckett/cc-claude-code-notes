# CLAUDE CODE Install Guide

Personal install guide for Claude Code on Windows 11, Ubuntu 26.04 LTS, and macOS, plus the cross-OS configuration that follows.

## Contents

- [Chrome preferences](#chrome-preferences)
- [Install](#install)
  - [Windows 11](#windows-11)
  - [Ubuntu 26.04](#ubuntu-2604)
  - [macOS](#macos)
- [Claude Code setup](#claude-code-setup)
  - [`settings.json`](#settingsjson)
  - [`statusline.sh`](#statuslinesh)
  - [Plugins](#plugins)
  - [Model and effort](#model-and-effort)
- [Git and GitHub](#git-and-github)
- [Shell environment](#shell-environment)
- [Verify install](#verify-install)

---

## Chrome preferences

- Font: Anthropic Sans

---

## Install

### Windows 11

Claude Code's tooling assumes a Unix shell, so **WSL2 with Ubuntu** is the most ergonomic path. Native Windows works for quick PowerShell-only use.

**Option A — WSL2 with Ubuntu 26.04 (recommended).** From an elevated PowerShell:

```powershell
wsl --install -d Ubuntu-26.04   # use -d Ubuntu for the default LTS image
```

Reboot if prompted, complete the Ubuntu user setup, then follow the [Ubuntu 26.04 instructions](#ubuntu-2604) inside the Ubuntu shell.

Pair this with **Windows Terminal** (preinstalled on Windows 11) and **VS Code with the WSL extension** for editing files inside WSL.

**Option B — Native Windows install.** winget ships with Windows 11 22H2+. PowerShell 5.1 is the default; PowerShell 7+ is recommended:

```powershell
winget install --id Anthropic.Claude -e          # Or: irm https://claude.ai/install.ps1 | iex
winget install --id Git.Git -e
winget install --id GitHub.cli -e
winget install --id Microsoft.PowerShell -e      # PowerShell 7+ (separate from built-in 5.1)
```

For Node, use a version manager (do not rely on a system-wide installer):

```powershell
# x64:
winget install --id CoreyButler.NVMforWindows -e

# ARM64 (e.g., Snapdragon laptops) — nvm-windows is x64-only; fnm supports both archs:
winget install --id Schniz.fnm -e
```

Then install LTS Node in a new PowerShell window:

```powershell
nvm install lts ; nvm use lts    # nvm-windows
# or
fnm install --lts ; fnm use lts  # fnm
node --version
npm --version
```

Set git line-ending handling and long-path support before cloning anything:

```powershell
git config --global core.autocrlf input
git config --global core.longpaths true
```

After install, continue at [Claude Code setup](#claude-code-setup).

### Ubuntu 26.04

Tested on a fresh Ubuntu 26.04 LTS install (amd64 or arm64). Other Debian-family distros likely work, but commands below assume `apt` and `dpkg`.

**Enable the `universe` repository.** Default on Desktop, sometimes missing on Server / minimal cloud images. Several packages below live in `universe`:

```bash
sudo add-apt-repository universe -y
sudo apt update
```

**Required tooling.** Claude Code itself only needs `curl`, `git`, `jq`, and Node — the rest are commonly-used adjuncts:

```bash
sudo apt install -y \
  build-essential git curl wget jq unzip zip tar \
  ca-certificates gnupg \
  python3-venv python3-pip pipx python-is-python3 \
  openssh-client bash-completion \
  pre-commit direnv
```

Put `~/.local/bin` on PATH (Ubuntu 26.04 enforces PEP 668 — `pip install` outside a venv is blocked, so tools land in user space via `pipx`):

```bash
pipx ensurepath
exec $SHELL -l
```

**Optional CLI niceties:**

```bash
sudo apt install -y bat fd-find ripgrep fzf htop duf graphviz plantuml pandoc
```

On Ubuntu, `bat` and `fd-find` ship as `batcat` / `fdfind` due to binary-name collisions with Debian's `bat`/`fd`. Alias if you want the upstream names:

```bash
echo "alias bat='batcat'" >> ~/.bashrc
echo "alias fd='fdfind'"  >> ~/.bashrc
```

**GitHub CLI from the upstream apt repository** (more current than the `universe` build, signed by GitHub's key):

```bash
sudo mkdir -p -m 755 /etc/apt/keyrings
wget -qO- https://cli.github.com/packages/githubcli-archive-keyring.gpg | \
  sudo tee /etc/apt/keyrings/githubcli-archive-keyring.gpg > /dev/null
sudo chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | \
  sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null
sudo apt update && sudo apt install -y gh
```

**nvm and the latest LTS Node** (do not use Ubuntu's apt Node — it lags and pulls in global-install footguns):

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
exec $SHELL -l            # restart the shell so NVM_DIR loads cleanly
nvm install --lts
nvm use --lts
node --version
npm --version
```

**Install Claude Code.** Inspect the install script before running it:

```bash
curl -fsSL https://claude.ai/install.sh -o /tmp/claude-install.sh
less /tmp/claude-install.sh
bash /tmp/claude-install.sh
```

(The upstream one-liner `curl -fsSL https://claude.ai/install.sh | bash` works if you trust the source.)

After install, continue at [Claude Code setup](#claude-code-setup).

### macOS

Same install script as Linux. Prerequisites via Homebrew:

```bash
brew install jq git gh nvm
curl -fsSL https://claude.ai/install.sh -o /tmp/claude-install.sh
less /tmp/claude-install.sh
bash /tmp/claude-install.sh
```

After install, continue at [Claude Code setup](#claude-code-setup).

---

## Claude Code setup

The following applies on any OS. First, run Claude Code once so it seeds `~/.claude/`:

```bash
claude --version
```

### `settings.json`

Back up the default before editing (no-op if it doesn't exist yet):

```bash
[ -f ~/.claude/settings.json ] && cp ~/.claude/settings.json ~/.claude/settings.json-original
```

Write the statusline wiring with a heredoc — avoids paste/auto-indent corruption:

```bash
cat > ~/.claude/settings.json <<'EOF'
{
  "statusLine": {
    "type": "command",
    "command": "~/.claude/statusline.sh"
  }
}
EOF
```

### `statusline.sh`

Fetch a pinned upstream release:

```bash
curl -fsSL https://raw.githubusercontent.com/daniel3303/ClaudeCodeStatusLine/v1.4.2/statusline.sh \
  -o ~/.claude/statusline.sh
chmod +x ~/.claude/statusline.sh
```


### Plugins

- `claude-md-management`
- `frontend-design`
- `playwright`
- `superpowers`

### Model and effort

- Model: Opus 4.7 with 1M context
- Effort:
  - Normal work: `xhigh`
  - Extreme work: `max`

---

## Git and GitHub

Set identity once globally:

```bash
git config --global user.name "jamesbuckett"
git config --global user.email "james.buckett@gmail.com"
```

For credential storage, use `gh` — it configures a credential helper backed by your GitHub authentication, which is safer than `credential.helper store` (plaintext on disk) or `cache` (short-lived in-memory):

```bash
gh auth login
gh auth setup-git
gh auth status
```

Pick **GitHub.com → HTTPS → Login with a web browser** and follow the device-code prompt.

For non-GitHub remotes, prefer a system credential helper: `libsecret` (Linux), `osxkeychain` (macOS), or `wincred` (Windows) — not `store`.

---

## Shell environment

Add the following to `~/.bashrc` (or `~/.zshrc` on macOS):

```bash
#######################################################################################################
## Claude Code aliases
#######################################################################################################

# Default: plan mode prompts before every tool action.
alias claude="claude --permission-mode plan"

# Short alias; inherits plan mode.
alias c="claude"

# Bypass-all-permissions — disposable sandboxes only.
# bypassPermissions runs every tool without prompting; understand the
# blast radius before invoking this in a repo you care about.
alias cyolo="claude --permission-mode bypassPermissions"
```

**Anthropic API key (optional).** Claude Code uses OAuth via your Pro/Max subscription by default. Setting `ANTHROPIC_API_KEY` switches billing to the API — only set it if that is what you want. Do not paste the literal key into `.bashrc`; load it from a secret manager:

```bash
# Example using pass (passwordstore.org). Substitute your secret manager of choice.
# export ANTHROPIC_API_KEY="$(pass anthropic/api-key 2>/dev/null)"
```

Reload the shell after edits:

```bash
exec $SHELL -l
```

---

## Verify install

```bash
claude --version    # confirms the binary is on PATH
claude              # launches; run /login if not already authenticated
```

Inside Claude Code, the statusline should show model, context usage, and 5h/7d rate-limit bars. If blank, check that `~/.claude/statusline.sh` is executable (`ls -l ~/.claude/statusline.sh`) and that `settings.json` points at it.

## [CodeBurn](https://github.com/getagentseal/codeburn)

```bash
npm install -g codeburn
```

```bash
codeburn
codeburn optimize
```

Copy and paste codeburn optimize into Claude Code.
