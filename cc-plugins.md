# Claude Code Plugins

Plugins I have installed, grouped by author. Anthropic-maintained first, then third-party.

## Anthropic

- `claude-md-management`
- `frontend-design`
- `playwright`
- `skill-creator`
- `security-guidance`

## Third-party

### [superpowers](https://github.com/obie/superpowers) — Obie Fernandez

A curated bundle of skills (TDD, debugging, brainstorming, code review, etc.) that auto-route based on context.

### [firecrawl](https://github.com/firecrawl/firecrawl-claude-plugin) — Firecrawl

Web scraping, crawling, and search via the Firecrawl API.

```bash
npm install -g firecrawl-cli
firecrawl login --api-key "fc-YOUR-API-KEY"
export FIRECRAWL_API_KEY="fc-YOUR-API-KEY"
```

### [Exa](https://exa.ai/mcp) — Exa

Higher-quality web search via MCP.

```bash
claude mcp add --transport http exa https://mcp.exa.ai/mcp
export EXA_API_KEY="xxx"
```

Sample prompts once installed:

- "What happened with OpenAI today?"
- "Read this blog post and summarize it."
- "Recent papers on RAG."
- "Compare Notion vs Coda vs Slite."

### [Compound Engineering](https://github.com/EveryInc/compound-engineering-plugin) — Every Inc.

A workflow that flips the usual ratio: ~80% planning and review, ~20% execution.

```text
/plugin marketplace add EveryInc/compound-engineering-plugin
/plugin install compound-engineering
```

Typical loop:

- **Plan** thoroughly before writing code: `/ce-brainstorm`, `/ce-plan`
- **Execute**: `/ce-work`
- **Review** to catch issues and calibrate judgment: `/ce-code-review`, `/ce-doc-review`
- **Compound** — codify what was learned so it stays reusable: `/ce-compound`
- Repeat. Use `/ce-commit-push-pr` in place of the manual git workflow.

See the [upstream README](https://github.com/EveryInc/compound-engineering-plugin/blob/main/plugins/compound-engineering/README.md) for full command reference.

### [Caveman](https://github.com/JuliusBrussee/caveman) — Julius Brussee

```bash
curl -fsSL https://raw.githubusercontent.com/JuliusBrussee/caveman/main/install.sh | bash
```

Then run `/caveman` inside Claude Code. Known issue: [#353](https://github.com/JuliusBrussee/caveman/issues/353) — uninstall with:

```bash
npx -y github:JuliusBrussee/caveman -- --uninstall
```

### [BuildPartner.ai](https://buildpartner.ai) — BuildPartner

```bash
curl -fsSL buildpartner.ai/install.sh | sh -s -- --token sk_YOUR_BUILDPARTNER_TOKEN
```

Replace `sk_YOUR_BUILDPARTNER_TOKEN` with your own token from buildpartner.ai. Do not commit the real value.
