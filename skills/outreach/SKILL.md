---
name: outreach
description: Yellowtail Club — Writing to investors, in one command - draft (a forwardable intro blurb for a warm intro, or a short cold email on the investor's own thesis or a recent signal), update (the founder's monthly investor update for existing and pipeline investors) and objection (work through an objection or a pass and draft the reply). Use when someone says "draft outreach to [firm]", "intro blurb", "ask [mutual] for an intro", "cold email these investors", "investor update", "monthly update", "write the October update", "they said we're too early", "handle this objection" or "reply to this pushback".
argument-hint: "[draft | update | objection] [the action's own arguments]"
---

# /outreach

Writing to investors: outreach and intro blurbs, the monthly investor update, and answering an objection. One command, with an action for each job; each action's procedure is a reference in this
folder, read only when that action runs.

| Action | What it does | Arguments | Reference |
| --- | --- | --- | --- |
| **`draft`** | Draft investor outreach for a raise — a forwardable intro blurb a mutual contact can pass on (for a warm intro), or a short cold email anchored on the investor's own thesis or a recent Yellowtail signal | `<firm or person>[, <firm or person>…] [--warm via <mutual>] [--cold] [--as-founder]` | `references/draft.md` |
| **`update`** | Write the founder's monthly investor update — highlights, key metrics, lowlights, what's next, and specific asks — for existing investors and for investors in the pipeline | `[month] [--to existing\|pipeline\|both] [--list <name>] [--stages "In Conversation,Committed"]` | `references/update.md` |
| **`objection`** | Work through an objection on a deal - from an investor on a round ("too early", "valuation is too high", "market's too small", "not in our thesis", "we need a lead first", "come back with more traction", "portfolio conflict") or, on an advisor's Clients pipeline, from a founder about hiring an advisor ("your fees are too high", "we can do it ourselves", "not now") | `<firm or deal> [the objection, or a pasted email/transcript excerpt; left out, it finds the latest reply or pass]` | `references/objection.md` |

## Pick the action

1. The first word of the arguments names it (`/outreach draft …`); the rest are that action's own
   arguments.
2. No action named: match the request to the phrases below and take the closest.
3. Still unclear between two: ask one short question naming both. Never guess an action that writes.

- **`draft`** — Use when someone says "draft outreach to [firm]", "write to [partner]", "intro blurb for [firm]", "forwardable email", "ask [mutual] for an intro to [firm]", "cold email these Tier 1 investors", or "email the first ten on the round".
- **`update`** — Use when someone says "investor update", "monthly update", "update my investors", "write the October update", "send an update to the pipeline", or "keep warm investors in the loop".
- **`objection`** — Use when the user says "they said we're too early", "how do I answer [objection]", "handle this objection", "Northwind passed because…", "reply to this pushback", or pastes an investor's or founder's objection.

Then read `references/<action>.md` and follow it exactly: its tools, steps, rules, output and next
steps are the action's own. Read `${CLAUDE_PLUGIN_ROOT}/references/house-rules.md` first, as every
action says.

## Output

Display: inline — unless the action's reference says otherwise (some actions are a dashboard or a document): its own `## Output` section is the display for that action (`${CLAUDE_PLUGIN_ROOT}/references/display.md` §2).
