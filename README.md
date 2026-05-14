# Claude Code

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

Fetch a pinned upstream release rather than pasting the script body:

```bash
curl -fsSL https://raw.githubusercontent.com/daniel3303/ClaudeCodeStatusLine/v1.4.2/statusline.sh \
  -o ~/.claude/statusline.sh
chmod +x ~/.claude/statusline.sh
```

<details>
<summary>Full <code>statusline.sh</code></summary>

```bash
#!/bin/bash
# Source: https://github.com/daniel3303/ClaudeCodeStatusLine
# Single line: Model | tokens | %used | %remain | think | 5h bar @reset | 7d bar @reset | extra

set -f  # disable globbing
VERSION="1.4.2"

input=$(cat)

if [ -z "$input" ]; then
    printf "Claude"
    exit 0
fi

# ANSI colors matching oh-my-posh theme
blue='\033[38;2;0;153;255m'
orange='\033[38;2;255;176;85m'
green='\033[38;2;0;160;0m'
cyan='\033[38;2;46;149;153m'
red='\033[38;2;255;85;85m'
yellow='\033[38;2;230;200;0m'
purple='\033[38;2;167;139;250m'
white='\033[38;2;220;220;220m'
dim='\033[2m'
reset='\033[0m'

# Format token counts (e.g., 50k / 200k)
format_tokens() {
    local num=$1
    if [ "$num" -ge 1000000 ]; then
        awk "BEGIN {v=sprintf(\"%.1f\",$num/1000000)+0; if(v==int(v)) printf \"%dm\",v; else printf \"%.1fm\",v}"
    elif [ "$num" -ge 1000 ]; then
        awk "BEGIN {printf \"%.0fk\", $num / 1000}"
    else
        printf "%d" "$num"
    fi
}

# Format number with commas (e.g., 134,938)
format_commas() {
    printf "%'d" "$1"
}

# Return color escape based on usage percentage
# Usage: usage_color <pct>
usage_color() {
    local pct=$1
    if [ "$pct" -ge 90 ]; then echo "$red"
    elif [ "$pct" -ge 70 ]; then echo "$orange"
    elif [ "$pct" -ge 50 ]; then echo "$yellow"
    else echo "$green"
    fi
}

# Resolve config directory: CLAUDE_CONFIG_DIR (set by alias) or default ~/.claude
claude_config_dir="${CLAUDE_CONFIG_DIR:-$HOME/.claude}"

# Return 0 (true) if $1 > $2 using semantic versioning
version_gt() {
    local a="${1#v}" b="${2#v}"
    local IFS='.'
    read -r a1 a2 a3 <<< "$a"
    read -r b1 b2 b3 <<< "$b"
    a1=${a1:-0}; a2=${a2:-0}; a3=${a3:-0}
    b1=${b1:-0}; b2=${b2:-0}; b3=${b3:-0}
    [ "$a1" -gt "$b1" ] 2>/dev/null && return 0
    [ "$a1" -lt "$b1" ] 2>/dev/null && return 1
    [ "$a2" -gt "$b2" ] 2>/dev/null && return 0
    [ "$a2" -lt "$b2" ] 2>/dev/null && return 1
    [ "$a3" -gt "$b3" ] 2>/dev/null && return 0
    return 1
}
# ===== Extract data from JSON =====
model_name=$(echo "$input" | jq -r '.model.display_name // "Claude"')
model_name=$(echo "$model_name" | sed 's/ *(\([0-9.]*[kKmM]*\) context)/ \1/')  # "(1M context)" → "1M"

# Context window
size=$(echo "$input" | jq -r '.context_window.context_window_size // 200000')
[ "$size" -eq 0 ] 2>/dev/null && size=200000

# Token usage
input_tokens=$(echo "$input" | jq -r '.context_window.current_usage.input_tokens // 0')
cache_create=$(echo "$input" | jq -r '.context_window.current_usage.cache_creation_input_tokens // 0')
cache_read=$(echo "$input" | jq -r '.context_window.current_usage.cache_read_input_tokens // 0')
current=$(( input_tokens + cache_create + cache_read ))

used_tokens=$(format_tokens $current)
total_tokens=$(format_tokens $size)

if [ "$size" -gt 0 ]; then
    pct_used=$(( current * 100 / size ))
else
    pct_used=0
fi
pct_remain=$(( 100 - pct_used ))

used_comma=$(format_commas $current)
remain_comma=$(format_commas $(( size - current )))

# Check reasoning effort
settings_path="$claude_config_dir/settings.json"
effort_level="medium"
if [ -n "$CLAUDE_CODE_EFFORT_LEVEL" ]; then
    effort_level="$CLAUDE_CODE_EFFORT_LEVEL"
elif [ -f "$settings_path" ]; then
    effort_val=$(jq -r '.effortLevel // empty' "$settings_path" 2>/dev/null)
    [ -n "$effort_val" ] && effort_level="$effort_val"
fi

# ===== Build single-line output =====
out=""
out+="${blue}${model_name}${reset}"

# Current working directory
cwd=$(echo "$input" | jq -r '.cwd // empty')
if [ -n "$cwd" ]; then
    display_dir="${cwd##*/}"
    git_branch=$(git -C "${cwd}" rev-parse --abbrev-ref HEAD 2>/dev/null)
    out+=" ${dim}|${reset} "
    out+="${cyan}${display_dir}${reset}"
    if [ -n "$git_branch" ]; then
        out+="${dim}@${reset}${green}${git_branch}${reset}"
        git_stat=$(git -C "${cwd}" diff --numstat 2>/dev/null | awk '{a+=$1; d+=$2} END {if (a+d>0) printf "+%d -%d", a, d}')
        [ -n "$git_stat" ] && out+=" ${dim}(${reset}${green}${git_stat%% *}${reset} ${red}${git_stat##* }${reset}${dim})${reset}"
    fi
fi

out+=" ${dim}|${reset} "
out+="${orange}${used_tokens}/${total_tokens}${reset} ${dim}(${reset}${green}${pct_used}%${reset}${dim})${reset}"
out+=" ${dim}|${reset} "
out+="effort: "
case "$effort_level" in
    low)    out+="${dim}${effort_level}${reset}" ;;
    medium) out+="${orange}med${reset}" ;;
    high)   out+="${green}${effort_level}${reset}" ;;
    xhigh)  out+="${purple}${effort_level}${reset}" ;;
    max)    out+="${red}${effort_level}${reset}" ;;
    *)      out+="${green}${effort_level}${reset}" ;;
esac

# ===== Cross-platform OAuth token resolution (from statusline.sh) =====
# Tries credential sources in order: env var → macOS Keychain → Linux creds file → GNOME Keyring
get_oauth_token() {
    local token=""

    # 1. Explicit env var override
    if [ -n "$CLAUDE_CODE_OAUTH_TOKEN" ]; then
        echo "$CLAUDE_CODE_OAUTH_TOKEN"
        return 0
    fi

    # 2. macOS Keychain (Claude Code appends a SHA256 hash of CLAUDE_CONFIG_DIR to the service name)
    if command -v security >/dev/null 2>&1; then
        local keychain_svc="Claude Code-credentials"
        if [ -n "$CLAUDE_CONFIG_DIR" ]; then
            local dir_hash
            dir_hash=$(echo -n "$CLAUDE_CONFIG_DIR" | shasum -a 256 | cut -c1-8)
            keychain_svc="Claude Code-credentials-${dir_hash}"
        fi
        local blob
        blob=$(security find-generic-password -s "$keychain_svc" -w 2>/dev/null)
        if [ -n "$blob" ]; then
            token=$(echo "$blob" | jq -r '.claudeAiOauth.accessToken // empty' 2>/dev/null)
            if [ -n "$token" ] && [ "$token" != "null" ]; then
                echo "$token"
                return 0
            fi
        fi
    fi

    # 3. Linux credentials file
    local creds_file="${claude_config_dir}/.credentials.json"
    if [ -f "$creds_file" ]; then
        token=$(jq -r '.claudeAiOauth.accessToken // empty' "$creds_file" 2>/dev/null)
        if [ -n "$token" ] && [ "$token" != "null" ]; then
            echo "$token"
            return 0
        fi
    fi

    # 4. GNOME Keyring via secret-tool
    if command -v secret-tool >/dev/null 2>&1; then
        local blob
        blob=$(timeout 2 secret-tool lookup service "Claude Code-credentials" 2>/dev/null)
        if [ -n "$blob" ]; then
            token=$(echo "$blob" | jq -r '.claudeAiOauth.accessToken // empty' 2>/dev/null)
            if [ -n "$token" ] && [ "$token" != "null" ]; then
                echo "$token"
                return 0
            fi
        fi
    fi

    echo ""
}

# ===== LINE 2 & 3: Usage limits with progress bars =====
# First, try to use rate_limits data provided directly by Claude Code in the JSON input.
# This is the most reliable source — no OAuth token or API call required.
builtin_five_hour_pct=$(echo "$input" | jq -r '.rate_limits.five_hour.used_percentage // empty')
builtin_five_hour_reset=$(echo "$input" | jq -r '.rate_limits.five_hour.resets_at // empty')
builtin_seven_day_pct=$(echo "$input" | jq -r '.rate_limits.seven_day.used_percentage // empty')
builtin_seven_day_reset=$(echo "$input" | jq -r '.rate_limits.seven_day.resets_at // empty')

use_builtin=false
if [ -n "$builtin_five_hour_pct" ] || [ -n "$builtin_seven_day_pct" ]; then
    use_builtin=true
fi

# Cache setup — shared across all Claude Code instances to avoid rate limits
claude_config_dir_hash=$(echo -n "$claude_config_dir" | shasum -a 256 2>/dev/null || echo -n "$claude_config_dir" | sha256sum 2>/dev/null)
claude_config_dir_hash=$(echo "$claude_config_dir_hash" | cut -c1-8)
cache_file="/tmp/claude/statusline-usage-cache-${claude_config_dir_hash}.json"
cache_max_age=60  # seconds between API calls
mkdir -p /tmp/claude

needs_refresh=true
usage_data=""

# Always load cache — used as primary source for API path, and as fallback when builtin reports zero
if [ -f "$cache_file" ] && [ -s "$cache_file" ]; then
    cache_mtime=$(stat -c %Y "$cache_file" 2>/dev/null || stat -f %m "$cache_file" 2>/dev/null)
    now=$(date +%s)
    cache_age=$(( now - cache_mtime ))
    if [ "$cache_age" -lt "$cache_max_age" ]; then
        needs_refresh=false
    fi
    usage_data=$(cat "$cache_file" 2>/dev/null)
fi

# When builtin values are all zero AND reset timestamps are missing, it likely indicates
# an API failure on Claude's side — fall through to cached data instead of displaying
# misleading 0%. Genuine zero responses (after a billing reset) still include valid
# resets_at timestamps, so we trust those.
effective_builtin=false
if $use_builtin; then
    # Trust builtin if any percentage is non-zero
    if { [ -n "$builtin_five_hour_pct" ] && [ "$(printf '%.0f' "$builtin_five_hour_pct" 2>/dev/null)" != "0" ]; } || \
       { [ -n "$builtin_seven_day_pct" ] && [ "$(printf '%.0f' "$builtin_seven_day_pct" 2>/dev/null)" != "0" ]; }; then
        effective_builtin=true
    fi
    # Also trust if reset timestamps are present — genuine zero responses include valid reset times
    if ! $effective_builtin; then
        if { [ -n "$builtin_five_hour_reset" ] && [ "$builtin_five_hour_reset" != "null" ] && [ "$builtin_five_hour_reset" != "0" ]; } || \
           { [ -n "$builtin_seven_day_reset" ] && [ "$builtin_seven_day_reset" != "null" ] && [ "$builtin_seven_day_reset" != "0" ]; }; then
            effective_builtin=true
        fi
    fi
fi

# Refresh API cache when stale — runs regardless of builtin rate_limits because
# extra_usage is only exposed through the OAuth usage endpoint (not stdin JSON).
# Throttled to cache_max_age and stampede-locked via touch for shared panes.
if $needs_refresh; then
    touch "$cache_file"  # stampede lock: prevent parallel panes from fetching simultaneously
    token=$(get_oauth_token)
    if [ -n "$token" ] && [ "$token" != "null" ]; then
        response=$(curl -s --max-time 10 \
            -H "Accept: application/json" \
            -H "Content-Type: application/json" \
            -H "Authorization: Bearer $token" \
            -H "anthropic-beta: oauth-2025-04-20" \
            -H "User-Agent: claude-code/2.1.34" \
            "https://api.anthropic.com/api/oauth/usage" 2>/dev/null)
        # Only cache valid usage responses (not error/rate-limit JSON)
        if [ -n "$response" ] && echo "$response" | jq -e '.five_hour' >/dev/null 2>&1; then
            usage_data="$response"
            echo "$response" > "$cache_file"
        fi
    fi
    # Remove the stampede sentinel if the fetch failed to produce valid JSON —
    # otherwise an empty cache file would suppress retries for a full cache_max_age window.
    [ -f "$cache_file" ] && [ ! -s "$cache_file" ] && rm -f "$cache_file"
fi

# Cross-platform ISO to epoch conversion
# Converts ISO 8601 timestamp (e.g. "2025-06-15T12:30:00Z" or "2025-06-15T12:30:00.123+00:00") to epoch seconds.
# Properly handles UTC timestamps and converts to local time.
iso_to_epoch() {
    local iso_str="$1"

    # Try GNU date first (Linux) — handles ISO 8601 format automatically
    local epoch
    epoch=$(date -d "${iso_str}" +%s 2>/dev/null)
    if [ -n "$epoch" ]; then
        echo "$epoch"
        return 0
    fi

    # BSD date (macOS) - handle various ISO 8601 formats
    local stripped="${iso_str%%.*}"          # Remove fractional seconds (.123456)
    stripped="${stripped%%Z}"                 # Remove trailing Z
    stripped="${stripped%%+*}"               # Remove timezone offset (+00:00)
    stripped="${stripped%%-[0-9][0-9]:[0-9][0-9]}"  # Remove negative timezone offset

    # Check if timestamp is UTC (has Z or +00:00 or -00:00)
    if [[ "$iso_str" == *"Z"* ]] || [[ "$iso_str" == *"+00:00"* ]] || [[ "$iso_str" == *"-00:00"* ]]; then
        # For UTC timestamps, parse with timezone set to UTC
        epoch=$(env TZ=UTC date -j -f "%Y-%m-%dT%H:%M:%S" "$stripped" +%s 2>/dev/null)
    else
        epoch=$(date -j -f "%Y-%m-%dT%H:%M:%S" "$stripped" +%s 2>/dev/null)
    fi

    if [ -n "$epoch" ]; then
        echo "$epoch"
        return 0
    fi

    return 1
}

# Format ISO reset time to compact local time
# Usage: format_reset_time <iso_string> <style: time|datetime|date>
format_reset_time() {
    local iso_str="$1"
    local style="$2"
    { [ -z "$iso_str" ] || [ "$iso_str" = "null" ]; } && return

    # Parse ISO datetime and convert to local time (cross-platform)
    local epoch
    epoch=$(iso_to_epoch "$iso_str")
    [ -z "$epoch" ] && return

    # Format based on style
    # Try GNU date first (Linux), then BSD date (macOS)
    # Previous implementation piped BSD date through sed/tr, which always returned
    # exit code 0 from the last pipe stage, preventing the GNU date fallback from
    # ever executing on Linux.
    local formatted=""
    case "$style" in
        time)
            formatted=$(date -d "@$epoch" +"%H:%M" 2>/dev/null) || \
            formatted=$(date -j -r "$epoch" +"%H:%M" 2>/dev/null)
            ;;
        datetime)
            formatted=$(date -d "@$epoch" +"%b %-d, %H:%M" 2>/dev/null) || \
            formatted=$(date -j -r "$epoch" +"%b %-d, %H:%M" 2>/dev/null)
            ;;
        *)
            formatted=$(date -d "@$epoch" +"%b %-d" 2>/dev/null) || \
            formatted=$(date -j -r "$epoch" +"%b %-d" 2>/dev/null)
            ;;
    esac
    [ -n "$formatted" ] && echo "$formatted"
}

sep=" ${dim}|${reset} "

# Render extra_usage segment from API usage data (not available via stdin rate_limits).
# Appends to the global $out. No-op when data is missing or is_enabled is false.
render_extra_usage() {
    local data="$1"
    [ -z "$data" ] && return
    local enabled
    enabled=$(echo "$data" | jq -r '.extra_usage.is_enabled // false' 2>/dev/null)
    [ "$enabled" != "true" ] && return

    local pct used limit
    pct=$(echo "$data" | jq -r '.extra_usage.utilization // 0' | awk '{printf "%.0f", $1}')
    used=$(echo "$data" | jq -r '.extra_usage.used_credits // 0' | LC_NUMERIC=C awk '{printf "%.2f", $1/100}')
    limit=$(echo "$data" | jq -r '.extra_usage.monthly_limit // 0' | LC_NUMERIC=C awk '{printf "%.2f", $1/100}')

    if [ -n "$used" ] && [ -n "$limit" ] && [[ "$used" != *'$'* ]] && [[ "$limit" != *'$'* ]]; then
        local color
        color=$(usage_color "$pct")
        out+="${sep}${white}extra${reset} ${color}\$${used}/\$${limit}${reset}"
    else
        out+="${sep}${white}extra${reset} ${green}enabled${reset}"
    fi
}

if $effective_builtin; then
    # ---- Use rate_limits data provided directly by Claude Code in JSON input ----
    # resets_at values are Unix epoch integers in this source
    if [ -n "$builtin_five_hour_pct" ]; then
        five_hour_pct=$(printf "%.0f" "$builtin_five_hour_pct")
        five_hour_color=$(usage_color "$five_hour_pct")
        out+="${sep}${white}5h${reset} ${five_hour_color}${five_hour_pct}%${reset}"
        if [ -n "$builtin_five_hour_reset" ] && [ "$builtin_five_hour_reset" != "null" ]; then
            five_hour_reset=$(date -j -r "$builtin_five_hour_reset" +"%H:%M" 2>/dev/null || date -d "@$builtin_five_hour_reset" +"%H:%M" 2>/dev/null)
            [ -n "$five_hour_reset" ] && out+=" ${dim}@${five_hour_reset}${reset}"
        fi
    fi

    if [ -n "$builtin_seven_day_pct" ]; then
        seven_day_pct=$(printf "%.0f" "$builtin_seven_day_pct")
        seven_day_color=$(usage_color "$seven_day_pct")
        out+="${sep}${white}7d${reset} ${seven_day_color}${seven_day_pct}%${reset}"
        if [ -n "$builtin_seven_day_reset" ] && [ "$builtin_seven_day_reset" != "null" ]; then
            seven_day_reset=$(date -j -r "$builtin_seven_day_reset" +"%b %-d, %H:%M" 2>/dev/null || date -d "@$builtin_seven_day_reset" +"%b %-d, %H:%M" 2>/dev/null)
            [ -n "$seven_day_reset" ] && out+=" ${dim}@${seven_day_reset}${reset}"
        fi
    fi

    # Render extra_usage from API cache (stdin rate_limits doesn't expose it)
    render_extra_usage "$usage_data"

    # Cache builtin values so they're available as fallback when API is unavailable.
    # Convert epoch resets_at to ISO 8601 for compatibility with the API-format cache parser.
    # Preserve extra_usage from prior API response so we don't clobber it.
    _fh_reset_json="null"
    if [ -n "$builtin_five_hour_reset" ] && [ "$builtin_five_hour_reset" != "null" ] && [ "$builtin_five_hour_reset" != "0" ]; then
        _fh_iso=$(date -u -r "$builtin_five_hour_reset" +"%Y-%m-%dT%H:%M:%SZ" 2>/dev/null || \
                  date -u -d "@$builtin_five_hour_reset" +"%Y-%m-%dT%H:%M:%SZ" 2>/dev/null)
        [ -n "$_fh_iso" ] && _fh_reset_json="\"$_fh_iso\""
    fi
    _sd_reset_json="null"
    if [ -n "$builtin_seven_day_reset" ] && [ "$builtin_seven_day_reset" != "null" ] && [ "$builtin_seven_day_reset" != "0" ]; then
        _sd_iso=$(date -u -r "$builtin_seven_day_reset" +"%Y-%m-%dT%H:%M:%SZ" 2>/dev/null || \
                  date -u -d "@$builtin_seven_day_reset" +"%Y-%m-%dT%H:%M:%SZ" 2>/dev/null)
        [ -n "$_sd_iso" ] && _sd_reset_json="\"$_sd_iso\""
    fi
    _extra_json=$(echo "$usage_data" | jq -c '.extra_usage // null' 2>/dev/null)
    [ -z "$_extra_json" ] && _extra_json="null"
    printf '{"five_hour":{"utilization":%s,"resets_at":%s},"seven_day":{"utilization":%s,"resets_at":%s},"extra_usage":%s}' \
        "${builtin_five_hour_pct:-0}" "$_fh_reset_json" \
        "${builtin_seven_day_pct:-0}" "$_sd_reset_json" \
        "$_extra_json" > "$cache_file" 2>/dev/null
elif [ -n "$usage_data" ] && echo "$usage_data" | jq -e '.five_hour' >/dev/null 2>&1; then
    # ---- Fall back: API-fetched usage data ----
    # ---- 5-hour (current) ----
    five_hour_pct=$(echo "$usage_data" | jq -r '.five_hour.utilization // 0' | awk '{printf "%.0f", $1}')
    five_hour_reset_iso=$(echo "$usage_data" | jq -r '.five_hour.resets_at // empty')
    five_hour_reset=$(format_reset_time "$five_hour_reset_iso" "time")
    five_hour_color=$(usage_color "$five_hour_pct")

    out+="${sep}${white}5h${reset} ${five_hour_color}${five_hour_pct}%${reset}"
    [ -n "$five_hour_reset" ] && out+=" ${dim}@${five_hour_reset}${reset}"

    # ---- 7-day (weekly) ----
    seven_day_pct=$(echo "$usage_data" | jq -r '.seven_day.utilization // 0' | awk '{printf "%.0f", $1}')
    seven_day_reset_iso=$(echo "$usage_data" | jq -r '.seven_day.resets_at // empty')
    seven_day_reset=$(format_reset_time "$seven_day_reset_iso" "datetime")
    seven_day_color=$(usage_color "$seven_day_pct")

    out+="${sep}${white}7d${reset} ${seven_day_color}${seven_day_pct}%${reset}"
    [ -n "$seven_day_reset" ] && out+=" ${dim}@${seven_day_reset}${reset}"

    render_extra_usage "$usage_data"
else
    # No valid usage data — show placeholders
    out+="${sep}${white}5h${reset} ${dim}-${reset}"
    out+="${sep}${white}7d${reset} ${dim}-${reset}"
fi

# ===== Update check (cached, 24h TTL) =====
version_cache_file="/tmp/claude/statusline-version-cache.json"
version_cache_max_age=86400  # 24 hours

version_needs_refresh=true
version_data=""

if [ -f "$version_cache_file" ]; then
    vc_mtime=$(stat -c %Y "$version_cache_file" 2>/dev/null || stat -f %m "$version_cache_file" 2>/dev/null)
    vc_now=$(date +%s)
    vc_age=$(( vc_now - vc_mtime ))
    if [ "$vc_age" -lt "$version_cache_max_age" ]; then
        version_needs_refresh=false
    fi
    version_data=$(cat "$version_cache_file" 2>/dev/null)
fi

if $version_needs_refresh; then
    touch "$version_cache_file" 2>/dev/null
    vc_response=$(curl -s --max-time 5 \
        -H "Accept: application/vnd.github+json" \
        "https://api.github.com/repos/daniel3303/ClaudeCodeStatusLine/releases/latest" 2>/dev/null)
    if [ -n "$vc_response" ] && echo "$vc_response" | jq -e '.tag_name' >/dev/null 2>&1; then
        version_data="$vc_response"
        echo "$vc_response" > "$version_cache_file"
    elif [ ! -s "$version_cache_file" ]; then
        # Fetch failed and the cache has no usable content — drop the empty
        # stampede lock so the next render retries instead of the fresh mtime
        # suppressing update checks for the full 24h TTL.
        rm -f "$version_cache_file" 2>/dev/null
    fi
fi

update_line=""
if [ -n "$version_data" ]; then
    latest_tag=$(echo "$version_data" | jq -r '.tag_name // empty')
    if [ -n "$latest_tag" ] && version_gt "$latest_tag" "$VERSION"; then
        update_line="\n${dim}Update available: ${latest_tag} → Tell Claude: \"Find my installed status bar and update it\"${reset}"
    fi
fi

# Output
printf "%b" "$out$update_line"

exit 0
```

</details>

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
