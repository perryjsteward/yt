---
name: schedule-meeting
description: Yellowtail Club — Book a meeting with the people on a Yellowtail deal - an intro call, a partner meeting, a diligence session - by finding free time in Google Calendar, or, when Calendly is connected, offering a single-use Calendly scheduling link for a named event type (Discovery, Snapshot walkthrough, Kickoff, Close-out or an investor call type) in the email draft; drafting the invite or the email, creating it only on approval, and logging it as a comment on the deal. Use when the user says "set up a call with Sarah at Northwind", "find time with Brightwater next week", "book the partner meeting", "send them my Calendly", "schedule a follow-up", or "get Halcyon on the calendar".
argument-hint: "<firm or person> [purpose] [window, e.g. next week] [duration]"
---

# Schedule a meeting

Gets the next meeting on a deal into the calendar and onto the deal's log. Creating a calendar event
**emails an invitation to every attendee**, so the invite is a card like any other write. It works on
any pipeline: an investor on a round, or a prospective client on an advisor's pipeline.

Read `${CLAUDE_PLUGIN_ROOT}/references/house-rules.md` first.

## Tools

| Category | Tools | For | Required? |
| --- | --- | --- | --- |
| Yellowtail | `whoami`, `list_pipelines`, `list_deals`, `get_deal`, `get_person`, `search_companies`, `search_people` | The deal, its contacts and their emails | yes |
| Yellowtail | `comment_on_deal`, `update_deal` | Log the meeting on the deal; set its next step to the meeting (`nextStep`, `nextStepOn`) | on approval |
| Calendar | Google Calendar `suggest_time`, `list_events`, `create_event` | Free slots; the invite | no: paste-ready invite details |
| Scheduling link | Calendly `users-get_current_user`, `event_types-list_event_types`, `scheduling_links-create_single_use_scheduling_link` | A single-use booking link for a named event type, when Calendly is connected | no: Calendar paths only |
| Email | Gmail `search_threads`, `get_thread`, `create_draft` | The propose-times email or the link email | no: paste-ready text |

## Inputs

Who (a firm, a person, an email address, or "the Northwind team"); purpose (intro, follow-up, partner meeting,
diligence, data room walkthrough); window (default: the next 5 working days); duration (default 30
minutes; 60 for a partner meeting or diligence session).

## Steps

