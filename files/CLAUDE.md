# CLAUDE.md

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

## 5. Work Defaults

- **Default deliverable**: single-file HTML for explainer, architecture, and education topics. Do not split into multiple files unless explicitly requested.
- **Required skills**: always apply both together — `style-guide` provides the visual chassis (typography, color, layout primitives); `build-educational-site` provides the content architecture (information hierarchy, pedagogical flow, section patterns).
- **Order of operations**: load `style-guide` first to establish the chassis, then `build-educational-site` to populate structure within it.

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.

## 6. Tools

This section describes MCP servers and external tools available to Claude Code in this project. For each tool, prefer the listed use cases over generic alternatives (e.g., default web search).

### 6.1 Exa MCP Server

**Purpose:** Neural/semantic web search and page retrieval. Returns higher-signal results than keyword search engines, with clean markdown extraction.

**Use when:**
- Performing **code lookups** across public repositories or technical documentation
- Conducting **company or people research** (profiles, affiliations, recent activity)
- Running **dated or filtered queries** where recency or domain filtering matters
- Reading a **specific page as clean markdown** for downstream summarization or analysis

**Do not use when:**
- A simple keyword search or direct URL fetch is sufficient
- The query is fully answerable from training data or repo context

---

### 6.2 Firecrawl

**Purpose:** Live web access for search, scrape, crawl, site mapping, and browser interaction. Use when training data is stale or insufficient for the task.

**Use when:**
- Checking **regulator updates** (e.g., MAS, APRA, HKMA, DORA notices)
- Pulling **vendor documentation** for current product behaviour
- Reviewing **CVE feeds** or security advisories
- Gathering **Zero Trust Architecture (ZTA) reference material** (NIST 800-207, CISA ZTMM, vendor whitepapers)
- Retrieving **APAC compliance notices** or other time-sensitive regulatory content
- **Crawling or mapping** a site when more than a single page is needed

**Do not use when:**
- Exa returns sufficient results for a single-page lookup (Exa is cheaper for clean reads)
- The target requires authentication not configured in this environment

---
