# Claude Code Cheat Sheet

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![GitHub stars](https://img.shields.io/github/stars/jamesbuckett/cc-claude-code-notes?style=social)](https://github.com/jamesbuckett/cc-claude-code-notes/stargazers)
[![Last commit](https://img.shields.io/github/last-commit/jamesbuckett/cc-claude-code-notes)](https://github.com/jamesbuckett/cc-claude-code-notes/commits)
[![Open issues](https://img.shields.io/github/issues/jamesbuckett/cc-claude-code-notes)](https://github.com/jamesbuckett/cc-claude-code-notes/issues)

> Personal notes on using Claude Code — install steps, plugin list, prompting approach, and shareable config files.

## About

Documents how I install, configure, and prompt Claude Code across Windows 11, Ubuntu 26.04, and macOS. Covers the install flow, enabled plugins, a slash-command cheat sheet, and the shareable config files (`CLAUDE.md`, `settings.json`) that get reused across machines. Published as a personal reference, but adaptable for anyone setting up Claude Code from scratch.

## Usage

Guides:

- **[Install Guide](cc-install.md)** — installing Claude Code on Windows 11, Ubuntu, and macOS, plus cross-OS configuration and adjacent CLI tooling (e.g. `yt-dlp` for the YouTube rip skill).
- **[Plugins](cc-plugins.md)** — Anthropic and community plugins I have enabled.
- **[Prompting Claude Code](cc-prompting-claude-code.md)** — how Anthropic engineers actually prompt Claude Code (skills, not one-off prompts).
- **[Slash Commands](cc-slash-commands.md)** — handy built-in and custom slash commands.

Config files for direct reuse:

- [`files/CLAUDE.md`](files/CLAUDE.md) — global behavioral guidelines that get merged into every project.
- [`files/settings.json`](files/settings.json) — Claude Code `settings.json` with plugins, status line, and the session-start `git pull` hook.

Personal skills live under [`files/skills/`](files/skills/) — none published yet.

## Project Structure

```
.
├── cc-install.md                 # install guide for Windows / Ubuntu / macOS
├── cc-plugins.md                 # enabled plugins
├── cc-prompting-claude-code.md   # prompting approach and takeaways
├── cc-slash-commands.md          # slash command cheat sheet
└── files/                        # shareable config
    ├── CLAUDE.md
    ├── settings.json
    └── skills/
```

## Prompting Takeaways

Distilled from [cc-prompting-claude-code.md](cc-prompting-claude-code.md) — the shift Anthropic engineers make when prompting Claude Code.

### Mental Model

Three layers of the stack: **Model** (Claude) → **Agents + prompts** (where most people stop) → **Skills** (the application layer worth controlling). A skill is a folder packaging composable procedural knowledge that Claude invokes by name or auto-routes to by description.

### The Four Rules

1. **Prompt skills, not Claude.** One-off prompts evaporate when the chat closes. Skills persist and compound.
2. **Tools are the unlock, not instructions.** A skill has three layers — *description*, *instructions*, *tools*. Most users obsess over instructions and ship bare-bones tools. The leverage lives in scripts, API calls, and reference files the skill can run.
3. **Compose, don't customize.** Build small focused skills (`/youtube-idea-research`, `/linkedin-post`) that call each other — not a mega `/content-creation` skill. Issues are easier to spot, improvements compound, and reuse beats rebuild.
4. **Skills get smarter every session.** After each run ask: _"one-time fix, or should this live in the skill forever?"_ The 30-second post-run edit is where compounding comes from.

### Two Patterns Worth Knowing

- **Save scripts inside skills.** Code is deterministic; AI is not. If it can be code, make it code — and let AI write the code for you once.
- **Control who invokes what.** `user-invocable: false` hides a skill from the slash menu (agents only). `disable-model-invocation: true` blocks the model from running it (humans only — good for risky actions).

### So What

- Stop accumulating prompts; start accumulating skills.
- Audit existing skills for the tools layer — a skill that is 95% instructions is leaving leverage on the table.
- Split any skill whose name contains more than one verb.

## Contributing

Issues and pull requests welcome. Please open an issue first to discuss substantial changes.

## License

[MIT](LICENSE) © 2026 James Buckett
