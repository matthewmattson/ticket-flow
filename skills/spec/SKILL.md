---
name: spec
description: Shape a feature idea into a one-page spec on its Notion Feature ideas page by interviewing the user, then slice it into tickets. Only runs when the user types /ticket-flow:spec.
argument-hint: "[idea name or one-sentence idea]"
disable-model-invocation: true
---

# /spec: the layer above the ticket

Only run when the user types this command. Use it when a feature is bigger than one ticket: more than one session of work, a new screen or flow, a new data model, a new external API, or a change to core business logic. Small changes go straight to `/ticket-flow:ticket`. This skill writes no code.

Idea: $ARGUMENTS

## Config

Read `.claude/ticket-flow.json` at the project root. If it is missing, tell the user to run `/ticket-flow:setup` and stop. It gives you:

- `prefix`: the ticket prefix, written `<PREFIX>` below (tickets are `<PREFIX>-n`)
- `notion.ideas`: the Feature ideas data source (title `Idea`, `Stage` = Idea / Spec'd / Building / Shipped, `Tickets` relation)
- `notion.tickets`: the Tickets data source (`Name`, `Status`, `Type` = Feature / Bug / Chore, `Priority` = P1 blocker / P2 this sprint / P3 nice to have, `Feature idea` relation). `ID` is automatic; never set it.
- `rulesFile` and `decisionLog`: the project's conventions and decision log, if any

## Steps

1. **Find or create the idea.** Search Feature ideas for it. If several match, ask which. If none, create it with Stage = Idea. Read the page and any tickets already linked to it.
2. **Ground yourself in the code.** If the repo is available, briefly look at the areas this would touch (a handful of key files) and the decision log, so your questions are specific. Do not ask the user anything the code already answers.
3. **Interview the user.** Ask in rounds of 2-4 questions with concrete options, 3 rounds at most. Skip anything already settled on the page. Cover:
   - **Problem:** who hits it, when, and what evidence (user feedback, their own use, data)
   - **Appetite:** the time budget (half a sprint, one sprint). Scope bends to fit it, not the other way round.
   - **Smallest version:** the least that would solve the problem
   - **Flows and states:** main path; empty, error, offline, first run, permission denied
   - **Data:** models touched, migrations, what gets stored, effect on existing data
   - **Domain rules and safety:** the source behind any domain rule (a standard, a regulation, research), and anything that could harm users or needs a warning
   - **Measurement:** the success metric and the analytics events needed to see it
   - **No-gos:** what this will deliberately not do
4. **Write the spec on the idea page.** Put it at the top under a "Spec" heading and keep the user's existing notes below it under "Original notes". One page at most. Link to rules file or decision log entries; never paste them. Drop empty sections.

   ```markdown
   ## Spec (<date>)
   ### Problem
   ### Appetite
   ### Solution sketch
   Main flow and key screens in words: places, what's on them, where each action leads.
   ### Rules and edge cases
   Given/When/Then for the tricky ones.
   ### Data and architecture
   Models, migrations, decision log entries needed.
   ### Domain rules and safety
   ### Success metric and tracking
   ### Rabbit holes
   Known risks and how the spec avoids them.
   ### No-gos
   ### Open questions
   ```

5. **Slice it.** Propose 3-6 tickets, in build order. Each must:
   - be a vertical slice that is visible to the user or testable on its own
   - fit in one session (about 3 hours) with 5 Done when lines at most
   - make the first slice the thinnest working end-to-end path

   Add Chore tickets for migrations or analytics events if they need their own step. Show the list and wait for the user to approve or edit it.
6. **Create the tickets.** Title: imperative verb plus object, under 60 characters. Set Status = Not started (shaped work skips Triage), Type, Priority, and `Feature idea` = this idea. Leave Sprint empty; sprint planning decides when. Body, dropping any section with nothing real to say:

   ```markdown
   ## Goal
   1-3 plain sentences: what changes for the user and why.

   ## Context
   - Start here: 1-3 file or pattern pointers
   - Rules that apply: link to this spec page, plus any rules file section or decision log ID

   ## Scope
   - In:
   - Out:

   ## Done when
   - [ ] Observable outcome a user or a test can see
   - [ ] Given <state>, when <action>, then <result> (only for tricky state)

   ## Verify
   - Tests: the test command or a narrower test target, plus new tests to add
   - Manual: short walkthrough; what screenshot or output to attach

   ## Open questions
   - [NEEDS CLARIFICATION] anything not yet decided
   ```

7. **Finish.** Set the idea's Stage to Spec'd. Report the idea link and the new ticket IDs in a short list.

## Rules

- If the interview shows the idea is not worth its appetite, say so and stop. Leave Stage = Idea and add a one-line note with the reason.
- Keep the spec to one page.
