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

### Caveman
- [caveman](https://github.com/juliusbrussee/caveman)
  - Install: `curl -fsSL https://raw.githubusercontent.com/JuliusBrussee/caveman/main/install.sh | bash`
    - `run /caveman` in Claude Code
  - [Issue](https://github.com/JuliusBrussee/caveman/issues/353)
    - Uninstall: `npx -y github:JuliusBrussee/caveman -- --uninstall`

### [Exa MCP](https://exa.ai/mcp)
- Install: `claude mcp add --transport http exa https://mcp.exa.ai/mcp`
- Export EXA_API_KEY="xxx"
- Usage:
  - "What happened with OpenAI today?"
  - "Read this blog post and summarize it"
  - "Recent papers on RAG"
  - "Compare Notion vs Coda vs Slite"


### [Firecrawl Plugin for Claude Code](https://github.com/firecrawl/firecrawl-claude-plugin)
- Install CLI: `npm install -g firecrawl-cli`
- 'firecrawl login --api-key "fc-YOUR-API-KEY"'
  - Login successful!
- Export FIRECRAWL_API_KEY=fc-YOUR-API-KEY
