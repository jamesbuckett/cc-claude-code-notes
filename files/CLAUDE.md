# CLAUDE.md

Behavioral guidelines to reduce common LLM coding mistakes. Where a project CLAUDE.md or an explicitly invoked skill conflicts with this file, the more specific instruction wins.

**Tradeoff:** These guidelines bias toward caution over speed. **Trivial** = read-only work, or a small single-file edit (~≤20 lines) that changes no API, schema, dependency, or behavioral contract — skip the plan-and-pause for those. If unsure, treat as non-trivial.

**Scope:** Ask/pause rules apply only when I can respond. In unattended runs (subagents, scheduled agents, pipelines), state your plan and assumptions in output, then proceed — never block on confirmation that cannot arrive.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Plan Before You Code

**Propose a plan. Wait for confirmation. Then build.**

- For anything beyond a trivial edit in an interactive session, use plan mode — present the plan and wait for my approval before touching code, even when you have enough information to act.
- If the request is vague, interview me — surface assumptions, edge cases, and what *not* to build — before you start.
- Move slow to move fast: the cost of a good plan is dwarfed by the cost of debugging the wrong solution.

## 3. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 4. Surgical Changes

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

## 5. Goal-Driven Execution

**Define success criteria. State how you'll verify. Loop until verified.**

Before writing code, state how you'll know it worked — a test, an observable behavior, a file diff, a browser check. If there's no clear way to verify, flag that; it usually means the goal is underspecified. Turn tasks into verifiable goals (e.g., "fix the bug" → "write a failing test that reproduces it, then make it pass").

**End-of-task summary**: a few sentences — what changed, how it was verified, and any known gaps or untested paths. If you didn't run the verification, say so rather than claiming success.

## 6. Work Defaults

- **Default deliverable**: single-file HTML for explainer, architecture, and education topics. Do not split into multiple files unless explicitly requested.
- **Required skills**: for those deliverables, YOU MUST apply both, in order — `skill-style-guide` first (visual chassis: typography, color, layout), then `skill-build-educational-site` (content architecture: hierarchy, pedagogical flow). For non-educational single-file pages (landing, marketing, prototype, mockup), use `skill-style-guide` alone — prefer it over the `frontend-design` skills.
