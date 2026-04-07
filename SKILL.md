---
name: handoff
description: Session handoff skill for kaizen students — saves progress, updates logs, commits code, and generates a handoff prompt so the next session picks up seamlessly. Drop this into any project's .claude/skills/ folder. Written & Orchestrated by Ned Thomas.
---

# Handoff

Cleanly wrap up the current session so the next one can pick up seamlessly. Works across devices.

**Every step is mandatory. No skipping. Auto-create any files that don't exist yet.**

## When to Use

- Context window is getting heavy (long conversation, lots of tool calls)
- You're wrapping up for the day
- You've finished a major piece of work
- You're about to switch to a different project
- Invoke manually with `/handoff`

## Workflow

```
1. Log       → Create/update conversation-log.md (auto-create if missing)
2. Backlog   → Create/update backlog.md with remaining tasks (auto-create if missing)
3. Vault     → Sync to Obsidian vault (only if configured in CLAUDE.md — skip otherwise)
4. Git       → Commit and push (only if this is a git repo — skip otherwise)
5. Handoff   → Generate handoff prompt and explain how to use it
```

---

## Step 1: Update Conversation Log

**If `conversation-log.md` doesn't exist in the project root, create it.**

Append a new session entry — never overwrite previous entries.

```markdown
## Session [N] — [Date]

### What Was Done
- [List all completed work with file references]

### Decisions Made
- [Any architectural, design, or business decisions]

### What Failed & Why
- [Failed approaches and root causes — so the next session doesn't repeat them]

### Blockers
- [Anything unresolved that's blocking progress]

### What's Next
- [Prioritised list of next steps]
```

**How to determine the session number:** Read the existing conversation-log.md, find the highest session number, and increment by 1. If the file is new or empty, start at Session 1.

## Step 2: Capture Remaining Tasks

**If `backlog.md` doesn't exist in the project root, create it.**

Add unfinished work with enough context that a fresh session can pick it up without needing the old conversation.

Format:

```markdown
- [ ] [Task description] — [Why it matters / context needed to do it]
```

If all tasks were completed this session and there's nothing outstanding, still create the file but note that the backlog is clear.

## Step 3: Sync to Obsidian Vault (Conditional)

**Check the user's CLAUDE.md for an Obsidian vault path** (look for `vault_path`, `obsidian`, `second-brain`, or similar).

- **If found:** sync session context to the vault (see below)
- **If not found:** skip this step entirely and move to Step 4

### 3a. Update or create the project note

Find or create a project note at `{vault_path}/Projects/{project-name}.md`:

```markdown
# {Project Name}

## What It Is
[One-liner description]

## Current State
[What's built, what's working, what's deployed]

## Progress Log
### {Date} — {Brief Summary}
- What was done
- Key decisions
- Open issues
```

Append to the Progress Log — never overwrite previous entries.

### 3b. Save task notes (if needed)

For significant unfinished work, create `{vault_path}/Tasks/{task-name}.md`:

```markdown
---
type: task
status: open
project: {project-name}
priority: {high|medium|low}
created: {date}
---

## {Task Name}

[What needs doing and why]

### Context
[Enough detail to pick this up cold]
```

### 3c. Save key learnings (if any)

Non-obvious insights or failed approaches worth remembering — create a note in `{vault_path}/Learnings/` or similar.

## Step 4: Commit & Push (Conditional)

**First, check if this project is a git repository** (look for a `.git` folder in the project root).

- **If NOT a git repo:** skip this step entirely and move to Step 5
- **If it IS a git repo:** commit and push using the steps below

```bash
git fetch origin && git status
# Check for divergence — pull first if needed
git add {specific files}  # Never git add -A (may include secrets)
git commit -m "Session handoff: [brief description of work done]"
git push origin main
```

**Safety rules:**
- Always `git fetch` before committing to check for remote changes
- If local and remote have diverged, pull first (use `--no-rebase` to avoid conflicts)
- Stage specific files by name — never use `git add -A` or `git add .` (risk of committing .env files, secrets, or temp files)
- If push fails, pull and retry — never force push
- If there's no remote configured (local-only repo), just commit — skip the push

## Step 5: Generate Handoff Prompt

Output a complete handoff prompt that the user can copy:

```
## Handoff — [Project Name] — [Date]

### Context
[What we were working on and why]

### Completed
- [Done items with file references where helpful]

### In Progress
- [Partially done items with current state]

### Failed Approaches
- [What didn't work and why — so the next session doesn't repeat it]

### Next Steps
1. [Prioritised action items]
2. [Include exact commands or file paths where helpful]

### Key Files
- [List of files that were created or modified this session]

### Verification
- [How to verify the work done this session — e.g. "run npm test", "check localhost:3000"]
```

Then tell the user:

**"Copy the handoff prompt above. When you start your next Claude Code session, paste it in as your first message. Claude will read your CLAUDE.md automatically AND have the handoff context — so it knows both your project setup and exactly where you left off. You won't need to re-explain anything."**

---

## Important Rules

- **Auto-create missing files** — conversation-log.md and backlog.md should be created automatically if they don't exist. Never ask the user to create them manually.
- **Never skip the conversation log** — this is how context survives between sessions
- **Include failed approaches** — the biggest time waste is repeating work that already didn't work
- **Append, don't overwrite** — conversation logs and project notes should accumulate over time, building a history of the project
- **Gracefully skip what doesn't apply** — no git? Skip the commit. No Obsidian? Skip the vault sync. The skill should work for everyone regardless of their setup.

## Quick Start — One-Command Install

Paste this into any Claude Code session and the entire handoff system sets itself up:

```
Set up my project for session handoffs. Do all of this automatically without asking me questions:

1. Create .claude/skills/handoff/SKILL.md with the Kaizen handoff skill (fetch it from https://raw.githubusercontent.com/itsnedthomas/kaizen-handoff/main/SKILL.md)
2. If CLAUDE.md doesn't exist in the project root, create a starter one with sections for: About Me, What This Project Does, Tech Stack, Current Status, and a Conversation Log section at the bottom
3. Confirm what was created and tell me to type /handoff at the end of any session to save my progress
```

That's it. One paste, everything gets built. Then just type `/handoff` whenever you're done working.
