# Privacy

ticket-flow is a set of Markdown instructions that Claude follows on your machine. It has no server, no analytics and no telemetry.

## What the author collects

Nothing. The plugin sends no data to its author or to any service the author runs. There is no account to create and nothing is retained.

## What the plugin does with your data

When you run a command, Claude acts with your own tools and sign-ins:

- **Notion:** reads and writes pages in the databases you chose during setup, through the Notion connection you signed in to. Ticket and spec text, including anything you paste into it, is stored in your Notion workspace.
- **git and GitHub:** reads your repository, creates branches and commits, and, when you ship, pushes the branch and opens a pull request with the GitHub CLI under your account.
- **Local file:** writes `.claude/ticket-flow.json` in your project. It holds Notion database IDs and project settings, never credentials.

That data is handled by Notion, GitHub and Anthropic under their own terms and privacy policies, as with any other use of Claude Code.

## Removing your data

Uninstall the plugin and delete `.claude/ticket-flow.json`. Tickets, specs and pull requests stay in your Notion workspace and GitHub repository until you delete them there.

## Questions

Open an issue at https://github.com/matthewmattson/ticket-flow/issues.
