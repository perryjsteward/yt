---
name: pipeline
description: Yellowtail Club — The round's pipeline, in one command - create (set up the round as a pipeline: target, currency, instrument, stages), add (firms onto it as deals, with a primary contact), update (move a stage, set value, owner, close date, next step, pass or win), log (a call, email, meeting or intro on a deal), deal (a deep dive on one deal's health and risk), review (the whole pipeline's health against target) and hygiene (missing contacts, next steps, duplicates, fixed on approval). Use when someone says "set up the round", "create a pipeline", "add X to the round", "put these on the pipeline", "move Northwind to diligence", "mark Brightwater as passed", "log a call with", "review the Northwind deal", "what's blocking this deal", "pipeline review", "how's the round going", "what's stuck", "clean up the CRM" or "which deals have no contact".
argument-hint: "[create | add | update | log | deal | review | hygiene] [the action's own arguments]"
---

# /pipeline

The round as a pipeline of deals: set it up, put firms on it, change a deal, log what happened, review one deal or the whole pipeline, and clean up its data. One command, with an action for each job; each action's procedure is a reference in this
folder, read only when that action runs.

| Action | What it does | Arguments | Reference |
| --- | --- | --- | --- |
| **`create`** | Set up a fundraising round in Yellowtail as a pipeline on the project - its name ("Seed 2026"), description, target in the round's own currency, status (planning, raising, closed), round details (round type, instrument such as SAFE or priced equity), its dates as milestones on the round's timeline (launch, lead term sheet, lead committed, legal close) and its stages, either Yellowtail's defaults or a fundraising set (Researching, Contacted, First meeting, Partner meeting, Due diligence, Term sheet, Committed, Passed) | `[round name] [target] [project name]` | `references/create.md` |
| **`add`** | Put investor firms on a Yellowtail pipeline as deals, one deal per firm, in the first stage or a named one, with the primary contact chosen from Yellowtail's people at that firm when the user names none, and the choice explained | `[pipeline name] <firms… \| list name> [stage]` | `references/add.md` |
| **`update`** | Change a deal on a Yellowtail pipeline without opening the board | `<firm or deal> <what changed> [why]` | `references/update.md` |
| **`log`** | Record a call, email, meeting or intro on a Yellowtail deal as a comment in its activity log, written up from what the user describes (or a pasted email or note), dated when it happened, and optionally moving the deal's stage and setting its next step at the same time | `<firm or deal> <what happened> [when] [new stage]` | `references/log.md` |
| **`deal`** | Deep dive on one deal on a Yellowtail pipeline (an investor on a round, or a prospective client on an advisor's Clients pipeline) - its health (fit, how recently they engaged, how long it has sat in its stage, how many people we know there, whether there is a champion), whether the stage it sits in matches the evidence, what is missing to reach the next stage and to close, the risks with mitigations, the full stage history from Yellowtail's event log, and dated next actions | `<firm or deal> [--to next-stage \| close]` | `references/deal.md` |
| **`review`** | Check the health of a round (or any Yellowtail pipeline, including an advisor's own client pipeline) — deal count and value per stage, deals stuck with no update or comment in N days, deals with no primary contact or no next step, soft versus hard commits against the target, concentration on one or two firms, and the actions that would move the round this week | `[pipeline name] [--stale 14] [--include-closed]` | `references/review.md` |
| **`hygiene`** | Data-health check for one Yellowtail project — deals with no primary contact, no or overdue next step, stuck, or no value; primary contacts missing email, a verified email, LinkedIn, X, phone or location; people with no firm; likely duplicates; deals on firms outside the project's interests; and plan vs record (overdue milestones, deals closing after legal close, a raising round with no timeline) | `[pipeline name \| --all-pipelines] [--stale 14] [--fix]` | `references/hygiene.md` |

## Pick the action

1. The first word of the arguments names it (`/pipeline create …`); the rest are that action's own
   arguments.
2. No action named: match the request to the phrases below and take the closest.
3. Still unclear between two: ask one short question naming both. Never guess an action that writes.

- **`create`** — Use when someone says "set up the round", "create a pipeline", "start the seed round", "new raise for [client]", "change the stages", or after /setup project or /setup user.
- **`add`** — Use when the user says "add X to the round", "put these on the pipeline", "start deals for these funds", "move my Tier 1 list onto Seed 2026", "import the list into the pipeline", or picks firms from a shortlist to pursue.
- **`update`** — Use when the user says "move Northwind to diligence", "mark Brightwater as passed", "we lost Halcyon", "set the Lantern ticket to €500k", "Sarah owns this one now", "push the close to December", "next step is the data room by Friday", "update the deal", or "Tom is the main contact now".
- **`log`** — Use when the user says "log a call with Northwind", "note that I emailed Sarah the deck", "record that Anna intro'd us to Halcyon", "add to the Brightwater deal that…", "track this conversation", or "what's logged on Northwind?".
- **`deal`** — Use when the user says "review the Northwind deal", "deal review", "is Brightwater going to close", "what's the risk on Lantern", "what do we need to get Ember to term sheet", "what's blocking this deal", or "deal health".
- **`review`** — Use when someone says "pipeline review", "how's the round going", "round health", "what's stuck", "where are we against target", "who haven't we followed up", or at a weekly round check-in.
- **`hygiene`** — Use when someone says "clean up the CRM", "CRM hygiene", "data health", "what's missing on my deals", "audit the pipeline data", "find duplicates", "which deals have no contact", or before a round review or a hand-over.

Then read `references/<action>.md` and follow it exactly: its tools, steps, rules, output and next
steps are the action's own. Read `${CLAUDE_PLUGIN_ROOT}/references/house-rules.md` first, as every
action says.

## Output

Display: inline — unless the action's reference says otherwise (some actions are a dashboard or a document): its own `## Output` section is the display for that action (`${CLAUDE_PLUGIN_ROOT}/references/display.md` §2).
