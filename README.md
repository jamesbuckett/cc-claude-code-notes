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

## Contributing

Issues and pull requests welcome. Please open an issue first to discuss substantial changes.

## License

[MIT](LICENSE) © 2026 James Buckett
