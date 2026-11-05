# /round forecast

The `forecast` action of `/round`. Arguments: `[pipeline name] [--what-if "<firm> slips 1 month | passes | at <amount>"] [--since 2026-10-02]`.

`/pipeline review` asks "is the round healthy". This asks "how much money, by when, and how sure
are we" — the number an advisor gives a founder and a founder gives the board. Every figure ties to
a deal that was read, every likelihood carries its evidence, and nothing is converted between
currencies. A slip scenario is just the same arithmetic with one deal moved.

Read `${CLAUDE_PLUGIN_ROOT}/references/house-rules.md` first.

## Tools

| Tool | For | Required? |
| --- | --- | --- |
| `whoami` | Active project | yes |
| `list_pipelines`, `get_pipeline` | The round: stages in order, `target` and `currency`, status, `openValues` | yes |
| `list_milestones` | The round's timeline: `lead_committed` and `legal_close` to forecast against; the next dates in pre-launch mode | yes |
| `list_project_notes` | The data room index, in pre-launch mode | pre-launch |
| `list_deals` | Open deals (no `status`), then `status: "won"` and `"lost"`; page `nextCursor` until `meta.hasMore` is false | yes |
| `get_deal` | Comments (the evidence), contacts, `nextStep`, `nextStepOn`, `lastActivityAt` | every deal in Commit or Best case |
| `list_history` | `{ entityType: "deal", projectId, verb: "stage_changed" \| "status_changed" \| "value_changed" }`: what moved since the last forecast | for the "changed" section |
| Calendar (Google Calendar `search_events`; not Yellowtail's `list_history`) | Partner meetings or IC dates booked with a deal's contacts | optional |
| Email (Gmail `search_threads`) | Late-stage threads (term sheet, docs, wire) for commit deals | optional |

## Steps

1. **Orient.** `whoami`; `list_pipelines`; the one named, else the only one with status `raising`,
   else ask. `get_pipeline` for stages, `target`, `currency` and status. No target → say so and
   forecast totals only; never invent one. `list_milestones` for the dates the round is held to.
   Nothing contacted yet → **Pre-launch mode** (below), not a forecast.
2. **Read every deal**, open, won and lost. Say "N deals read".
3. **Bucket each deal.** Map stages to phases by name as `/pipeline review` does (early, active,
   soft commit, hard commit, out), then:
   - **In the bank**: `won`, or a commit-phase deal whose comments show signed documents or money
     received (quote the line).
   - **Commit**: a soft-commit-phase deal with a stated ticket (`value` set), activity in the last
     14 days and a dated next step towards documents. "Committed" with none of that is Best case.
   - **Best case**: active-phase deals with recent activity (≤ 14 days) and a next step, plus
     commit-phase deals missing one of the commit tests above.
   - **Pipeline**: everything else open (early phase, or active but stale).
   - **Out**: `lost`. Not counted; listed under changes if lost in the window.
   If the stages don't map clearly, ask **once** before any number, and say which mapping you used.
4. **Likelihood per deal** (Commit and Best case; Pipeline in aggregate). One word — *high*,
   *medium*, *low* — and the evidence behind it, from these and nothing else:
   stage and days in it (`stageAt`); `lastActivityAt`; `nextStep` and whether `nextStepOn` has
   passed (old deals: the newest `**Next:**` line in comments or notes); what the comments say
   (partner meeting held, IC date, diligence requests, a stated concern); contacts (single-threaded
   or not); whether it is the lead. Never use a percentage nobody stated; "medium" with three lines
   of evidence is more honest than "60%".
5. **When.** Expected close per bucket: the deal's `expectedCloseOn`, else a date its comments
   state, else "no date". The round's earliest realistic close = when enough Commit + In the bank
   value reaches the minimum the founder will close on (ask for it once if it matters; default:
   the target) — name the deal that sets that date. Set it against the timeline's `lead_committed`
   and `legal_close` (quoted with their notes): a forecast close later than `legal_close` is the
   headline, and `/round close` is where the milestone gets moved.
6. **Totals, per currency.** For each currency: In the bank, Commit, Best case, Pipeline, as sums
   and counts. The `target` is in the round's `currency`, so only deals in that currency are
   measured against it: gap = target − (bank + commit), coverage = (best case + pipeline) ÷ gap,
   arithmetic shown. Deals in other currencies get their own lines, unconverted, with "not counted
   against the target". If the user gives a rate, use it, say whose it is, and show both.
7. **What changed.** `--since`, else the last 7 days. `list_history { projectId, entityType: "deal", since:
   <window start> }`, following `nextCursor`: stage moves (`payload.from.name →
   payload.to.name`), wins and passes, value changes. Each with who did it (`actorName`). If the
   user pasted last week's forecast, diff the buckets too.
8. **Slip scenarios.** For each `--what-if` (or offer the two that matter most: the largest Commit
   deal slipping a month, and the lead passing):
   - **Slips N weeks**: move its close date; recompute the round's earliest close and what lands
     before the founder's deadline if there is one.
   - **Passes**: remove it; recompute totals, gap and coverage. If it was the lead, say the round has
     no lead and what that does to the rest (followers waiting on a lead are Best case at most).
   - **Smaller ticket**: replace the value; recompute.
   Then the **backfill**: Best case deals that could realistically cover the gap in time — recent
   activity, one stage from commit, a close date in range — with what has to happen and by when.
   A deal needing more than one stage in under two weeks is not a realistic backfill; say so.

## Pre-launch mode

Nothing contacted yet (the pipeline's `status` is `planning`, or every open deal is still in the
first stage and none is won or lost) → a forecast (in the bank, commit, best case, coverage, slip scenarios) would be empty or made up, so skip it. Say "Pre-launch:
nothing contacted yet" and give a short readiness view instead:

- **Ready**: what is in place, with counts: deals on the round, how many tiered (a tier in the
  deal's notes or comments, or a `/round strategy` doc), lead candidates named, deals with a
  primary contact, contacts with an email (and how many verified), the data room index
  (`list_project_notes`, `q: "Data room index"`) and its date.
- **Missing**: what launch needs and isn't there: no tiers, no lead candidates, deals with no
  primary contact or no email, no data room index or its known gaps, no target or currency on the
  round.
- **Next dates**: from the timeline (`list_milestones`): the next two or three milestones
  (`launch`, `outreach_start`, `calls_start`, …) with their status and days to go; overdue ones
  first. No milestones → say so and point at `/pipeline create` or `/round strategy` to set them.
- **Before launch**: at most five actions, each naming the command (`/investors contacts`,
  `/round strategy`, `/round data-room`, `/outreach draft`).

Ten lines or so. No buckets, likelihoods or scenarios.

## Output

Display: dashboard — a live Yellowtail Dashboard artifact for the project, made once and reused; cards inline (`${CLAUDE_PLUGIN_ROOT}/references/display.md` §4). Yellowtail's reads draw nothing on the way; this output is the one thing shown (§ Only the answer is drawn).

```
Seed 2026 — Fernway · raising · target $2.5m · 31 open, 3 won, 6 passed · read 9 Oct 2026

| USD          | Deals | Value   |
| In the bank  | 2     | $600k   |
| Commit       | 2     | $450k   |  ← bank + commit = $1.05m (42% of target)
| Best case    | 5     | $1.2m   |
| Pipeline     | 22    | $1.9m   |
Gap to target: $2.5m − $1.05m = $1.45m · coverage (best case + pipeline) ÷ gap = 2.1×
EUR (not counted against the USD target): Commit €150k (1) · Best case €300k (2)

Earliest realistic close: mid-Dec, set by Northwind (IC 12 Dec, comment 8 Oct)
Timeline: lead committed 28 Nov (upcoming) · legal close 19 Dec (upcoming): holds only if Northwind's IC does

Commit
- Northwind Capital · $300k · Partner meeting · high — IC booked 12 Dec (comment 8 Oct); next step
  "send model v3" due 10 Oct; two partners engaged
Best case
- Lantern Ventures · $250k · In conversation · medium — met 2 Oct; no next step set; single-threaded

Changed since 2 Oct: Halcyon → Passed (Jack, 6 Oct) · Ember In conversation → Partner meeting …

What if Northwind slips a month: bank + commit $1.05m unchanged, close moves to mid-Jan; no backfill
reaches IC before Christmas. What if Northwind passes: $750k (30%), gap $1.75m; backfill …
```

Link every deal: `/pipelines?pipeline={pipelineId}&deal={dealId}`. Say what was read and what
wasn't (calendar not connected, comments not read for Pipeline deals).

## Rules specific to this skill

- Read-only. A deal that belongs in another stage, a missing close date or next step: list it as a
  proposed `/pipeline update` change with its evidence; don't write it here.
- Never add currencies, never convert without a rate the user gave, never invent a value or a date.
- A stage label never outranks comments that contradict it; say which you went by.

## What comes next

- `/pipeline update` for the corrections the forecast surfaced; `/investors map` on a Commit deal
  that is single-threaded.
- `/round close` when the forecast close misses the timeline; `/round passes` if passes are piling
  up. Advisors running a client's raise: `/yt-raise:client-update` to send the founder the number.
