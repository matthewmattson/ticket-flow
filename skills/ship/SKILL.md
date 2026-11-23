---
name: ship
description: Close out the current ticket with an independent review, a PR that references the ticket, the PR link in Notion, and Done after merge. Only runs when the user types /ticket-flow:ship.
argument-hint: "[optional: ticket ID, e.g. TKT-42]"
disable-model-invocation: true
---

# /ship: review, PR, merge, close the ticket

Only run when the user types this command. Safe to run more than once: each run picks up wherever the ticket is (not yet reviewed, PR open, or PR merged).

Ticket: $ARGUMENTS, or the number in the branch name (`<prefix>-42-...` means `<PREFIX>-42`).

Requires the GitHub CLI (`gh`), signed in.

## Config

Read `.claude/ticket-flow.json` at the project root. If it is missing, tell the user to run `/ticket-flow:setup` and stop. It gives you:

- `prefix`: the ticket prefix, written `<PREFIX>` below (lowercase `<prefix>` in branch names)
- `notion.tickets`: the Tickets data source (`Status`, `PR` url, `Feature idea`)
- `notion.ideas`: the Feature ideas data source (`Stage`, `Tickets`)
- `baseBranch`, written `<base>` below
- `testCommand`, `rulesFile`, `decisionLog`: may be empty

## Steps

1. **Get context.** Run `git branch --show-current`, `git status --short`, `git log --oneline <base>..HEAD`, `git diff --stat <base>...HEAD` and `gh pr view --json number,state,url` (no PR is fine).
2. **Find the ticket** and fetch it. If there is no ticket, stop and ask.
3. **Skip ahead if possible.**
   - PR state MERGED: go to step 9.
   - PR state OPEN: report its URL and go to step 8.
4. **Preconditions.** If there are uncommitted changes, show them and ask whether to commit them as `<PREFIX>-<n>: ...`. Run `testCommand` (or the test command in the rules file). If anything fails, stop and report; do not ship red.
5. **Independent review.** Start a subagent that has not seen this session. Give it only:
   - the ticket's Goal, Scope and Done when sections, and its Changes section if it has one
   - the output of `git diff <base>...HEAD`
   - the path to the rules file

   Instruct it to stay read-only, skip style, and return a short list split into **Blocking** and **Non-blocking**, checking:
   - each Done when line: met, with the evidence in the diff or tests, or not met
   - changes outside Scope
   - amendments in the Changes section: each has a reason, and none simply loosens a Done when line to match what was built. An amendment with no reason is Blocking.
   - rules file breaks (size limits, layering, conventions)
   - missing tests for new logic or for the bug fix
   - risky changes: schema or data migrations, permissions, anything that could lose user data
6. **Act on the review.** Show the user the findings. Fix blocking items only with their OK, then re-run the tests. Non-blocking items: list them and offer `/ticket-flow:ticket` for any worth keeping.
7. **Open the PR.**
   - Tick the Done when checkboxes in the Notion ticket that are now verified.
   - `git push -u origin HEAD`
   - `gh pr create --title "<PREFIX>-<n>: <ticket title>"` with a body containing:
     - `Fixes <PREFIX>-<n>` and the Notion ticket link
     - 2-4 bullets on what changed
     - the Done when checklist with results
     - Changes to the ticket made during the build, if any, with their reasons
     - Verification: the exact commands run and their results; screenshot paths
     - Follow-ups: tickets created or suggested
   - Put the PR URL in the ticket's `PR` property. Status stays In progress.
8. **Merge, only on the user's word.** Ask: merge now (`gh pr merge --squash --delete-branch`), or later in GitHub? Never merge without an explicit yes. If later, stop here; running this command again after the merge finishes the job.
9. **Close out after the merge.** Confirm with `gh pr view` that the state is MERGED, then:
   - Set the ticket's Status to Done.
   - `git switch <base> && git pull --ff-only`, then delete the local branch if it still exists.
   - If the ticket links a Feature idea and all of that idea's tickets are now Done or Won't do, ask the user whether to set its Stage to Shipped.
   - If this work made an architecture or data decision and the project keeps a decision log, remind the user to add it.
10. **Report in one line:** `<PREFIX>-<n> shipped: <PR URL>`.

## Rules

- This skill is the only thing that sets a ticket to Done.
- Never force-push, never merge without the user's explicit yes, never edit other tickets.
