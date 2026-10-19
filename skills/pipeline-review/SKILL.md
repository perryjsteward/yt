---
name: pipeline-review
description: Yellowtail Club — Check the health of a round (or any Yellowtail pipeline, including an advisor's own client pipeline) — deal count and value per stage, deals stuck with no update or comment in N days, deals with no primary contact or no next step, soft versus hard commits against the target, concentration on one or two firms, and the actions that would move the round this week. Before anything is contacted it gives a short pre-launch view instead (what is ready, what is missing, the next dates from the round's timeline). Read-only; changes go through /update-deal. Use when someone says "pipeline review", "how's the round going", "round health", "what's stuck", "where are we against target", "who haven't we followed up", or at a weekly round check-in.
argument-hint: "[pipeline name] [--stale 14] [--include-closed]"
---

# /pipeline-review

A round dies quietly: a firm that took a first meeting three weeks ago and heard nothing since,
a "committed" angel nobody has sent documents to, a total that looks healthy because one fund is
most of it. This skill reads the pipeline the way an advisor does on a Monday morning and says
what needs doing, with the deal behind every claim. It works on any pipeline — a raise, or the
practice's Clients pipeline.

Read `${CLAUDE_PLUGIN_ROOT}/references/house-rules.md` first.

## Tools

| Tool | For | Required? |
| --- | --- | --- |
| `whoami` | Active project | yes |
| `list_pipelines`, `get_pipeline` | Pick the pipeline; stages in order, `target` and `currency`, status, `openValues` | yes |
| `list_milestones` | The round's timeline: overdue and due-soon dates; the next dates in pre-launch mode | yes |
| `list_project_notes` | The data room index, in pre-launch mode | pre-launch |
| `list_deals` | Open deals (no `status` = open only) with `sort: "lastActivityAt"` (quietest first); again with `status: "won"` and `"lost"` for commits and passes. `limit` up to 100; follow `nextCursor` | yes |
| `list_history` (Yellowtail) | The event log: what moved in the last week, `{ projectId, entityType: "deal", since: <7 days ago, ISO 8601> }` | yes |
| `get_deal` | The comment thread: commit evidence, pass reasons, and a `**Next:**` line on old deals with no `nextStep` | only where step 3, 4 or 6 needs the comments |
| Calendar (Google Calendar `list_events`, not Yellowtail's) | Meetings booked with a deal's contacts in the next two weeks — a deal with one booked isn't stuck | optional |

## Steps

1. **Orient.** `whoami`; `list_pipelines` for the active project; use the one named, or the only one
   with status `raising`, or ask. `get_pipeline` for stages, target, currency and status;
   `list_milestones` for the timeline. Nothing contacted yet → **Pre-launch mode** (below) instead
   of steps 3–7.
2. **Read every deal.** `list_deals { pipelineId, query: { sort: "lastActivityAt" } }` (open,
   oldest activity first, so the stuck deals come first), paging until `meta.hasMore` is false; then
   won and lost.
   Say "N deals read" — never review a partial page as if it were the round.
3. **Classify stages.** Map each stage to a phase by name: *early* (researching, contacting,
   contacted), *active* (in conversation, meeting, diligence, partner meeting), *soft commit*
   (verbal, soft circle, committed-not-signed), *hard commit* (term sheet signed, docs signed,
   closed), *out* (passed). The default stages are Researching, Contacting, Contacted, In
   Conversation, Committed, Passed. **"Committed" is ambiguous**: count it as soft unless the deal
   is `won`, or its comments show signed documents or money received. If stage names don't map
   clearly, ask once, before the numbers.
4. **Per-deal facts.** For each open deal, all from the `list_deals` row: stage, value + currency,
   owner, `stageAt` (days in stage), last activity = `lastActivityAt` (the latest of its update,
   stage change and newest comment), primary contact (a contact with `isPrimary`), next step =
   `nextStep` with its due date `nextStepOn`, expected close. A deal with no `nextStep` may be an old
   one that kept its next step as a `**Next:**` comment line: `get_deal` and use the newest such
   line, marked "(from comment)".
4b. **What moved.** `list_history { projectId, entityType: "deal", since: <7 days ago> }`, following
   `nextCursor`, keeping this pipeline's deals (there is no pipeline filter; match `entityId` to the
   deals read in step 2): `stage_changed` (from → to), `status_changed` (wins, passes, reopens),
   `created`, `owner_changed`, `contact_added`. This is the "since last week" line; never infer a
   move from `stageAt` alone.
