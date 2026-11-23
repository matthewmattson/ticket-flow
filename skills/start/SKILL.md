---
name: start
description: Start work on a ticket from the Notion Tickets database, clarify, branch, plan, build, verify. Only runs when the user types /ticket-flow:start.
argument-hint: "[ticket ID, e.g. TKT-42 or 42]"
disable-model-invocation: true
---

# /start <ticket>: pick up a ticket in a fresh session

Only run when the user types this command. This session has no memory of where the ticket came from. The ticket, its linked spec and the repo are the whole picture. Work one ticket only.

Ticket: $ARGUMENTS (`<PREFIX>-42` or just `42`). If none was given, ask for one and stop.

## Config

Read `.claude/ticket-flow.json` at the project root. If it is missing, tell the user to run `/ticket-flow:setup` and stop. It gives you:

- `prefix`: the ticket prefix, written `<PREFIX>` below (lowercase `<prefix>` in branch names)
- `notion.tickets`: the Tickets data source (`ID` is the ticket number; `Status`, `Sprint`, `Feature idea`, `Type`, `PR`)
- `notion.sprints`: the Sprints data source, or null if sprints are not used (current one: `Sprint status` = Current)
- `notion.ideas`: the Feature ideas data source (`Stage` = Idea / Spec'd / Building / Shipped)
- `baseBranch`, written `<base>` below
- `testCommand`, `rulesFile`, `decisionLog`: may be empty

## Steps

1. **Get context.** Run `git branch --show-current`, `git status --short` and `git log --oneline -5`.
2. **Load the ticket.** Query the tickets data source for `ID` = the number and fetch the page.
   - Status Done or Won't do: say so and stop.
   - Status Triage: note it was never triaged and ask whether to go ahead.
3. **Load the spec, if any.** If `Feature idea` is set, fetch that page and read its Spec section. Use it for intent and edge cases; the ticket's Scope and Done when still define this session's work.
4. **Check git.** If the tree has uncommitted changes, stop and ask whether to commit or stash them. Switch to `<base>` and run `git pull --ff-only` if a remote exists.
5. **Close the gaps before coding.**
   - Ask the user every `[NEEDS CLARIFICATION]` item (concrete options, 4 questions max). Write each answer back into the ticket under Open questions as "Resolved: ...".
   - If Done when or Verify is missing or vague, draft them and confirm with the user before going on.
   - If the work clearly exceeds one session (more than about 3 hours, more than 5 Done when lines, or several unrelated areas), propose a split and stop. The user can create the extra tickets with `/ticket-flow:ticket`.
6. **Update Notion.**
   - Status = In progress.
   - If sprints are used and `Sprint` is empty, set it to the Current sprint so the board stays accurate.
   - If the linked Feature idea has Stage = Spec'd, set it to Building.
   - If two or more other tickets are already In progress, mention it once. Change nothing else.
7. **Branch.** `git switch -c <prefix>-<n>-<slug>`, with a slug of 3-5 kebab-case words from the title.
8. **Explore, narrowly.** Read the Start here files, the rules file or decision log entries the ticket names, and only what those lead to. Do not survey the whole codebase.
9. **Plan and wait.** Present a short plan: files to change, tests to add, risks such as data migrations, permissions or data loss. Wait for the user's approval before editing any code.
10. **Build.**
   - Type = Bug: write a test that reproduces the bug, run it, confirm it fails for the right reason, then fix.
   - Feature: add tests for new logic as you go.
   - Commit in small steps with messages starting `<PREFIX>-<n>: `, unless the rules file sets a different commit format; then follow it and put the ticket ID in the message.
   - Stay inside Scope. Anything new you notice is a separate ticket: tell the user and suggest `/ticket-flow:ticket`. Do not build it.
11. **Verify.** Run the ticket's Verify steps and `testCommand` (or the test command in the rules file). For UI changes, run the app if you can and check the changed screens, including any themes and text sizes the project supports. Fix and re-run until everything passes.
12. **Hand off.** Report each Done when line as met or not met, with the evidence (command output, screenshot path). End with: "Ready for /ticket-flow:ship." Do not push, open a PR, or mark the ticket Done.
