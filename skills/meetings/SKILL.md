---
name: meetings
description: Yellowtail Club — Meetings with investors, in one command - prep (the Meeting Prep Brief for an upcoming meeting), summary (a finished meeting turned into a summary, coaching notes, deal updates and a follow-up draft), schedule (book a meeting from free calendar time or a Calendly link) and qa (the hard questions this investor will ask, with draft answers). Use when the user says "prep me for my call with", "call prep", "meeting prep", "summarise my call", "write the follow-up", "here's the transcript", "set up a call with", "find time with", "book the partner meeting", "what will [firm] ask", "hard questions", "mock partner meeting" or "diligence questions".
argument-hint: "[prep | summary | schedule | qa] [the action's own arguments]"
---

# /meetings

Meetings with investors: prepare for one, summarise it afterwards, book it, and rehearse the hard questions. One command, with an action for each job; each action's procedure is a reference in this
folder, read only when that action runs.

| Action | What it does | Arguments | Reference |
| --- | --- | --- | --- |
| **`prep`** | The Meeting Prep Brief for an upcoming meeting with an investor (or any firm on a Yellowtail pipeline) | `[meeting, firm or person] [date/time \| today \| this week] [--doc] [--email-founder]` | `references/prep.md` |
| **`summary`** | Turn a finished meeting with an investor (or any firm on a Yellowtail pipeline) into a summary, a What landed / What to improve coaching section (what resonated, what fell flat or was dodged, one or two things to change before the next call, each citing its line), a follow-up email draft to the external attendees, proposed deal updates (stage, next step, a log comment) each citing the line it came from, new people met filed as contacts, and investor intel kept as a project note | `[meeting, firm or person] or pasted notes/transcript` | `references/summary.md` |
| **`schedule`** | Book a meeting with the people on a Yellowtail deal - an intro call, a partner meeting, a diligence session - by finding free time in Google Calendar, or, when Calendly is connected, offering a single-use Calendly scheduling link for a named event type (Discovery, Snapshot walkthrough, Kickoff, Close-out or an investor call type) in the email draft; drafting the invite or the email, creating it only on approval, and logging it as a comment on the deal | `<firm or person> [purpose] [window, e.g. next week] [duration]` | `references/schedule.md` |
| **`qa`** | Prepare a founder for the hard questions an investor will ask — drawn from that firm's thesis, portfolio and recent signals in Yellowtail, the stage's norms, and what has already come up on the deal — with draft answers written only from the client's data room index (read first) and own material (deck, model, notes), each citing the document, and the weak spots flagged where no good answer exists yet | `[firm] [--stage first-meeting\|partner-meeting\|diligence]` | `references/qa.md` |

## Pick the action

1. The first word of the arguments names it (`/meetings prep …`); the rest are that action's own
   arguments.
2. No action named: match the request to the phrases below and take the closest.
3. Still unclear between two: ask one short question naming both. Never guess an action that writes.

- **`prep`** — Use when the user says "prep me for my call with Northwind", "call prep", "meeting prep", "I'm meeting Sarah Lee at 3, get me ready", "what do I need to know before the Brightwater meeting", or "brief me on my next investor meeting".
- **`summary`** — Use when the user says "summarise my call with Northwind", "call summary", "write the follow-up", "process these notes", "log the meeting", "here's the transcript", or pastes rough notes or a transcript after a pitch, partner meeting or diligence call.
- **`schedule`** — Use when the user says "set up a call with Sarah at Northwind", "find time with Brightwater next week", "book the partner meeting", "send them my Calendly", "schedule a follow-up", or "get Halcyon on the calendar".
- **`qa`** — Use when someone says "prep me for [firm]", "what will [firm] ask", "investor Q&A", "hard questions", "mock partner meeting", "diligence questions", or before a first meeting or partner meeting.

Then read `references/<action>.md` and follow it exactly: its tools, steps, rules, output and next
steps are the action's own. Read `${CLAUDE_PLUGIN_ROOT}/references/house-rules.md` first, as every
action says.

## Output

Display: inline — unless the action's reference says otherwise (some actions are a dashboard or a document): its own `## Output` section is the display for that action (`${CLAUDE_PLUGIN_ROOT}/references/display.md` §2).
