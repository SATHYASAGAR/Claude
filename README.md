# Claude

A collection of Claude Code skills.

## Skills

### mock-interview

Simulates a FAANG-style technical coding interview. Paste any coding problem and say "interview me on this" — Claude will act as the interviewer, probe your thinking, give only Socratic hints, and deliver a structured pass/fail scorecard at the end.

**Evaluated dimensions:**
1. Clarify — Did you ask about edge cases and constraints before diving in?
2. Approach + data structure — Did you explain your algorithm and justify your data structure choice?
3. Align before coding — Did you get confirmation before writing code?
4. Narrate while coding — Did you explain your logic as you wrote?
5. Test cases — Did you walk through examples including edge cases?

**Verdict:** PASS / BORDERLINE / FAIL with a 1–15 score.

## Installation

1. Copy the skill folder into your Claude skills directory:
   `~/Library/Application Support/Claude/local-agent-mode-sessions/skills-plugin/<id>/<id>/skills/`
2. Add an entry to `manifest.json` in that directory (see the skill's SKILL.md for the description text)

To find your skills directory:
```
find ~/Library -name "manifest.json" -path "*/skills-plugin/*" 2>/dev/null
```
