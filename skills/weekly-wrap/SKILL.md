---
name: weekly-wrap
description: Yellowtail Club — End-of-week wrap for a raise (or any Yellowtail pipeline, including an advisor's Clients pipeline) - deals that moved stage this week from Yellowtail's own event log, new deals, passes and wins, meetings held, commits against the round's target, what slipped (next steps whose date passed), the round's timeline at risk (overdue or due-soon milestones, and any rescheduled this week), and next week's plan. Covers the active project, or every project in the workspace when asked. Saves the wrap as a Google Drive doc when connected, otherwise markdown; advisors can turn it into the founder's report with /yt-raise:client-update. Read-only apart from the doc. Use when the user says "weekly wrap", "week in review", "what happened this week", "Friday summary", "wrap the week", "how did the round do this week", or on a Friday schedule.
argument-hint: "[project | --workspace] [--since 2026-10-03]"
---

# Weekly wrap

The Friday read: what actually happened to the round this week, not what the board looks like now.
Stage moves come from Yellowtail's event log, so "moved from Contacted to In Conversation on
Tuesday, by Sam" is a fact, not an inference. For an advisor running a client's raise it is also
the raw material for the founder's report, which `/yt-raise:client-update` turns into an email.

Read `${CLAUDE_PLUGIN_ROOT}/references/house-rules.md` first.

**Run it on a schedule.** To have it every Friday afternoon, set it up as a scheduled task (Claude
Code's `/schedule`, or a routine in the Claude app) with the prompt `/weekly-wrap`. The schedule is
yours, in Claude; Yellowtail itself runs nothing on its own. A scheduled run only reads (below).

## Tools

| Category | Tools | For | Required? |
| --- | --- | --- | --- |
| Yellowtail | `whoami`, `get_project`, `list_pipelines`, `get_pipeline` | Projects in scope; rounds, stages, `target` and `currency`, status | yes |
| Yellowtail | `list_milestones { pipelineId }`; `list_history { entityType: "pipeline", entityId, verb: "milestone_rescheduled" }` | The round's timeline; dates moved this week (from → to, with the note) | yes |
| Yellowtail | `list_history { projectId, entityType: "deal", limit: 100 }`, paged with `cursor` | The week's history: `created`, `stage_changed {from, to}`, `status_changed {from, to}`, `value_changed`, `owner_changed`, `contact_added` | yes |
| Yellowtail | `list_deals` (open; then `status: "won"`, `"lost"`) | Current state, `nextStep`, `nextStepOn`, `lastActivityAt`, value and currency | yes |
| Yellowtail | `get_deal` | Comments this week: pass reasons, commitments, what's next | for deals with events this week |
| Calendar | Google Calendar `list_events` | Meetings held this week; next week's meetings | no: say so; never "no meetings" |
| Meeting notes | Granola, Fireflies, Otter, Wispr Flow | Which meetings have notes | no |
| Docs | Google Drive `create_file` | The wrap as a doc | no: markdown in the reply |

Yellowtail `list_history` is the deal history; Calendar `list_events` is the calendar.

## Steps

1. **Orient and scope.** `whoami`. Default scope: the active project. "The whole workspace" or
   `--workspace`: every project in `whoami` for the active project's workspace, each wrapped in its
   own section, with a short roll-up at the top. Window: Monday 00:00 to now in the user's timezone,
   or `--since`. State the window.
2. **The week's events.** Per project, Yellowtail `list_history { projectId, entityType: "deal", since:
   <window start, ISO 8601 with offset> }`, following `nextCursor`. Rows are newest first; a deal that moved twice
   shows both. Say "N events read". Group by deal:
   - **Moved:** `stage_changed`. Report first `from` → last `to` in the week, with dates and
     `actorName`. A move back to an earlier stage (by `get_pipeline` order) is a **slip back**.
   - **New:** `created` (title, stage, firm).
   - **Won / passed:** `status_changed` to `won` / `lost`. A pass's reason comes from the deal's
     comments this week (`get_deal`); none written → "no reason logged", never a guessed one.
   - **Other:** `value_changed`, `owner_changed`, `contact_added`, worth a line only if material.
3. **Meetings held.** Calendar `list_events` for the window, events with an external attendee, each
   mapped to a deal (`search_people { email }`, then the firm's deal). Count per deal; mark which have
   notes. Meetings with nobody in Yellowtail are listed as such.
4. **Commits against target.** Per round: `target` and `currency` from `get_pipeline`; **hard** = won deals, plus
   deals whose comments show signed documents or money received; **soft** = deals in a soft-commit
   stage (as `/pipeline-review` classifies stages; "Committed" counts as soft unless won or signed).
   Value per currency, never converted; the target is in the round's currency, so only deals in it
   count against it and the rest are listed beside it. Show the change since the window began (deals that became won or moved
   into a commit stage this week).
5. **What slipped.** Open deals whose `nextStepOn` passed during the week with no event or comment
   after it; deals whose `expectedCloseOn` passed; deals with no activity all week
   (`lastActivityAt` before the window) in an active or commit stage. Old deals without `nextStep`
   fall back to a `**Next:**` line in the newest comment that has one.
5b. **Timeline at risk.** `list_milestones` per round: milestones `overdue` or `due_soon`, with days
   over or to go and their notes, plus any rescheduled this week (`list_history`, verb
   `milestone_rescheduled`: from → to, the note, who). An overdue milestone is next week's first
   priority. None at risk → "Timeline on track" in one line.
6. **Next week.** Calendar meetings next week mapped to deals; deals with `nextStepOn` next week;
   expected closes next week; then at most five priorities, each tied to a deal and a skill.
7. **The doc.** Show the wrap, then one card: save it to Drive as `<Project> — Weekly wrap
   (<Mon date>–<Fri date>)` (`create_file`). No Drive → the markdown is the deliverable.

## Output

Display: dashboard — a live Yellowtail Dashboard artifact for the project, made once and reused; cards inline (`${CLAUDE_PLUGIN_ROOT}/references/display.md` §4). Yellowtail's reads draw nothing on the way; this output is the one thing shown (§ Only the answer is drawn).

```
## Week of 6 Oct · Acme Robotics · Seed 2026 · 41 events read

**Headline** · $0.9m hard + $0.5m soft of $2.5m (56%, +$0.4m this week). Brightwater signed; still no lead.

**Moved** (6)
- Northwind Capital: Pitched → Partner meeting (Tue, Sam) · next: Q3 numbers, due 10 Oct
- Ember VC: In Conversation → **Contacted** (slip back, Thu, you) · "wants to see Q4"
**New** (4) · Lantern, Greyhound, Fjord, Osprey (all Researching)
**Won** (1) · Brightwater $400k · **Passed** (2) · Halcyon ("no hardware this year"), Kite (no reason logged)
**Meetings held** · 7 with 6 firms (5 with notes) · 2 not in Yellowtail
**Slipped** · Fjord next step due 7 Oct, nothing since · Ember close date 30 Sep passed
**Timeline at risk** · ⚠ Lead term sheet due 7 Oct, overdue 3 days · Legal close moved 31 Dec → 29 Jan
(Wed, Sam: "notarial deed timing")
**Next week** · 4 meetings · 3 next steps due · priorities:
1. Lead term sheet overdue: done, or move it with the reason → `/close-plan`
2. Northwind partner meeting Tue → `/call-prep Northwind`
3. Chase Fjord (overdue) → `/follow-ups Fjord`

Sources: Yellowtail events ✓ · Calendar ✓ · notes: Granola ✓ · Drive: not connected
```

```
### 1. Save to Drive
- "Acme Robotics — Weekly wrap (6–10 Oct)" in My Drive

Reply with: `approve 1` · `1: put it in <folder>` · `reject 1`
```

Link every deal `/pipelines?pipeline={pipelineId}&deal={dealId}`. A section with nothing in it
says so in one line.

## Rules specific to this skill

- Read-only in Yellowtail. Fixes go to `/update-deal`; this skill only names them.
- "Moved" means a `stage_changed` event in the window. Never infer movement from `updatedAt`.
- On a schedule: the wrap and the Drive doc only if the schedule was set up to save it.

## What comes next

- Advisors running a client's raise: `/yt-raise:client-update` to turn this into the founder's
  weekly email. A founder's own monthly note to investors is `/investor-update`.
- `/pipeline-review` for the round's full health; `/follow-ups` for what slipped.
