---
name: create-pipeline
description: Yellowtail Club — Set up a fundraising round in Yellowtail as a pipeline on the project - its name ("Seed 2026"), description, target in the round's own currency, status (planning, raising, closed), round details (round type, instrument such as SAFE or priced equity), its dates as milestones on the round's timeline (launch, lead term sheet, lead committed, legal close) and its stages, either Yellowtail's defaults or a fundraising set (Researching, Contacted, First meeting, Partner meeting, Due diligence, Term sheet, Committed, Passed). Shows one card, creates on approval. Use when someone says "set up the round", "create a pipeline", "start the seed round", "new raise for [client]", "change the stages", or after /create-project or /add-user.
argument-hint: "[round name] [target] [project name]"
---

# Create a pipeline (a round)

A pipeline is one round (or one outbound campaign) inside a project. It owns the stages, which are
the columns of the deal board, and every deal on it is one firm being worked for that round. A
project may have several pipelines (a seed round now, a bridge later), so this command never assumes
there is only one.

Read `${CLAUDE_PLUGIN_ROOT}/references/house-rules.md` first.

## Tools

| Tool | What for | Required? |
| --- | --- | --- |
| `whoami` | The project and its workspace | Yes |
| `get_project` | Project name and description, to prefill the round | Yes |
| `list_pipelines` | `{ projectId }` → existing rounds with stages, to avoid a duplicate | Yes |
| `create_pipeline` | `{ projectId, pipeline: { name, description?, target?, currency?, attributes?, status? } }` | Yes, on approve |
| `get_pipeline` | `{ id }` → the stages the new pipeline got, with ids, in order | Yes |
| `update_stage` | `{ pipelineId, stageId, changes: { name?, color? } }` to rename or recolour. Don't send `position` | For custom stages |
| `create_stage` | `{ pipelineId, stage: { name, color? } }`, added at the end. Don't send `position` | For custom stages |
| `set_stage_order` | `{ pipelineId, order: { ids: [every stage id, left to right] } }`, the whole board in one call | For custom stages |
| `update_pipeline` | `{ id, changes: {...} }`, only the fields sent change. For an existing round | Only then |
| `list_milestones` | `{ pipelineId }` → the round's timeline, on an existing round | Only then |
| `create_milestone` | `{ pipelineId, milestone: { key, dueOn, label?, note? } }`, one per dated milestone on the card | When the card has dates |
| `update_milestone` | `{ id, changes: { dueOn?, label?, note? } }`, a date that moves on an existing round, with the reason as `note` | Only then |

These are in the `pipelines` toolset (`CONNECTORS.md`).

## The fields

