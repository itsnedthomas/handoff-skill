---
name: handoff
description: Session handoff skill — saves progress, auto-logs context, and generates a handoff prompt so your next session picks up exactly where you left off. Built by Ned Thomas.
---

# Handoff

## First-Run Setup (automatic — runs once on first /handoff)

On the FIRST invocation of `/handoff`, check if `CLAUDE.md` exists in the project root.

**If it doesn't exist**, create a starter `CLAUDE.md`:

```markdown
# [Project Name] — CLAUDE.md

## About Me
- Name: [Your name]
- Business/Role: [What you do]
- Location: [City/timezone]
- Communication style: [e.g. "Keep it simple, no jargon, be direct"]

## What This Project Does
[One-liner about what you're building]

## Tech Stack
[List the tools, languages, and services you use]

## Current Status
[What's built so far, what stage you're at]

## Obsidian Vault (Optional)
<!-- If you use Obsidian as a second brain, uncomment the line below and set your vault path. The handoff skill will automatically sync your session notes there. -->
<!-- vault_path: ~/Documents/second-brain/ -->
```

Then explain to the user:

---

**Welcome — your handoff system is set up.**

**What this does:** At the end of every session, type `/handoff` and I'll automatically save everything — what we built, what failed, what's next — into files in your project. That way when you start a fresh session, you can pick up exactly where you left off without re-explaining anything.

**What gets created each time you run /handoff:**
- **conversation-log.md** — a running diary of every session (what was done, decisions, failures, next steps). I create this automatically.
- **backlog.md** — unfinished tasks with enough context to pick them up cold. I create this automatically.
- If you use **Obsidian**, it syncs your notes there too (set your vault path in CLAUDE.md to enable this)
- If you use **git**, it commits and pushes your work automatically

**When to use it:**
- End of every work session — just type `/handoff`
- When your conversation is getting long and things are slowing down
- When you've finished a big chunk of work
- Before switching to a different project

**What you do with the output:** I'll give you a handoff prompt at the end. Copy it. Start a new Claude Code session. Paste it as your first message. I'll know exactly what's going on without you explaining anything.

**One more thing:** I've created a `CLAUDE.md` file in your project root. Fill in the blanks when you get a chance — I read this automatically at the start of every session so I always know your project context.

---

After the first-run explanation, proceed with the normal handoff workflow below.

**If `CLAUDE.md` already exists**, skip the setup explanation — go straight to the handoff.

---

## When to Use

- End of every work session
- Conversation is getting long or slowing down
- You've finished a major piece of work
- You're switching to a different project
- Type `/handoff` to trigger

## Handoff Workflow

Every step is mandatory. Auto-create any files that don't exist. No skipping.

```
1. Log       → Create/update conversation-log.md (auto-create if missing)
2. Backlog   → Create/update backlog.md with remaining tasks (auto-create if missing)
3. Vault     → Sync to Obsidian vault (only if vault_path is set in CLAUDE.md)
4. Git       → Commit and push (only if .git exists — skip otherwise)
5. Handoff   → Generate handoff prompt and tell the user what to do with it
```

### Step 1: Update Conversation Log

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

**Session number:** Read the existing conversation-log.md, find the highest session number, increment by 1. New or empty file = Session 1.

### Step 2: Capture Remaining Tasks

**If `backlog.md` doesn't exist in the project root, create it.**

Add unfinished work with enough context to pick it up without the old conversation.

```markdown
- [ ] [Task description] — [Why it matters / context needed to do it]
```

If everything was completed and nothing is outstanding, note that the backlog is clear.

### Step 3: Sync to Obsidian Vault

**Check CLAUDE.md for a `vault_path` setting.**

- **If found:** sync session context to the vault (steps below)
- **If not found:** skip this step entirely

#### 3a. Update or create the project note

Find or create `{vault_path}/Projects/{project-name}.md`:

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

Append to the Progress Log — never overwrite.

#### 3b. Save task notes (if needed)

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

#### 3c. Save key learnings (if any)

Non-obvious insights or failed approaches — create a note in `{vault_path}/Learnings/`.

### Step 4: Commit & Push

**Check if `.git` exists in the project root.**

- **No `.git` folder:** skip this step entirely
- **Git repo exists:** commit and push

```bash
git fetch origin && git status
git add {specific files}
git commit -m "Session handoff: [brief description]"
git push origin main
```

**Safety rules:**
- Always `git fetch` before committing
- If diverged, pull first (`--no-rebase`)
- Stage specific files by name — never `git add -A` or `git add .`
- If push fails, pull and retry — never force push
- No remote configured? Just commit, skip the push

### Step 5: Generate Handoff Prompt

Output this for the user to copy:

```
## Handoff — [Project Name] — [Date]

### Context
[What we were working on and why]

### Completed
- [Done items with file references where helpful]

### In Progress
- [Partially done items with current state]

### Failed Approaches
- [What didn't work and why — prevents repeating mistakes]

### Next Steps
1. [Prioritised action items]
2. [Include exact commands or file paths where helpful]

### Key Files
- [Files created or modified this session]

### Verification
- [How to verify the work — e.g. "run npm test", "check localhost:3000"]
```

Then tell the user:

**"Copy the handoff prompt above. Start a new Claude Code session and paste it as your first message. I'll read your CLAUDE.md automatically AND have the handoff context — so I know your project setup and exactly where you left off. You won't need to re-explain anything."**

---

## Rules

- **Auto-create everything** — conversation-log.md, backlog.md, and CLAUDE.md get created automatically if missing. Never ask the user to make files manually.
- **Never skip the conversation log** — this is how context survives between sessions.
- **Include failed approaches** — the biggest time waste is repeating work that already didn't work.
- **Append, don't overwrite** — logs and notes accumulate over time, building project history.
- **Gracefully skip what doesn't apply** — no git? Skip commit. No Obsidian? Skip vault sync. Works for everyone regardless of setup.
