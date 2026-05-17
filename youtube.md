# How Anthropic Engineers ACTUALLY Prompt Claude Code

**TL;DR:** Anthropic engineers don't write better one-off prompts — they invest in **Claude Skills** (versioned folders of instructions + tools + scripts) that Claude auto-invokes by description, that compose with each other, and that get sharper every session by being edited from chat history. Four rules: (1) prompt skills, not Claude; (2) skills are more than prompts — the *tools* layer is where leverage lives; (3) make skills small and composable, not one mega-skill; (4) treat every session as a chance to update the skill.

## Source

- **Title:** How Anthropic Engineers ACTUALLY Prompt Claude Code
- **Channel:** Austin Marchese
- **Uploaded:** 2026-05-15
- **Duration:** 10:45
- **URL:** https://www.youtube.com/watch?v=qOvc9IUKEIc
- **Transcript source:** YouTube auto-captions, extracted with `yt-dlp --write-auto-subs` and cleaned (~2,400 words)

## The Mental Model

**Three layers of the AI stack (from Anthropic's own diagram):**
- **Layer 1 — Model:** the underlying LLM (Claude).
- **Layer 2 — Agents + prompts:** how most people interact today.
- **Layer 3 — Skills:** the *application layer*. Anthropic builds the phone; you build the apps. This is the layer worth controlling.

**What a Skill actually is:**
- An organised folder of files that packages *composable procedural knowledge* for an agent — a reusable "how to get this task done" that Claude can invoke by name (e.g. `/draft-email`) or pick up automatically from its description.

## The Four Rules

### Rule 1 — Prompt skills, not Claude

- Most work is repetitive; one-off prompts evaporate when the chat closes.
- Shift from writing custom prompts to writing skills you reference. Once a skill is described well, you don't even need to invoke it explicitly — Claude routes to it.

### Rule 2 — Skills are more than prompts (the *tools* layer is the unlock)

A skill has three internal layers:

| Layer | What it is | Why it matters |
|---|---|---|
| 1. Description | The folder's "label" Claude scans on every request | Vague label → Claude misses the skill. Specific label → automatic invocation. |
| 2. Instructions | Step-by-step playbook for the task | This is what most users obsess over. |
| 3. **Tools** | Scripts, API calls, reference files the skill can run | **Where the real leverage lives — and what most users skip.** |

- Anthropic engineer Eric (quoted in the video): people write beautiful prompts but ship "bare-bones" tools with no docs and parameters named `A` and `B` — no engineer could work with that, and neither can the model.
- Example from the video: a custom skill that programmatically checks domain availability — once the tool exists, sub-agents can scan 10,000+ domains in parallel.

### Rule 3 — Build composable skills, not custom skills

- **Wrong:** one giant `/content-creation` skill that does ideas, scripts, and posts.
- **Right:** small, focused skills (`/youtube-idea-research`, `/youtube-script-writer`, `/linkedin-post`) that *call each other*.

**Three concrete payoffs:**
- **Issues are easy to spot** — when a focused skill breaks, you know exactly where.
- **Improvements compound** — fixing one skill upgrades every workflow that uses it; mega-skills duplicate logic and get fixed in one place but stay broken in another.
- **Reuse beats rebuild** — a `check-domain` skill plugs into any future workflow with zero new code.

**Two technical patterns inside Rule 3:**

- **Pattern A — Save scripts inside skills.** Barry (Anthropic): they kept seeing Claude rewrite the same Python script to apply slide styling, so they saved the script inside the skill folder as a tool. Next session, Claude reruns the script instead of regenerating it. *Code is deterministic — same input, same output; AI is not.* Trading tokens for code compute is cheaper, faster, repeatable. **Rule of thumb: if it can be code, make it code** (and let AI write the code for you, once).
- **Pattern B — Control who invokes what.** Two flags most users don't know exist:
  - `user-invocable: false` → hides the skill from the slash menu; only agents can call it. Good for internal agent tooling.
  - `disable-model-invocation: true` → only the human can run it. Good for high-risk actions (sending messages, deploying to prod).

### Rule 4 — Skills get smarter every session

- A prompt dies when the chat closes; a skill *persists* and can be sharpened every time it's used.
- After each run, ask one question: **"Is this a one-time fix, or should this live in the skill forever?"** If forever — update the skill with the rule, example, or edge case.
- Practical loop: feed the chat back to Claude with *"Review the back-and-forth I just had after using this skill. Can we enhance the skill so this is handled automatically or we don't make the same mistake again?"*
- Anthropic's stated goal: **"Claude on day 30 of working with you should be a lot better than Claude on day one."**

## So What — Practical Takeaways

- **Stop accumulating prompts; start accumulating skills.** A prompt library is write-only memory. A skill library compounds.
- **Audit your existing skills for the tools layer.** If a skill is 95% instructions and 5% scripts, you're leaving the leverage on the table.
- **Split mega-skills.** If a single skill has more than one verb in its name, decompose it.
- **Build a habit of post-run skill updates.** The 30-second edit after each session is where the compounding comes from.

## Relevance to Regulated / Platform Work

Two observations worth carrying into a JPMC / regulated-platform context — not from the video, but flagged because they affect how this advice should be applied:

- **`disable-model-invocation`** maps cleanly onto break-glass / SoD controls — any skill that touches prod, sends external messages, or moves money should be human-invoke-only by default.
- **Composable skills + saved scripts** is essentially the "skills as code" analogue of the C4 + CALM split: the skill description is the human-facing contract; the script in the tools layer is the deterministic, auditable execution. Worth thinking about how skill folders get versioned, reviewed, and signed in an enterprise marketplace.