5. **Flags.** A deal can carry several:
   - **Stuck**: no activity in `--stale` days (default 14; 7 for deals in an active or commit phase).
   - **Long in stage**: `stageAt` older than twice the stale threshold.
   - **No primary contact** / **no contacts at all**.
   - **No next step**, or a next step whose date (`nextStepOn`) has passed.
   - **No value** on a deal in an active or commit phase (can't count it against target).
   - **Close date passed** while still open.
   - **Single-threaded**: one contact on a deal in an active or commit phase.
6. **Roll up.**
   - Per stage: count, value **per currency** (never add currencies), median days in stage,
     flagged count.
   - Against target: hard commits, soft commits, and active value in the round's `currency` (the
     `target` is in it). Deals in other currencies are listed separately, unconverted. Coverage =
     (soft + active) ÷ remaining target, as a ratio with the arithmetic shown.
   - Concentration: the share of committed value from the largest one and two deals.
   - Lead status: is there a deal in a commit phase whose comments or notes name it as the lead?
     If not, and the round is past its first weeks, that is the headline.
   - Passes (lost): count, and the reasons where comments give them — patterns matter.
   - Timeline: overdue and due-soon milestones (`status` `overdue` / `due_soon`), and whether the
     pipeline can still make `lead_committed` and `legal_close` as they stand.
7. **Recommend** at most seven actions for this week, each tied to deals: "Follow up Ember (no reply
   since 22 Sep, In Conversation)", "Set a primary contact on 4 deals", "Send closing docs to the two
   soft commits". Where a change is needed, name the `/update-deal` call it would take.

## Pre-launch mode

Nothing contacted yet (the pipeline's `status` is `planning`, or every open deal is still in the
first stage and none is won or lost) → a per-stage table, stuck-deal list or coverage ratio would be empty or made up, so skip it. Say "Pre-launch:
nothing contacted yet" and give a short readiness view instead:

- **Ready**: what is in place, with counts: deals on the round, how many tiered (a tier in the
  deal's notes or comments, or a `/round-strategy` doc), lead candidates named, deals with a
  primary contact, contacts with an email (and how many verified), the data room index
  (`list_project_notes`, `q: "Data room index"`) and its date.
- **Missing**: what launch needs and isn't there: no tiers, no lead candidates, deals with no
  primary contact or no email, no data room index or its known gaps, no target or currency on the
  round.
- **Next dates**: from the timeline (`list_milestones`): the next two or three milestones
  (`launch`, `outreach_start`, `calls_start`, …) with their status and days to go; overdue ones
  first. No milestones → say so and point at `/create-pipeline` or `/round-strategy` to set them.
- **Before launch**: at most five actions, each naming the command (`/find-contacts`,
  `/round-strategy`, `/data-room`, `/draft-outreach`).

Ten lines or so. No stage table, flags or coverage.

## Output

Display: dashboard — a live Yellowtail Dashboard artifact for the project, made once and reused; cards inline (`${CLAUDE_PLUGIN_ROOT}/references/display.md` §4). Yellowtail's reads draw nothing on the way; this output is the one thing shown (§ Only the answer is drawn).

```
Seed 2026 — Fernway · raising · target $2.5m · 34 open, 2 won, 6 passed (read 9 Oct 2026)

Headline: $0.6m hard + $0.4m soft = 40% of target; no lead yet; 9 of 14 active deals stuck.
Since last week: 3 moved forward (Lantern → Diligence, …), 1 passed (Greyhound), 2 added.

| Stage | Deals | Value | Median days in stage | Flagged |
…
Commits: hard $0.6m (2) · soft $0.4m (3, all "Committed", none signed) · other currencies: €150k (1)
Concentration: largest deal 50% of committed

Needs attention
| Deal | Stage | Flags | Last activity | Suggested action |
…

This week
1. …
```

Link every deal: `/pipelines?pipeline={pipelineId}&deal={dealId}`. Say which inputs were read and
which weren't (calendar not connected, comments not read for early-stage deals).

## Rules specific to this skill

- Read-only. Never change a stage, value or contact here; propose it and hand it to `/update-deal`.
- Never treat a stage label as a commitment the comments contradict; say which you went by.
- Never convert currencies or invent a value for a deal with none.

## What comes next

- `/update-deal` for the fixes; `/draft-outreach` for follow-ups to stuck deals.
- `/weekly-wrap` for the week's story. Advisors running a client's raise:
  `/yt-raise:client-update` to send the founder this week's picture.
