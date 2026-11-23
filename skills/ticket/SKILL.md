---
name: ticket
description: Capture a feature, bug or chore found during this session as a ticket in the Notion Tickets database, in Triage. Only runs when the user types /ticket-flow:ticket.
argument-hint: "[what you noticed, e.g. crash saving a draft with no title] [optional: sprint]"
disable-model-invocation: true
allowed-tools: Bash(git branch --show-current), Bash(git rev-parse *), Bash(git status *), Bash(git log *)
---

# /ticket: capture work without leaving the session

Only run when the user types this command. You are capturing a ticket so a later, fresh session can implement it with no access to this conversation. Capture what this session knows that the repo does not. Then go back to what we were doing. Do not start implementing the ticket.

Request: $ARGUMENTS

If the request includes "sprint" or "now", the ticket goes into the current sprint (see step 6).

## Config

Read `.claude/ticket-flow.json` at the project root. If it is missing, tell the user to run `/ticket-flow:setup` and stop. It gives you:

- `prefix`: the ticket prefix, written `<PREFIX>` below (tickets are `<PREFIX>-n`)
- `notion.tickets`: the Tickets data source. Properties: `Name` (title), `Status`, `Type` (Feature / Bug / Chore), `Priority` (P1 blocker / P2 this sprint / P3 nice to have), `Sprint` (relation, one page), `Feature idea` (relation), `PR` (url). `ID` is automatic; never set it.
- `notion.sprints`: the Sprints data source, or null if sprints are not used (the current sprint has `Sprint status` = Current)
- `notion.ideas`: the Feature ideas data source (title `Idea`, `Stage` = Idea / Spec'd / Building / Shipped, `Tickets` relation). New ideas get Stage = Idea.
- `rulesFile` and `decisionLog`: the project's conventions and decision log, if any

## Steps

1. **Get context.** If a repo is available, run `git branch --show-current`, `git rev-parse --short HEAD` and `git status --short`, and note today's date.
2. **Work out what to capture.** Use the request. If it is empty, use the most recent bug, gap or idea from this conversation. If that is unclear, ask one short question and stop.
3. **Is it a task?** A ticket is one concrete change that one session can finish (about 3 hours or less).
   - A broad area or a "someday" idea with no concrete change: say so and offer to add it to Feature ideas instead, with Stage = Idea.
   - A new feature that needs shaping first: suggest `/ticket-flow:spec`.
   - Too big but already clear: propose 2-3 smaller tickets and ask which to create.
4. **Check for duplicates.** Search the tickets data source for the main keywords. If an open ticket (not Done or Won't do) already covers it, show its ID and ask whether to skip or still create.
5. **Pull evidence from this session.** Exact error or console text, file:line, steps you actually observed, the test or command that failed. Never invent reproduction steps. Anything you could not determine goes under Open questions.
6. **Choose properties.**
   - Type: Bug if something works wrong today; Feature if it adds or changes behavior; Chore for releases, config, refactors, tooling.
   - Priority: P1 only if it blocks a release or loses user data. Default P3 unless the user says otherwise.
   - Status: always `Triage`.
   - Sprint: leave empty. Only if the user said "sprint" or "now" and sprints are used, look up the sprint with `Sprint status` = Current, set `Sprint` to it, and set Status to `Not started` instead of Triage.
   - Feature idea: set only if one clearly matches; otherwise leave empty.
7. **Preview, then create.** Show the user the title, Type, Priority, the Goal line and the Done when list. Ask once: create, edit, or cancel. On create, make the page with the tickets data source as parent and write the body yourself in the format below. Do not apply a database template; template content loads asynchronously.
8. **Report in one line:** `<PREFIX>-n: <title>` plus the Notion link. Fetch the new page if you need its ID. Then continue the earlier task.

## Title

An imperative verb plus an object, under 60 characters, plain text. Examples: "Fix crash saving draft with no title", "Add CSV export to reports page", "Migrate Order table to schema v4".

## Body format

Use these sections in this order. Drop any section with nothing real to say. Never write "N/A" or leave placeholder text.

```markdown
## Goal
1-3 plain sentences: what changes for the user and why. Paste feedback, crash or error text verbatim.

## Context
- Captured from: <branch> @ <commit>, <file:line>, <date>
- Start here: 1-3 pointers, e.g. @src/features/drafts/DraftEditor.tsx or the existing pattern to copy
- Rules that apply: rules file section or decision log ID. Link or name it; never paste the rule text.
- Assets: screenshot, design, docs link

## Scope
- In: what must be included (empty state, error state, accessibility labels)
- Out: what this ticket will not do

## Done when
- [ ] Observable outcome a user or a test can see
- [ ] Given <state>, when <action>, then <result> (only for tricky state such as offline, permission denied, first run)

## Verify
- Tests: the exact test command or a narrower test target, plus any new test to add
- Manual: short walkthrough; what screenshot or output to attach

## Open questions
- [NEEDS CLARIFICATION] anything this session could not decide

## Notes
Approach hunch, rabbit holes, related ticket IDs.
```

For Type = Bug, add this after Notes:

```markdown
---
### Bug details
- Start state: fresh install / signed in / offline / permission denied
- Steps:
  1. ...
- Actual: what happens, with exact error text
- Expected:
- Env: app version or commit, OS or browser, device
- Evidence: crash log, stack trace, recording
- Suspected cause: file:line, if known
- Rule: write a failing test that reproduces this first, then fix it.
```

## Rules

- Five Done when lines at most. More means the ticket should be split.
- Never describe the architecture or file layout; the rules file covers that.
- Never change any other ticket's status.
- Keep the whole ticket short enough to read in under a minute.