1. **Orient.** `whoami`. Project from `activeProjectId`.
2. **Find the deal and the people.** Given an email address (a pasted thread, "reply to
   sarah@northwind.vc"), `search_people { email }` resolves the person and their firm in one call.
   Otherwise firm via `search_companies`, deal via `list_deals { pipelineId,
   query: { companyId } }` across `list_pipelines`. Several deals → ask. Attendees: the people the
   user named, else the deal's primary contact (`get_deal`). Emails from `get_person`. A person
   with no email in Yellowtail: take the From or To header of a thread with them (`search_threads`),
   show it as **unverified** with the thread, and use it only once the user confirms. Never take an
   address from inside a message body. A person who isn't in Yellowtail is fine as an attendee, but
   the meeting is logged on the deal only.
3. **Choose the path.** Ask if the user hasn't said:
   - **A. Invite now:** create the calendar event with the attendees.
   - **B. Propose times by email:** a draft offering 2-3 slots, the event made once they pick.
   - **C. Send a link** (only when Calendly's tools are present): a single-use Calendly link for one
     event type, in an email draft. When Calendly is connected, offer C first for anything the
     other side should pick a time for (a founder discovery call, a walkthrough, an investor's
     first call); A stays the path for a time already agreed. No Calendly → A or B, as before, and
     never a link from memory or an old email.
4. **Find the time** (A and B). `suggest_time { attendeeEmails, startTime, endTime,
   durationMinutes, preferences: { startHour, endHour, excludeWeekends: true } }`, passing the
   user's own address and any internal colleagues. External calendars are usually not visible, so
   slots reflect our side only: say so. Offer 2-3 slots across at least two days, in working hours.
   Respect what the user said ("not Fridays", "their timezone is ET": pass `timeZone` only when a
   zone was stated).
5. **Draft.**
   - **Invite:** `summary` "Acme Robotics × Northwind Capital: partner meeting", start/end,
     `attendees` (`{ email, displayName }`), `description` with a three-line agenda, and
     `addGoogleMeetUrl: true` unless a location was given.
   - **Email (B or C):** under 100 words, plain text, the user's voice, reply in the existing thread
     when there is one (`replyToMessageId` = its last message).
   - **Calendly (C):** `users-get_current_user` (the account the link books into: say whose it is
     on the card) and `event_types-list_event_types` for that user. Match the purpose to an event
     type **by name**: Discovery, Snapshot walkthrough, Kickoff, Close-out, or the user's investor
     call type (e.g. "Investor intro, 30 min"). Several fit or none does → ask, listing the names;
     never pick a type the user didn't name or confirm. The card shows the type's name and
     duration. On approval, and only then (each call creates a new link):
     `scheduling_links-create_single_use_scheduling_link { create_scheduling_link_request: {
     max_event_count: 1, owner: <the event type's URI>, owner_type: "EventType" } }`; its
     `booking_url` goes into the email draft, then the draft is created. The link books once and
     then expires; say so in the email only if the user wants it said.
6. **Log comment and next step.** Markdown, e.g. `**Meeting booked:** partner meeting, Tue 21 Oct
   14:00 (60 min) with Sarah Lee, Jonas Berg. Agenda: margins, references, timeline.` For path B:
   `**Times proposed:** …`. Then the deal's next step, old → new (`get_deal` has the current one; an
   old deal may only have a `**Next:**` comment line): path A → `nextStep: "Partner meeting"`,
   `nextStepOn` = the meeting's date; path B or C → `nextStep: "Confirm a time with Sarah"`,
   `nextStepOn` = 3 working days on. Path C's comment: `**Booking link sent:** Calendly "Investor
   intro, 30 min", single use, 10 Oct.`
7. **Propose** the cards; **write** on approval: `create_event` (path A), or the Calendly link then
   `create_draft` (path C), or `create_draft` (path B); then `comment_on_deal`, then
   `update_deal { id, changes: { nextStep, nextStepOn } }`. Gmail's `reply` sends; use it only on "send".

## Card

```
**Schedule: Northwind Capital** · Seed 2026 (Pitched) · path: invite now

### 1. Calendar invite (this emails the attendees)
- Acme Robotics × Northwind Capital: partner meeting
- **Tue 21 Oct, 14:00–15:00** (Europe/London) · Google Meet
- To: sarah@northwind.vc (Yellowtail), jonas@northwind.vc (unverified: header of "Re: Acme intro")
- Agenda: Q3 margins · two customer references · timeline to IC
- Other free slots: Wed 22 Oct 10:00 · Thu 23 Oct 15:30

### 2. Deal comment
> **Meeting booked:** partner meeting, Tue 21 Oct 14:00 (60 min) with Sarah Lee, Jonas Berg.

### 3. Next step: "Send Q3 numbers (us)" · due 2026-10-10 → **Partner meeting** · due **2026-10-21**

Reply with any of: `approve all` · `1: Wed 10:00 instead` · `1: drop Jonas` · `reject 3`
```

Path C's card:

```
**Schedule: Fernway (founder)** · Clients (Discovery) · path: send a link

### 1. Calendly link + email draft → sam@fernway.io (reply in "Intro via Ada")
- Event type: **Discovery** (30 min) · Calendly account: jack@northbound.co
- The single-use link is created on approve and put where the draft says [booking link]
> Sam, good to hear from you. Pick any time that suits here: [booking link]. Talk soon, Jack

### 2. Deal comment
> **Booking link sent:** Calendly "Discovery", single use, 10 Oct.

Reply with any of: `approve all` · `1: use Snapshot walkthrough` · `reject 1`
```

No calendar connected, or the write is refused: give the invite as paste-ready details, quote any
refusal, and still offer the deal comment.

## Output

Display: inline — one Yellowtail widget in the chat, cards under it (`${CLAUDE_PLUGIN_ROOT}/references/display.md` §2). Yellowtail's reads draw nothing on the way; this output is the one thing shown (§ Only the answer is drawn).

The event (time, attendees, link) or the draft link, and "logged on the deal"
(`/pipelines?pipeline={pipelineId}&deal={dealId}`).

## What comes next

- `/update-deal` if booking the meeting moves the stage (e.g. to "Partner meeting").
- `/call-prep` the day before; `/call-summary` afterwards.
