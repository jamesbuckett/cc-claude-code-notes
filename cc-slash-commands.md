# Claude Code Slash Commands

## /doctor
* Diagnose and verify your Claude Code installation and settings

## /insights
* Generate a report analyzing your Claude Code sessions

## /goal
* [Claude Code /goal](https://www.buildgreatproducts.com/guides/claude-code-goal)
* /goal attaches a completion condition to the current session. 
* After each turn, the condition and the conversation so far are handed to a small fast model (Haiku by default)
  * That returns a yes-or-no decision and a short reason.
* A "no" tells Claude to keep going, with the reason as guidance. 
* A "yes" clears the goal and records an achieved entry in the transcript.

## /memory
* Use /memory to view and manage Claude memory

## Resume Work 
* Quick continue: `claude --continue` picks up exactly where you left off.
* Manual resume: `claude --resume` opens an interactive menu allowing you to select a specific past session based on start time, summary, or initial prompt.
  * Name your conversations with /rename to find them easily in /resume later

