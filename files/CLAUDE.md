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

## 2. Plan Before You Code

**Propose a plan. Wait for confirmation. Then build.**

- For anything beyond a trivial edit, write the plan first and pause before touching code.
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

Before writing code, state how you'll know it worked — a test, an observable behavior, a file diff, a browser check. If there's no clear way to verify, flag that; it usually means the goal is underspecified.

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

## 6. Work Defaults

- **Default deliverable**: single-file HTML for explainer, architecture, and education topics. Do not split into multiple files unless explicitly requested.
- **Required skills**: always apply both together — `skill-style-guide` provides the visual chassis (typography, color, layout primitives); `skill-build-educational-site` provides the content architecture (information hierarchy, pedagogical flow, section patterns).
- **Order of operations**: load `skill-style-guide` first to establish the chassis, then `skill-build-educational-site` to populate structure within it.

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.
