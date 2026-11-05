---
name: lists
description: Yellowtail Club — Lists of investor contacts, in one command - create (a named list such as Tier 1 climate leads), add (firms' relevant contacts or named people onto a list) and export (a CSV for outreach, or pushed into Hunter). Use when the user says "make a list", "create a list called", "new target list", "add X to my list", "put these funds on Tier 1", "track these investors", "export the Tier 1 list", "give me a CSV of this list", "push this list to Hunter", or "get the list ready for outreach".
argument-hint: "[create | add | export] [the action's own arguments]"
---

# /lists

Lists of investor contacts: make one, add people to it, export it for outreach. One command, with an action for each job; each action's procedure is a reference in this
folder, read only when that action runs.

| Action | What it does | Arguments | Reference |
| --- | --- | --- | --- |
| **`create`** | Create a named list in a Yellowtail project for tracking people, such as "Tier 1 climate leads", "Angels from the Founders Forum dinner" or "Warm intros via Sarah" | `<list name> [description] [people or firms to add]` | `references/create.md` |
| **`add`** | Add investors to a Yellowtail list | `<list name> <firms or people…>` | `references/add.md` |
| **`export`** | Export a Yellowtail list of investor contacts for outreach, as a CSV (first name, last name, email, email status, firm, title, LinkedIn, website and a personalisation line from the firm's latest signal or description) or pushed into a Hunter Leads list through Yellowtail (never a Hunter campaign) | `[list name] [csv \| hunter]` | `references/export.md` |

## Pick the action

1. The first word of the arguments names it (`/lists create …`); the rest are that action's own
   arguments.
2. No action named: match the request to the phrases below and take the closest.
3. Still unclear between two: ask one short question naming both. Never guess an action that writes.

- **`create`** — Use when the user says "make a list", "start a list of…", "create a list called…", "new target list", or wants to group investors' contacts before working them.
- **`add`** — Use when the user says "add X to my list", "put these funds on Tier 1", "track these investors", "add the angels from last night to a list", or picks names from a shortlist to track.
- **`export`** — Use when someone says "export the Tier 1 list", "give me a CSV of this list", "push this list to Hunter", "get the list ready for outreach", or "/lists export".

Then read `references/<action>.md` and follow it exactly: its tools, steps, rules, output and next
steps are the action's own. Read `${CLAUDE_PLUGIN_ROOT}/references/house-rules.md` first, as every
action says.

## Output

Display: inline — unless the action's reference says otherwise (some actions are a dashboard or a document): its own `## Output` section is the display for that action (`${CLAUDE_PLUGIN_ROOT}/references/display.md` §2).
