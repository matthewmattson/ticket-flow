# ticket-flow

A small workflow for building software with Claude Code, one ticket at a time. Ideas, specs and tickets live in your own Notion workspace; the code lives in your repo.

| Command | What it does |
|---|---|
| `/ticket-flow:setup` | Connects the project to Notion. Creates or links the databases and writes the config file. Run once per project. |
| `/ticket-flow:spec` | Interviews you about a feature idea, writes a one-page spec on its Notion page, and slices it into tickets. |
| `/ticket-flow:ticket` | Captures a bug, feature or chore you noticed mid-session as a ticket, then gets back to work. |
| `/ticket-flow:start` | Picks up one ticket in a fresh session: clarify, branch, plan, build, verify. |
| `/ticket-flow:ship` | Independent review, pull request, Notion link, and Done after merge. |

Every command runs only when you type it. Claude never starts one on its own.

## How the pieces fit

```
idea ──/spec──▶ spec + tickets ──/start──▶ branch + code ──/ship──▶ PR ──▶ Done
                     ▲
       /ticket ──────┘  (anything you notice along the way)
```

Each ticket is written so a fresh session can implement it with no memory of the conversation that created it: a goal, where to start, what is in and out of scope, and up to five "Done when" checks.

## Requirements

- [Claude Code](https://claude.com/claude-code) (CLI or desktop app)
- A Notion workspace you can create databases in
- A git repository; `/ticket-flow:ship` also needs the [GitHub CLI](https://cli.github.com) (`gh`), signed in

## Install

In Claude Code:

```
/plugin marketplace add matthewmattson/ticket-flow
/plugin install ticket-flow@ticket-flow
```

Or from a terminal:

```bash
claude plugin marketplace add matthewmattson/ticket-flow
claude plugin install ticket-flow@ticket-flow
```

## Connect Notion

The plugin talks to Notion through Notion's own MCP server, signed in as you. It never sees a token or password.

1. Connect Notion if you have not already:
   - CLI: `claude mcp add --transport http notion https://mcp.notion.com/mcp`, then run `/mcp` and sign in.
   - Desktop or web app: enable the Notion connector in Settings.
2. In your project, run `/ticket-flow:setup`. It asks for:
   - a ticket prefix (for example `APP`, giving `APP-1`, `APP-2`)
   - whether to create new databases or link ones you already have
   - whether you use sprints
   - your base branch, test command and rules file (`CLAUDE.md` by default)

Setup writes `.claude/ticket-flow.json`:

```json
{
  "prefix": "APP",
  "notion": {
    "tickets": "collection://<id>",
    "ideas": "collection://<id>",
    "sprints": null
  },
  "baseBranch": "main",
  "testCommand": "npm test",
  "rulesFile": "CLAUDE.md",
  "decisionLog": null
}
```

The file holds database IDs only, no credentials. Commit it to share the same databases with teammates, or add it to `.gitignore`.

## The Notion databases

Setup creates these for you, or checks an existing database against them.

**Tickets**: `Name`, `ID` (unique ID with your prefix), `Status` (Triage, Not started, In progress, Done, Won't do), `Type` (Feature, Bug, Chore), `Priority` (P1 blocker, P2 this sprint, P3 nice to have), `Feature idea` (relation), `Sprint` (relation, optional), `PR` (url)

**Feature ideas**: `Idea`, `Stage` (Idea, Spec'd, Building, Shipped), `Tickets` (relation)

**Sprints** (optional): `Name`, `Sprint status` (Past, Current, Next), `Dates`

## Conventions the skills follow

- Branches are named `<prefix>-<n>-<short-title>`, and `/ticket-flow:ship` reads the ticket number from the branch.
- Pull requests are titled `<PREFIX>-<n>: <ticket title>` and include `Fixes <PREFIX>-<n>`.
- Nothing is merged, and no code is edited after planning, without your explicit yes.
- Only `/ticket-flow:ship` sets a ticket to Done.

## What the plugin runs and sends

The plugin is five Markdown skill files. It contains no scripts, hooks, binaries or bundled MCP servers, and it sends nothing to its author or to any third party.

When you run a command, Claude acts with your own tools and sign-ins:

- **Notion**, through the Notion MCP server you connected: searches, reads, creates and updates pages in the databases named in `.claude/ticket-flow.json`. Ticket and spec text you approve is written to your workspace.
- **git**, in your repo: status, log, diff, branch, commit, and push when you ship.
- **GitHub CLI** (`gh`): views, creates and, only on your explicit yes, merges pull requests.
- **Your test command**, as set in the config file.

The only file it writes outside your normal code changes is `.claude/ticket-flow.json`.

## Customising

The skills are plain Markdown in `skills/<name>/SKILL.md`. Fork the repo and edit the interview topics, ticket template or review checklist to suit your stack.

## License

MIT
