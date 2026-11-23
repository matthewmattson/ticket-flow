---
name: setup
description: Connect ticket-flow to your Notion workspace. Creates or links the Tickets, Feature ideas and Sprints databases and writes .claude/ticket-flow.json. Only runs when the user types /ticket-flow:setup.
argument-hint: "[optional: ticket prefix, e.g. APP]"
disable-model-invocation: true
---

# /setup: connect this project to Notion

Only run when the user types this command. Run it once per project. It is safe to run again: it reads the existing config and only changes what the user asks to change.

Argument: $ARGUMENTS (an optional ticket prefix).

The other four skills (`spec`, `ticket`, `start`, `ship`) read everything they need from `.claude/ticket-flow.json` at the project root. This skill creates that file.

## Steps

1. **Check Notion is connected.** Look for Notion MCP tools in this session (search, fetch, create pages, create database, query data sources). The server may have any name. If there are none, tell the user to connect Notion first and stop:
   - Claude Code CLI: `claude mcp add --transport http notion https://mcp.notion.com/mcp`, then `/mcp` to sign in.
   - Claude desktop or web app: enable the Notion connector in Settings.

   The user signs in to their own workspace. Never ask for a Notion token or password.
2. **Read the existing config**, if `.claude/ticket-flow.json` exists. Show what is set and ask what to change. If nothing, stop.
3. **Ask the basics** in one round:
   - Ticket prefix: 2-5 capital letters (default `TKT`). Tickets become `TKT-1`, `TKT-2`, and branches `tkt-1-short-title`.
   - Databases: create new ones, or link existing ones?
   - Sprints: use a Sprints database, or skip sprints?
4. **Create or link the databases.**
   - **Create:** ask which Notion page should hold them (search for it by name, confirm the match). Create the databases described under Schema below, as children of that page.
   - **Link:** ask for each database's name or URL, fetch it, and compare its properties with the Schema. List anything missing or named differently and offer to add or rename it. Do not change an existing database without a yes.
   - The `ID` property must be Notion's unique ID type with the chosen prefix. If the tool cannot create or set it, ask the user to add it by hand in Notion (Add property, then ID, then set the prefix) and wait until they confirm.
5. **Ask about the project** in one round, offering what you can detect from the repo as defaults:
   - Base branch (default: the repo's default branch, usually `main`)
   - Test command (for example `npm test`, `pytest`, `swift test`). Empty is allowed.
   - Rules file: the file with the project's conventions (default `CLAUDE.md` if it exists)
   - Decision log: path to an architecture decision log, if the project keeps one
6. **Write `.claude/ticket-flow.json`** in this shape. Use the data source IDs (`collection://...`) that Notion returns, not page URLs.

   ```json
   {
     "prefix": "TKT",
     "notion": {
       "tickets": "collection://<id>",
       "ideas": "collection://<id>",
       "sprints": null
     },
     "baseBranch": "main",
     "testCommand": "",
     "rulesFile": "CLAUDE.md",
     "decisionLog": null
   }
   ```

7. **Ask whether to commit the file.** Committing it lets teammates share the same databases. Otherwise add it to `.gitignore`. The file holds database IDs only, never credentials.
8. **Verify.** Query the tickets data source once to confirm it is readable. Report the three database links and suggest `/ticket-flow:ticket` as a first try.

## Schema

Property names and option names must match exactly; the other skills rely on them.

**Tickets**

| Property | Type | Options |
|---|---|---|
| `Name` | title | |
| `ID` | unique ID | prefix = the ticket prefix |
| `Status` | status or select | Triage, Not started, In progress, Done, Won't do |
| `Type` | select | Feature, Bug, Chore |
| `Priority` | select | P1 blocker, P2 this sprint, P3 nice to have |
| `Feature idea` | relation to Feature ideas | two-way, shown there as `Tickets` |
| `Sprint` | relation to Sprints | only if sprints are used |
| `PR` | url | |

**Feature ideas**

| Property | Type | Options |
|---|---|---|
| `Idea` | title | |
| `Stage` | select | Idea, Spec'd, Building, Shipped |
| `Tickets` | relation to Tickets | the other side of `Feature idea` |

**Sprints** (optional)

| Property | Type | Options |
|---|---|---|
| `Name` | title | |
| `Sprint status` | select | Past, Current, Next |
| `Dates` | date | |

## Rules

- Never delete a database, a property or a page.
- Never store a token, password or other secret in the config file.
