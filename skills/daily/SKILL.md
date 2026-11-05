---
name: daily
description: Yellowtail Club — The daily and weekly rhythm of a raise, in one command - briefing (the morning rundown: today's meetings, quiet deals, what is due), end-of-day (today's meetings processed, promises tracked), weekly (the week's wrap), follow-ups (every deal where the ball is in our court, with drafts) and inbox (recent Gmail matched to deals, with what follows). Use when the user says "daily briefing", "what's my day", "start my day", "end of day", "EOD", "what did I promise today", "weekly wrap", "what happened this week", "who do I need to chase", "draft my follow-ups", "sweep my inbox", "anything from investors?", or on a morning, evening, Friday or weekly schedule.
argument-hint: "[briefing | end-of-day | weekly | follow-ups | inbox] [the action's own arguments]"
---

# /daily

The rhythm of a raise: the morning briefing, the end of the day, the weekly wrap, follow-ups to chase, and an inbox sweep. One command, with an action for each job; each action's procedure is a reference in this
folder, read only when that action runs.

| Action | What it does | Arguments | Reference |
| --- | --- | --- | --- |
| **`briefing`** | Morning rundown of a raise (or any Yellowtail pipeline) - today's meetings with investors and the context for each, deals gone quiet (no comment or change in N days), next steps due today or overdue, what moved since yesterday, deals due to close soon, the round's timeline at risk (overdue or due-soon milestones), replies waiting in Gmail in both directions, and the top three things to do, each pointing at the skill that does it | `[project or pipeline] [quiet threshold in days, default 10]` | `references/briefing.md` |
| **`end-of-day`** | Close out the day on a raise (or any Yellowtail pipeline, including an advisor's own Clients pipeline) - every external meeting today processed through /meetings summary or explicitly skipped, the commitments made today in meetings and sent mail turned into next steps and comments on the right deals, a check that every deal touched today says what happens next, the round's timeline at risk and whether today moved a milestone (proposed as a dated note on it), and tomorrow's top three | `[project] [--date 2026-10-09]` | `references/end-of-day.md` |
| **`weekly`** | End-of-week wrap for a raise (or any Yellowtail pipeline, including an advisor's Clients pipeline) - deals that moved stage this week from Yellowtail's own event log, new deals, passes and wins, meetings held, commits against the round's target, what slipped (next steps whose date passed), the round's timeline at risk (overdue or due-soon milestones, and any rescheduled this week), and next week's plan | `[project \| --workspace] [--since 2026-10-03]` | `references/weekly.md` |
| **`follow-ups`** | Work through every open deal on a round (or any Yellowtail pipeline, including an advisor's Clients pipeline) where the ball is in our court or the investor has gone quiet - reads the latest Gmail thread with each deal contact, decides the right follow-up (a reply we owe, a nudge, a value-add update, a fresh signal about the firm as a hook, or a break-up email), creates Gmail drafts only, and on approval logs a deal comment and sets the next step and its date | `[pipeline] [--quiet 7] [--max 20] [firm…]` | `references/follow-ups.md` |
| **`inbox`** | Sweep recent Gmail for threads with the people and firms on the project's Yellowtail pipelines, match each thread to its deal, and propose what follows from it - stage moves ("asked for the data room → Diligence"), log comments, reply drafts, and new investor contacts who aren't in Yellowtail yet | `[lookback, e.g. 3d or 2w] [pipeline name]` | `references/inbox.md` |

## Pick the action

1. The first word of the arguments names it (`/daily briefing …`); the rest are that action's own
   arguments.
2. No action named: match the request to the phrases below and take the closest.
3. Still unclear between two: ask one short question naming both. Never guess an action that writes.

- **`briefing`** — Use when the user says "daily briefing", "morning briefing", "what's my day", "what's happening with the round", "what needs me today", "start my day", or on a weekday schedule.
- **`end-of-day`** — Use when the user says "end of day", "close out my day", "EOD", "wrap up today", "process today's calls", "what did I promise today", or on an evening schedule. The bookend to /daily briefing.
- **`weekly`** — Use when the user says "weekly wrap", "week in review", "what happened this week", "Friday summary", "wrap the week", "how did the round do this week", or on a Friday schedule.
- **`follow-ups`** — Use when the user says "follow-ups", "who do I need to chase", "follow up on the round", "draft my follow-ups", "who owes whom a reply", "chase the quiet investors", "nudge everyone", or at a weekly follow-up session.
- **`inbox`** — Use when the user says "sweep my inbox", "what investor emails need a reply", "check email for the round", "anything from investors?", "update the pipeline from my email", or on a schedule.

Then read `references/<action>.md` and follow it exactly: its tools, steps, rules, output and next
steps are the action's own. Read `${CLAUDE_PLUGIN_ROOT}/references/house-rules.md` first, as every
action says.

## Output

Display: inline — unless the action's reference says otherwise (some actions are a dashboard or a document): its own `## Output` section is the display for that action (`${CLAUDE_PLUGIN_ROOT}/references/display.md` §2).
