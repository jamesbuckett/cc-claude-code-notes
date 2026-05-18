# Plugins

## Anthropic Plugins

- `claude-md-management`
- `frontend-design`
- `firecrawl`
  - See below for CLI and API key   
- `playwright`
- `skill-creator`
- `superpowers`

## Other Plugins

### [Caveman](https://github.com/juliusbrussee/caveman)
- Install: `curl -fsSL https://raw.githubusercontent.com/JuliusBrussee/caveman/main/install.sh | bash`
  - `run /caveman` in Claude Code
- [Issue](https://github.com/JuliusBrussee/caveman/issues/353)
  - Uninstall: `npx -y github:JuliusBrussee/caveman -- --uninstall`

### [Exa](https://exa.ai/mcp) - Better Web Search 
- Install: `claude mcp add --transport http exa https://mcp.exa.ai/mcp`
- Export EXA_API_KEY="xxx"
- Usage:
  - "What happened with OpenAI today?"
  - "Read this blog post and summarize it"
  - "Recent papers on RAG"
  - "Compare Notion vs Coda vs Slite"


### [Firecrawl](https://github.com/firecrawl/firecrawl-claude-plugin) - Get the Web Resources
- Install CLI: `npm install -g firecrawl-cli`
- 'firecrawl login --api-key "fc-YOUR-API-KEY"'
  - Login successful!
- Export FIRECRAWL_API_KEY=fc-YOUR-API-KEY

### [Compound Engineering](https://github.com/EveryInc/compound-engineering-plugin)
- /plugin marketplace add EveryInc/compound-engineering-plugin
- /plugin install compound-engineering

Compound engineering inverts this. 80% is in planning and review, 20% is in execution:
- Plan thoroughly before writing code with `/ce-brainstorm` and `/ce-plan`
- Review to catch issues and calibrate judgment with `/ce-code-review` and `/ce-doc-review`
- Codify knowledge so it is reusable with `/ce-compound`
- Keep quality high so future changes are easy