| Field | Rule |
| --- | --- |
| `name` | Required, at most 120 characters. Round and year: "Seed 2026", "Series A", "Bridge Q3 2026" |
| `description` | Optional markdown: what is being raised, on what terms, and who should lead. Two to four sentences |
| `target` | Optional whole number in the round's `currency`, no decimals (`2000000`) |
| `currency` | ISO 4217 code (`GBP`, `EUR`, `USD`): the currency the founder is raising in. New deals on the round take it unless they are given another. **Never convert** one currency into another: £2m is `target: 2000000, currency: "GBP"`. (`targetUsd` is deprecated; don't send it) |
| `status` | `planning` (default: building the list), `raising` (outreach has started), `closed` |
| `attributes` | Free-form round details, values string, number, boolean or null. Use these keys: `round` (`pre-seed`, `seed`, `series-a`, `bridge`, …), `instrument` (`SAFE`, `ASA`, `convertible note`, `priced equity`), `minimumCheque`, `valuationCap` (only figures the user gave). Never the target, its currency or any date: those have their own fields and the timeline |

The app does not display `attributes` today; they are kept for the API and for Claude, and are worth
setting because later commands (`/pipeline-review`, `/round-strategy`) read them.

## The round's dates: the timeline

A round's dates are **milestones** on its timeline, never `attributes` and never only prose in a
doc. The timeline is the one source every later command reads (`/close-plan`, `/daily-briefing`,
`/crm-hygiene`), and the app shows it as the Timeline panel on the pipeline page, which founders
who are guests on the project see too.

| `key` | Means |
| --- | --- |
| `launch` | Outreach to investors goes live |
| `outreach_start`, `calls_start` | First emails out; first investor calls |
| `lead_term_sheet` | A lead's term sheet is signed |
| `lead_committed` | The lead is committed (and the round is anchored) |
| `legal_close` | Documents signed and money in: the round is closed |
| `kickoff`, `plan_end`, `extension_notice` | An advisor's engagement dates |

Any other `lower_snake_case` key is a custom milestone and needs a `label`. One per key per round.
"Close by" is never one date: lead term sheet, lead committed and legal close are three, and a
founder who says "close by 31 Dec" is asked which they mean (or the card says which you assumed,
with a note). A date that waits on a decision goes on with `note: "open decision: …"`.

## Stages

`create_pipeline` always creates Yellowtail's **default stages** (a database trigger):

| Position | Default |
| --- | --- |
| 0 | Researching |
| 1 | Contacting |
| 2 | Contacted |
| 3 | In Conversation |
| 4 | Committed |
| 5 | Passed |

Offer the **fundraising set** as the alternative, for a round that will run through partner meetings
and diligence:

Researching · Contacted · First meeting · Partner meeting · Due diligence · Term sheet · Committed · Passed

or any set the user describes. A stage name is at most 60 characters.

**How to build any set.** Work from the stages `get_pipeline` returns, never from the table above.

1. **Keep** every existing stage whose name is already in the target set. It stays as it is.
2. **Rename** spare existing stages to the target names still missing (`update_stage`, name only).
   Stage names are unique per pipeline: if a new name is still held by another stage, rename that one
   away first.
3. **Create** whatever target names are still missing (`create_stage`, name only). They land at the
   end for now.
4. `get_pipeline` for the full list of stage ids, then **`set_stage_order`** once with every id, left
   to right in the target order. It must list every stage of the pipeline exactly once, spare ones
   included (put spares at the end).

Never move a stage with `update_stage`'s `position`: positions are unique and nothing shifts, so a
move onto an occupied position is refused. `set_stage_order` is the only way to reorder.

For the fundraising set, from the defaults:

| Step | Call | Result |
| --- | --- | --- |
| 1 | keep `Researching`, `Contacted`, `Committed`, `Passed` | no call |
| 2 | `update_stage` `Contacting` → `First meeting` | |
| 3 | `update_stage` `In Conversation` → `Partner meeting` | |
| 4 | `create_stage` `Due diligence`, then `create_stage` `Term sheet` | added at the end |
| 5 | `get_pipeline`, then `set_stage_order` | Researching, Contacted, First meeting, Partner meeting, Due diligence, Term sheet, Committed, Passed |

A set with **fewer** stages than the defaults: there is no stage delete over MCP for customers
(deletes are the Yellowtail desk's only, by policy). Order the spares last with `set_stage_order`,
leave their names alone, and tell the user which columns to delete on the board in the app. Don't
rename a stage to "Spare".

Reshape only a pipeline with no deals unless the user asks. On an existing round, a reorder leaves
every deal in its stage, but a rename changes what every deal in that column means: show which deals
sit in each renamed column on the card first.

## Steps

### 1. Orient

`whoami`, then the project (active unless named, house rules §1). `get_project` and
`list_pipelines`. If a pipeline with the same or a near name exists, stop and ask: change that one
(`update_pipeline`, and stages as above) or create another?

### 2. Gather

Take what the user gave. Ask in one message for anything that matters and is missing: round name,
target and its currency, instrument, the dates they have in mind (launch, lead term sheet, lead
committed, legal close), and default or fundraising stages. Offer sensible defaults (name from the
round type and year; `status: planning`) and accept "skip". Never invent a target, cap or date
(house rules §6): every date the user gives becomes a milestone on the card, and a date they don't
give is left off, not guessed. For a full backwards-dated timeline, `/round-strategy` works one out.
If `/create-project` just ran, reuse its likely round.

### 3. Card

```
1. New pipeline in Kestrel Robotics (Northbound Advisory)
   Name:        Seed 2026
   Description: £2m seed on a SAFE to extend runway to Series A; looking for a UK lead
                with robotics or energy experience.
   Target:      £2,000,000 (target 2000000 · currency GBP; new deals start in GBP)
   Status:      planning
   Attributes:  round seed · instrument SAFE
   Timeline:    launch            2026-11-03
                lead_term_sheet   2027-01-15
                lead_committed    2027-01-29
                legal_close       2027-02-26  note "open decision: you said 'close by end of
                                  February'; read as money in, not lead committed"
   Stages:      fundraising set (8): Researching → Contacted → First meeting → Partner meeting
                → Due diligence → Term sheet → Committed → Passed
Reply: approve · 1: default stages · 1: no timeline · 1: <feedback>
```

### 4. Create

On approve:

1. `create_pipeline` with `projectId` and `pipeline`. Keep the returned `id`.
2. `get_pipeline` to read the stage ids and positions that were actually created. Don't assume the
   defaults above; work from what comes back.
3. If custom stages were approved: keep, rename, create, then `get_pipeline` and one
   `set_stage_order` (above). If a call fails, stop, report which stages are done and which are not,
   and show the board as it now stands.
4. `get_pipeline` again and confirm the stage list matches the card.
5. **Timeline.** `create_milestone` for each approved milestone (`{ pipelineId, milestone: { key,
   dueOn, label?, note? } }`), then `list_milestones` to confirm. On an existing round, a date that
   moves is `update_milestone` with the reason as `note` (the history keeps from → to), never a
   second milestone with the same key.

## Output

Display: inline — one Yellowtail widget in the chat, cards under it (`${CLAUDE_PLUGIN_ROOT}/references/display.md` §2). Yellowtail's reads draw nothing on the way; this output is the one thing shown (§ Only the answer is drawn).

```
Created  Seed 2026 in Kestrel Robotics   /pipelines?pipeline=<id>
Target   £2,000,000 (GBP) · planning · seed · SAFE
Timeline launch 3 Nov · lead term sheet 15 Jan · lead committed 29 Jan · legal close 26 Feb
         (Timeline panel on the pipeline page; the founder sees it too)
Stages   Researching → Contacted → First meeting → Partner meeting → Due diligence → Term sheet
         → Committed → Passed
```

## What comes next

- `/find-investors` to build the shortlist for this round.
- `/round-strategy` to size, sequence and date the round properly, checking the timeline is
  feasible (legal close takes longer in some jurisdictions).
- `/add-to-pipeline` if the user already has firms or a list in mind (a list can be imported whole).
