# Investor brief

The `brief` action of `/investors`. Arguments: `<firm name, website or Yellowtail firm>`.

What a good associate hands the founder before the first approach: does this firm fit, who to
talk to, what has already happened, what they will ask and what to lead with. It is the **Meeting
Prep Brief** for a firm with no meeting booked yet, so the layout is the one `/meetings prep` uses and a
founder who has read one can read the other. It writes nothing.

Read `${CLAUDE_PLUGIN_ROOT}/references/house-rules.md` first. The sections, sources and rules are
`${CLAUDE_PLUGIN_ROOT}/references/meeting-prep-brief.md`; follow it exactly. Answers come from
`${CLAUDE_PLUGIN_ROOT}/references/data-room-index.md`. Fit is `${CLAUDE_PLUGIN_ROOT}/references/fit-scoring.md`. Worked example:
`${CLAUDE_PLUGIN_ROOT}/references/examples-investors.md` § investor-brief.

## Tools

| Tool | For | Required |
| --- | --- | --- |
| `whoami`, `get_project`, `list_tags`, `list_pipelines` | the project, its interests named, the open round | yes |
| `search_companies`, `get_company`, `get_company_kind` | the firm: fields, tags, `attributes` read against real keys, people, latest signals | yes |
| `search_signals` | `companyId` (and each partner's `personId`), last 18 months, `status: "published"` | yes |
| `list_deals`, `get_deal` | the firm on this project's rounds (`query.companyId`, statuses `open`, `won`, `lost`); contacts and comment thread | yes |
| `list_project_notes`, `get_note` | the data room index (first); the team's notes on the firm and each partner; a saved competitor list | yes |
| Web search / fetch | thesis page, portfolio, recent deals, partner posts, fund news | yes |
| Email / calendar connectors | the last thread with the firm; a booked meeting (then `/meetings prep` is the better command) | no |

## Steps

1. **Orient.** `whoami`; `get_project` → description and interests (named via `list_tags`); the
   open round from `list_pipelines` (target, `attributes`). Competitors as in
   `meeting-prep-brief.md` § What it reads; if you had to research them, say they are your list.
2. **The firm.** `search_companies` by every key. Not in Yellowtail → the brief still works from the
   web; mark the whole brief "not in Yellowtail" and offer `/investors research` at the end. Ambiguous →
   ask.
3. **The client's brain.** The data room index (`data-room-index.md` § Finding it); missing → say so
   and use the client's files you can read.
4. **Read the rest** (`meeting-prep-brief.md` § What it reads): `get_company`, `search_signals` over
   18 months, the deals on this project's pipelines with `get_deal` for each (stage, contacts with
   the primary first, the comment thread newest first), `list_project_notes` with the firm's name and
   each partner's, the last email thread if Gmail is connected, then the web. A meeting already in
   the calendar → say so and offer `/meetings prep`.
5. **Which partner** (section 1, **Who**): rank the firm's people as `/investors contacts` does — who led
   deals most like this one, who said something relevant recently. One recommendation with one
   reason, one alternate. Prefer an existing deal contact if the thread shows a live relationship.
6. **Write the brief**, sections 1–10 in order (`meeting-prep-brief.md`). In place of the call's
   **So far**, the header carries **History so far**: the deal's stage and the dated comments and
   notes. Section 9 is **the goal for first contact** ("a 30-minute intro with Raj").

## Output

Display: document — a private, Yellowtail-branded artifact; cards inline (`${CLAUDE_PLUGIN_ROOT}/references/display.md` §5). Yellowtail's reads draw nothing on the way; this output is the one thing shown (§ Only the answer is drawn).

The brief, in the layout of `meeting-prep-brief.md` § Example, headed:

```
## Northwind Capital — brief for Voltframe, Seed 2026 · no meeting booked
Fit: **Strong** · in Yellowtail · on Seed 2026 at *First meeting* since 24 Sep
**History so far**
- 24 Sep — Jack: "Intro via Kiln. Sarah wants the grid-flex numbers before a partner meeting."
- Note 18 Sep — "Sarah at Founders Forum: keen on software-only grid plays"
```

then sections 1–10 and the Sources line. Keep it to two phone screens. Unknowns are said ("No
stated cheque range"), not filled; a conflict or "may be inactive" is in the header, never below the
angle.

## What comes next

`/outreach draft` to the partner recommended, or `/meetings prep` once a meeting is booked. Not on a
pipeline yet → `/pipeline add`. Not in Yellowtail → `/investors research`. `/meetings qa` to
rehearse the weak spots.
