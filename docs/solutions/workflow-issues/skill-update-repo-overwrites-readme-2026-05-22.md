---
title: skill-update-repo silently overwrites bespoke README sections
date: 2026-05-22
category: workflow-issues
module: skill-update-repo
problem_type: workflow_issue
component: documentation
severity: high
applies_when:
  - "Running /skill-update-repo on a repo with hand-authored README sections"
  - "Confirmation prompt lists target filenames without section-level diffs"
  - "README contains content outside the skill's template placeholders"
  - "Repo has prior commits worth preserving via git show"
tags:
  - skill-update-repo
  - readme
  - overwrite
  - scaffolding
  - confirmation-ux
  - recovery
  - git-show
  - template-substitution
---

# skill-update-repo silently overwrites bespoke README sections

## Context

`/skill-update-repo` regenerates `README.md` from a template by design — its own docs flag this as an overwrite, not a merge. Phase C confirmation lists target file paths (`README.md`, `LICENSE`) but does not diff section-level content, so bespoke H2 sections under the existing README are invisible at the point of approval. In this session, a personal notes repo lost two hand-authored sections ("Prompting Takeaways" and "Planned Skill Ideas") because the user approved on filenames alone. Recovery was only possible because the user had committed before invoking the skill.

## Guidance

Treat `/skill-update-repo` as a destructive rewrite of `README.md`. Three mitigations, in order of preference:

1. **Commit in-flight work first.** Always land a clean commit on the current branch before invoking the skill. This is the only reliable safety net — Phase C will not show you what's being overwritten.
2. **Fold bespoke sections into the template inputs.** When the skill prompts for project description, usage notes, or "about" content, paste your custom sections there so they end up inside the regenerated README rather than being replaced by it. Don't keep ad-hoc H2 sections in `README.md` that the template doesn't know about.
3. **Plan the recovery path before approving.** If sections do get overwritten, recover them from the pre-scaffold commit:

```bash
# Identify the commit immediately before the scaffold
git log --oneline README.md

# Extract the prior README to inspect or salvage sections
git show <prior-sha>:README.md > /tmp/README.prior.md

# Diff to see exactly what the scaffold removed
git diff <prior-sha> HEAD -- README.md

# Re-add the lost sections in a follow-up commit (do not amend the scaffold commit)
```

## Why This Matters

The skill's own documentation accurately warns that it overwrites `README.md`. The gap is in the confirmation UI: Phase C surfaces filenames, not section-level diffs, so a user who skims the prompt cannot see which bespoke content is about to disappear. The failure mode is invisible until after the commit lands, and recovery depends entirely on whether the user happened to commit beforehand. Documenting this closes the loop between "the skill is honest" and "the user can act on that honesty."

## When to Apply

- Before running `/skill-update-repo` on any repo whose `README.md` contains hand-authored sections beyond the standard template (about, install, usage, license).
- When advising others on first-time use of repo-scaffolding skills that regenerate from templates.
- During any skill invocation where the Phase C / pre-write confirmation lists files but not content.
- When designing or reviewing scaffolder skills — consider whether the confirmation step should diff content, not just filenames.

## Examples

**Before scaffold** (`README.md` at SHA `bc89619`):

```markdown
## Planned Skill Ideas

- skill-rip-youtube — transcript + summaries to single-file HTML
- skill-improve-quality — audit explainer pages against style guide
- skill-update-repo — scaffold README/LICENSE/topics in one shot

## Prompting Takeaways

### Mental Model
...

### Four Rules
...

### Two Patterns
...

### So What
...
```

**After scaffold** (SHA `bf53dcd`): both H2 sections gone, replaced by template-substituted About/Usage/License blocks.

**After recovery** (SHAs `eee29cb` and `a24b381`): bespoke sections re-added as follow-up commits, sourced via:

```bash
git show bc89619:README.md > /tmp/README.prior.md
# Copy the "Planned Skill Ideas" and "Prompting Takeaways" sections back into README.md
git add README.md
git commit -m "Re-add Planned Skill Ideas section lost in scaffold"
git commit -m "Re-add Prompting Takeaways section lost in scaffold"
```

The recovery worked only because `bc89619` existed. Had the bespoke sections been uncommitted edits at the time of `/skill-update-repo`, they would be unrecoverable.

## Related

- Plugin: `compound-engineering` — provides the `/ce-compound` workflow that captured this learning
- Skill: `skill-update-repo` — the scaffolder whose template-overwrite behavior this learning describes
