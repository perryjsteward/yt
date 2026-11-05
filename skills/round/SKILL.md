---
name: round
description: Yellowtail Club — Plan, forecast and close a round, in one command - strategy (how much, which instrument, the sequence), forecast (where and when it lands, with what-ifs), close (a dated close plan from term sheet to wires), passes (win/loss from the passes and commits), term-sheet (clause by clause in plain English), pitch (the deck reviewed slide by slide), data-room (build, audit or share it in Google Drive) and portfolio (the advisor's view across every client). Use when someone says "plan the round", "how much should we raise", "forecast the round", "when do we close", "close plan", "we have a term sheet, what now", "why are investors passing", "review this term sheet", "review my deck", "set up the data room", "are we diligence-ready", "portfolio review" or "how are all my clients doing".
argument-hint: "[strategy | forecast | close | passes | term-sheet | pitch | data-room | portfolio] [the action's own arguments]"
---

# /round

Plan, forecast and close the round: strategy, forecast, close plan, pass analysis, term sheet, pitch deck, data room, and the advisor's portfolio view. One command, with an action for each job; each action's procedure is a reference in this
folder, read only when that action runs.

| Action | What it does | Arguments | Reference |
| --- | --- | --- | --- |
| **`strategy`** | Plan a fundraising round before outreach starts — how much to raise (runway months × burn plus a buffer, tied to the milestones that make the next round fundable), which instrument, the valuation context, the sequence (find the lead first, then fill), a tiered target list built from the project's pipeline and Yellowtail's catalog, a timeline dated backwards from legal close as a feasibility check up front (legal lead times by jurisdiction and instrument), and a readiness checklist (deck, model, data room, references) | `[pipeline name] [--target 2.5m EUR] [--legal-close 2027-03-31]` | `references/strategy.md` |
| **`forecast`** | Forecast where a round will land and when — money in the bank, commit, best case and pipeline against the round's target, per currency, with each open deal's likelihood argued from its evidence (stage, days in stage, last activity, next step and its date, what the comments say), what changed since the last forecast, and slip scenarios ("if Northwind slips a month", "if Lantern passes", "if Brightwater comes in at half") showing the likely total and close date | `[pipeline name] [--what-if "<firm> slips 1 month \| passes \| at <amount>"] [--since 2026-10-02]` | `references/forecast.md` |
| **`close`** | Get a round from soft circles and a term sheet to money in the bank - a mutual close plan with dated steps (term sheet signed, confirmatory diligence, legal documents, board and shareholder approvals, signatures, wires, the closing announcement), an owner on each side for every step, and what is still outstanding per investor in the round | `[pipeline or deal] [--legal-close 2026-12-15]` | `references/close.md` |
| **`passes`** | Win/loss review of a round — or across several rounds or every project in the workspace — from the investors who passed and the ones who committed | `[pipeline name \| --all-rounds \| --workspace] [--since 2026-07-01]` | `references/passes.md` |
| **`term-sheet`** | Explain a venture term sheet clause by clause in plain English — price and valuation, the option pool shuffle, liquidation preference, participation, anti-dilution, board, protective provisions, pro rata, information rights, founder vesting, drag-along and tag-along, no-shop and exclusivity — show what each means for the founder in money and control, flag terms that look off-market as questions to put to a lawyer, give each clause a suggested action (accept, ask for, push back on, ask your lawyer about), and compare several term sheets side by side in one table | `<term sheet upload(s) or Drive link(s)> [--deal <firm>]` | `references/term-sheet.md` |
| **`pitch`** | Review a pitch deck slide by slide against what this round's investors look for — stage-appropriate expectations for problem, solution, why now, market, traction, business model, competition, team, and the ask and use of funds — and against the specific firms on the project's pipeline and their theses in Yellowtail | `<deck upload or Drive link> [pipeline name] [--for <firm>]` | `references/pitch.md` |
| **`data-room`** | Build or audit a fundraising data room in Google Drive — the index investors expect at this stage (company, deck, financials and model, cap table, legal, team, product, traction, IP), what exists, what's missing or stale, a folder structure created on approval, and who has access against where their deal stands | `[audit \| build \| share <firm> \| --refresh] [Drive folder link or name] [pipeline name]` | `references/data-room.md` |
| **`portfolio`** | The advisor's view across every client in the workspace — for each client project, the round's status, target against money in and committed, this week's momentum from the event log (stage moves, wins, passes, new deals), stalled deals, the timeline at risk (overdue or due-soon milestones on each round), risks (no lead, a commit gone quiet, a close date passing, one fund carrying the round), and where the advisor's time should go this week, ranked | `[workspace name] [--since 2026-10-02] [--include-practice]` | `references/portfolio.md` |

## Pick the action

1. The first word of the arguments names it (`/round strategy …`); the rest are that action's own
   arguments.
2. No action named: match the request to the phrases below and take the closest.
3. Still unclear between two: ask one short question naming both. Never guess an action that writes.

- **`strategy`** — Use when someone says "plan the round", "how much should we raise", "round strategy", "who do we go to first", "build the target list", "are we ready to raise", or is about to start a raise for a client or their own startup.
- **`forecast`** — Use for "forecast the round", "where will we land", "commit vs best case", "when do we close", "what if Northwind slips", "can we still close without Halcyon", "model losing [firm]", or before a board or founder call about the raise.
- **`close`** — Use when the user says "close plan", "closing checklist", "how do we get to close", "we have a term sheet, what now", "plan the close", "who still needs to sign", "chase the wires", "mutual action plan", or a round reaches soft commit.
- **`passes`** — Use when someone says "why are investors passing", "pass analysis", "win/loss", "what are the passes telling us", "review the passes", "why did we lose Halcyon and the others", "why aren't prospects signing", or after a run of passes.
- **`term-sheet`** — Use when someone uploads or pastes a term sheet and says "review this term sheet", "explain these terms", "is this a good term sheet", "compare these two offers", "what does 1x participating mean", or "what should I push back on".
- **`pitch`** — Use when someone uploads or links a deck and says "review my deck", "pitch review", "is this deck ready", "what will investors think of this", "tear this deck apart", or before outreach starts.
- **`data-room`** — Use when someone says "set up the data room", "audit the data room", "refresh the data room index", "what's missing from the data room", "are we diligence-ready", "who has data room access", "share the data room with Northwind", or when a deal reaches diligence.
- **`portfolio`** — Use when an advisor says "portfolio review", "how are all my clients doing", "which rounds need me this week", "Monday review", "across all projects", "where should I spend my time", or at the start of the week.

Then read `references/<action>.md` and follow it exactly: its tools, steps, rules, output and next
steps are the action's own. Read `${CLAUDE_PLUGIN_ROOT}/references/house-rules.md` first, as every
action says.

## Output

Display: inline — unless the action's reference says otherwise (some actions are a dashboard or a document): its own `## Output` section is the display for that action (`${CLAUDE_PLUGIN_ROOT}/references/display.md` §2).
