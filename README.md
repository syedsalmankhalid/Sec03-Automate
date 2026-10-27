# Sec03 – Automating Tasks with Claude Code

A reference for the ways you can make Claude Code do work for you: on a schedule, on its own, or with extra tools plugged in.

## Contents

1. [Scheduled tasks](#1-scheduled-tasks)
2. [Loops (`/loop`)](#2-loops-loop)
3. [Cloud routines (`/schedule`)](#3-cloud-routines-schedule)
4. [Hooks](#4-hooks)
5. [Skills](#5-skills)
6. [Slash commands](#6-slash-commands)
7. [Subagents](#7-subagents)
8. [Connectors (MCP servers)](#8-connectors-mcp-servers)
9. [Plugins](#9-plugins)
10. [Headless mode and CI](#10-headless-mode-and-ci)
11. [Which one should I use?](#11-which-one-should-i-use)

---

## 1. Scheduled tasks

These run a saved prompt on a schedule in the Claude desktop app.

- Each task is stored as a `SKILL.md` file under `~/.claude/scheduled-tasks/<task-name>/`.
- The prompt runs in a fresh session in the folder you chose. Nobody is watching when it runs, so write it to work without any input from you.
- You can manage tasks from the desktop app, or by asking Claude to create, update, list, run, or delete one.

**Example from this course:** `daily-project-review` searches this folder for TODO/FIXME/HACK/XXX/BUG comments and likely bugs, then writes a short report. It doesn't change any files.

**Tips for writing a scheduled prompt**
- Say exactly what Claude may do: read-only, edit files, or send messages.
- Say what the output should look like (for example, "a summary of 15 lines or fewer").
- Say what to report when there's nothing to find.

## 2. Loops (`/loop`)

A loop repeats a prompt or slash command while the current session stays open.

```text
/loop 5m /code-review          # every 5 minutes
/loop check whether the build passed   # no interval: Claude picks the timing
```

Good for: checking on a deploy, a CI run, or a long job until it finishes.

## 3. Cloud routines (`/schedule`)

A cloud routine runs Claude Code in Anthropic's cloud on a cron schedule, so your computer doesn't need to be on.

```text
/schedule every weekday at 9am summarize open PRs
/schedule once at 3pm remind me to check the release
```

Good for: recurring reports, nightly checks, and one-off reminders.

## 4. Hooks

Hooks are shell commands the Claude Code harness runs automatically when certain events happen. They're configured in `settings.json` (`~/.claude/settings.json`, `.claude/settings.json`, or `.claude/settings.local.json`).

| Event | When it runs | Example use |
|---|---|---|
| `PreToolUse` | Before a tool runs | Block edits to protected files |
| `PostToolUse` | After a tool runs | Run the formatter or linter after every edit |
| `UserPromptSubmit` | When you send a prompt | Add context, or block certain prompts |
| `Stop` | When Claude finishes replying | Play a sound or send a notification |
| `SessionStart` | When a session starts | Load project context |
| `Notification` | When Claude needs your attention | Show a desktop alert |

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [{ "type": "command", "command": "npx prettier --write ." }]
      }
    ]
  }
}
```

> To set up a rule like "from now on, whenever X happens, do Y", use a **hook**. Claude's memory won't run anything automatically, so it can't do this for you.

## 5. Skills

A skill is a folder of instructions, with an optional script, that Claude loads when a task matches it. Each one is a `SKILL.md` file with a name and a description at the top.

- Personal skills: `~/.claude/skills/<name>/SKILL.md`
- Project skills: `.claude/skills/<name>/SKILL.md`
- Run one by name with `/<skill-name>`, or let Claude choose it from its description.

Built-in and installed skills you can try:

| Area | Skills |
|---|---|
| Code quality | `code-review`, `simplify`, `security-review`, `init` |
| Automation | `loop`, `schedule`, `update-config`, `fewer-permission-prompts` |
| Documents | `docx`, `pdf`, `pptx`, `xlsx`, `docs`, `google-workspace` |
| Browsing | `built-in-browser`, `chrome-browser`, `computer-use` |
| Research | `deep-research` |
| Learning | `claude-code-quiz` |
| Making your own | `skill-creator`, `plugin-authoring` |

## 6. Slash commands

These are shortcuts you type in the chat box. Custom ones are Markdown files in `.claude/commands/` (project) or `~/.claude/commands/` (personal). The file name becomes the command name.

```markdown
<!-- .claude/commands/standup.md -->
Summarize what changed in this repo since yesterday as a 5-bullet standup update.
```

Run it with `/standup`.

## 7. Subagents

A subagent is a separate Claude instance with its own context and its own set of tools. Use one to hand off a side task without filling up the main conversation.

- Built-in types: `general-purpose`, `Explore` (read-only search), `Plan` (makes plans).
- Custom subagents: `.claude/agents/<name>.md`, with the model, tools, and reasoning effort set at the top of the file.

## 8. Connectors (MCP servers)

Connectors use the **Model Context Protocol (MCP)** to give Claude access to outside apps and data. You add them in the claude.ai / desktop app settings, or with `claude mcp add` in the CLI. Run `/mcp` to see their status and sign in.

Common connectors:

| Connector | What Claude can do with it |
|---|---|
| Google Calendar | List, search, create, and update events, and suggest meeting times |
| Google Drive | Search, read, create, copy, and share files |
| Google Docs | Read and update docs |
| Canva | Create, edit, resize, and export designs |
| Claude Docs | Create shared docs that people can comment on |
| Claude in Chrome | Control your real Chrome browser |
| Built-in browser | Browse and preview dev servers inside the desktop app |
| Data tools (BigQuery, Amplitude, Hex, Atlassian, …) | Query data and work with tickets, once you've signed in |

> Some connectors have to be signed in through `/mcp` before Claude can use them.

## 9. Plugins

A plugin packages skills, slash commands, hooks, subagents, and MCP servers together so you can install them in one step.

- Browse and install them from the plugin catalog. Claude can also search the catalog for you.
- Examples installed here:
  - **anthropic-skills**: document, browser, research, and memory skills
  - **mattpocock-skills**: `tdd`, `diagnosing-bugs`, `code-review`, `prototype`, `grilling`, `wizard`, …
  - **data**: `analyze`, `build-dashboard`, `sql-queries`, `create-viz`, `validate-data`, …
- To build your own plugin, use the `plugin-authoring` skill.

## 10. Headless mode and CI

You can run Claude Code without the interactive interface, from scripts, cron jobs, or CI.

```bash
claude -p "Find all TODO comments and list them" --output-format json
```

- **GitHub Actions:** run Claude on pull requests and issues (for example, `@claude` mentions or automatic reviews).
- **Claude Agent SDK:** build your own agents in Python or TypeScript using the same agent loop Claude Code uses.

## 11. Which one should I use?

| You want to… | Use |
|---|---|
| Run a prompt every day or week on your computer | Scheduled task |
| Run something on a schedule when your computer is off | `/schedule` (cloud routine) |
| Check on something repeatedly during this session | `/loop` |
| Have something happen every time Claude edits a file or finishes | Hook |
| Reuse a set of instructions for a workflow | Skill or slash command |
| Hand off a side task | Subagent |
| Connect Claude to Gmail, Drive, Calendar, Canva, databases | Connector (MCP) |
| Install a ready-made bundle of the above | Plugin |
| Run Claude from a script or CI pipeline | `claude -p` / GitHub Actions / Agent SDK |
