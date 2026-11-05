---
name: investors
description: Yellowtail Club — Everything about finding and knowing investors, in one command - find (a ranked shortlist for the round from Yellowtail and the web), research (research a firm or a list in depth and add it to Yellowtail), brief (a read-only brief on one firm before a pitch), contacts (the people worth contacting at a firm), map (who decides at a firm on a deal, and how warm we are) and prospect (every firm in a segment, added to the database). Use when someone says "find investors", "who should we pitch", "build a target list", "add [firm]", "research and add", "enrich Northwind", "brief me on [firm]", "should we pitch", "who should we talk to at [firm]", "get me emails for", "who decides at", "are we single-threaded", "who are we missing", "find every [segment] fund", or replies create 2 7 or brief 3 to a shortlist.
argument-hint: "[find | research | brief | contacts | map | prospect] [the action's own arguments]"
---

# /investors

Find, research and brief on investors for the round: the ranked shortlist, a firm researched into Yellowtail, a pre-pitch brief, the people to contact at a firm, who decides there, and a whole segment prospected. One command, with an action for each job; each action's procedure is a reference in this
folder, read only when that action runs.

| Action | What it does | Arguments | Reference |
| --- | --- | --- | --- |
| **`find`** | Build a ranked shortlist of investors for the active project's round from Yellowtail (firms held to the project's interests, plus recent signals) and the live web (firms Yellowtail doesn't have yet) | `[count, default 20] [--snapshot] [--yellowtail-only \| --web-only] [extra criteria, e.g. "leads only", "UK only"]` | `references/find.md` |
| **`research`** | Research one investor firm or a short list in depth (thesis, stages, cheque sizes, sectors, geography, fund size and vintage, recent deals, website, LinkedIn, HQ) plus the partners worth contacting with their LinkedIn, work email, X, phone and location, proposing any missing industry or focus tag | `<firm name, website or LinkedIn URL — or a list / CSV of them>` | `references/research.md` |
| **`brief`** | A read-only brief on one investor firm before a pitch or first contact, for the active project's round, in the same layout as the Meeting Prep Brief - the partner to target and why, the fund and its status, an activity check (flagged may be inactive after 18 months without a deal), recent deals and signals, portfolio conflicts with the client's competitors, tensions to handle, likely questions with suggested answers from the client's data room index, the angle, the goal for first contact and what to ask - plus the deal's history so far on this project's pipelines | `<firm name, website or Yellowtail firm>` | `references/brief.md` |
| **`contacts`** | Find the people worth contacting at one investor firm — partners, principals, platform or IR — ranked by fit with the active project's stage and sector, each with title, background, recent investments and posts, and a contact search — LinkedIn, work email (Hunter through Yellowtail, then a published address, then a pattern only when the firm's own addresses show it; never a role inbox), X, work phone and location — each sourced and marked unverified or verified, or not found with where it looked | `<firm name, website or Yellowtail firm> [focus, e.g. "climate partner"]` | `references/contacts.md` |
| **`map`** | Map the people at one investor firm on a deal — who decides (the partners, the investment committee, who can lead at which cheque size), who we know there and how warm each relationship is (deal contacts, email threads, meetings held), whether we have a champion, who is missing, and the best way in to the people we haven't reached (warm paths through Yellowtail people and notes, founders the firm has backed, colleagues who moved) | `<firm or deal> [pipeline name]` | `references/map.md` |
| **`prospect`** | Grow Yellowtail's investor database for a whole segment rather than one round - takes a segment like "UK climate pre-seed funds" or "family offices doing B2B SaaS in DACH", searches the web for every firm in it, triages them against Yellowtail so nothing already there is researched again, and proposes only the missing firms as researched /investors research cards (five per round), filed on approval to the shared investor catalog for the project's workspace, with any missing industry or focus tag created first | `<segment, e.g. "UK climate pre-seed funds"> [max firms, default 30]` | `references/prospect.md` |

## Pick the action

1. The first word of the arguments names it (`/investors find …`); the rest are that action's own
   arguments.
2. No action named: match the request to the phrases below and take the closest.
3. Still unclear between two: ask one short question naming both. Never guess an action that writes.

- **`find`** — Use when someone says "find investors", "who should we pitch", "build a target list", "who's leading seed climate rounds in the UK", or "who are we missing".
- **`research`** — Use when someone says "add [firm]", "create these investors", "research and add", "put Kiln Climate in Yellowtail", "enrich Northwind", "fill in the gaps on these funds", or replies `create 2 7` to /investors find.
- **`brief`** — Use when someone says "brief me on [firm]", "what do we know about", "should we pitch", "who at [firm] should we go to", or replies `brief 3` to /investors find. With a meeting already booked, /meetings prep is the same brief with the meeting's attendees and threads.
- **`contacts`** — Use when someone says "who should we talk to at [firm]", "find contacts at", "find the partners at", "get me emails for Northwind", "who covers climate at", or when /lists add or /pipeline add meets a firm with no contacts.
- **`map`** — Use when someone says "who decides at Northwind", "map the stakeholders", "are we single-threaded", "who's our champion at", "how do we get to the partner", "who else should we talk to at", or before a partner meeting.
- **`prospect`** — Use when someone says "who are we missing", "build out our list of [segment]", "find every [segment] fund", "prospect for investors in", "fill the database with", or "map the [segment] landscape".

Then read `references/<action>.md` and follow it exactly: its tools, steps, rules, output and next
steps are the action's own. Read `${CLAUDE_PLUGIN_ROOT}/references/house-rules.md` first, as every
action says.

## Output

Display: inline — unless the action's reference says otherwise (some actions are a dashboard or a document): its own `## Output` section is the display for that action (`${CLAUDE_PLUGIN_ROOT}/references/display.md` §2).
