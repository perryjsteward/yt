---
name: portfolio-review
description: Yellowtail Club — The advisor's view across every client in the workspace — for each client project, the round's status, target against money in and committed, this week's momentum from the event log (stage moves, wins, passes, new deals), stalled deals, the timeline at risk (overdue or due-soon milestones on each round), risks (no lead, a commit gone quiet, a close date passing, one fund carrying the round), and where the advisor's time should go this week, ranked. Read-only. Use when an advisor says "portfolio review", "how are all my clients doing", "which rounds need me this week", "Monday review", "across all projects", "where should I spend my time", or at the start of the week.
argument-hint: "[workspace name] [--since 2026-10-02] [--include-practice]"
---

# /portfolio-review

An advisor runs several raises at once, and the one that needs them is rarely the one shouting.
This skill reads every client's round side by side — the same numbers, the same flags — and ends
with a ranked list of where this week's hours go. It is the multi-client version of
`/pipeline-review`, at one line per deal that matters rather than every deal.

Read `${CLAUDE_PLUGIN_ROOT}/references/house-rules.md` first.

## Tools

| Tool | For | Required? |
| --- | --- | --- |
| `whoami` | Workspaces and every project in them (`projects[].workspaceId`) | yes |
| `list_pipelines`, `get_pipeline` | Each project's rounds: status, `target` and `currency`, stages, `openValues` | yes |
| `list_milestones` | Each round's timeline: overdue and due-soon milestones, `legal_close` | yes |
| `list_deals` | Per raising pipeline: open, then `status: "won"` and `"lost"`; page `nextCursor` | yes |
| `get_deal` | Commit-phase and stalled deals only: comments, `nextStep`, `nextStepOn` | for those |
| `list_history` | `{ projectId }` per project, newest first, paged back to `--since`: the week's movement | yes |
| `list_project_members` | Who the founder guests are (for the "needs from founder" line) | optional |
| Calendar (Google Calendar `list_events`) | Investor meetings this week per client | optional |

## Steps

1. **Scope.** `whoami`. The workspace named, else the one with the most projects where the user is
   a member (ask if two are close). Every project in it is a client, except a project named
   "Yellowtail Raise" (the advisor's own pipeline) unless `--include-practice`. Say "N clients".
   (`whoami` is the list of projects you can reach.)
2. **The rounds.** `list_pipelines` per project. A client with no `raising` pipeline is listed in
   one line by status (planning: "not launched"; closed: "closed") and not read further.
3. **Read each raising round.** All deals (open, won, lost); counts.
4. **Per client, the numbers** — per currency, never added across currencies:
   - target (`target`, in the round's `currency`) · **in** (won) · **committed** (soft-commit
     stages; "Committed" counts as soft unless won or comments show signed documents) · **active**
     value · % of target from won + committed deals in the round's currency; other currencies listed
     beside it, unconverted.
   - **Lead**: a commit-phase deal whose comments or notes name it as lead, or "no lead".
5. **Momentum.** `list_history { projectId }` back to `--since` (default 7 days): `stage_changed`
   forward vs backward (by stage order), `status_changed` to won / lost, `created` deals,
   `value_changed`. One word per client — *moving*, *steady*, *stalled* — with the counts behind
   it. Who did the work (`actorName`) when it helps ("the founder moved 4 deals").
6. **Stalled and at-risk deals** (at most 3 per client, the ones that matter most by value and
   stage): commit or active deals with `lastActivityAt` over 14 days, `nextStepOn` passed,
   `expectedCloseOn` passed; one-line evidence each.
6b. **Timeline at risk.** `list_milestones` per raising round (and per planning round, where
   `launch` may be close): milestones `overdue` or `due_soon`, with days over or to go and their
   notes. Any overdue milestone puts that client near the top of step 8, and becomes its first
   action.
7. **Risks per client**: no lead past the first month of raising; a soft commit quiet for two
   weeks; one deal over 50% of committed; a close date in the next 30 days with the gap still
   open; zero new deals in two weeks with coverage under 2×.
8. **Where your time goes this week.** Rank the clients by need (risk × value at stake × how soon),
   then give at most seven actions across the whole workspace, each tied to a client and a deal:
   "Fernway: get Northwind's IC date — it decides whether the round closes before Christmas".
   Include what to ask the founder where a risk is theirs to fix (deck, numbers, intros).

## Output

Display: dashboard — a live Yellowtail Dashboard artifact for the project, made once and reused; cards inline (`${CLAUDE_PLUGIN_ROOT}/references/display.md` §4). Yellowtail's reads draw nothing on the way; this output is the one thing shown (§ Only the answer is drawn).

```
Portfolio — Atlas Advisory · 5 clients, 4 raising · week to 9 Oct 2026

| Client | Round | Target | In | Committed | % | Lead | Momentum | Timeline | Risk |
| Fernway | Seed 2026 | $2.5m | $600k | $450k | 42% | Northwind | moving (5 fwd, 1 pass) | legal close 12 Dec, on track | — |
| Voltframe | Pre-seed | £1.2m | £0 | £150k | 13% | none | stalled (0 moves) | ⚠ lead term sheet overdue 9 days | no lead, week 7 |
| Lumen | Seed | $3m | $1.1m | €400k + $700k | 60% (USD) | Halcyon | steady | lead committed in 5 days | 1 fund = 61% |
| Orbit | Series A | — | planning: not launched | launch 3 Nov |

Voltframe — needs you most
- No lead in week 7; 0 stage moves this week; 3 active deals quiet > 14 days
- Ember Capital · Partner meeting · last activity 19 Sep · next step "IC date" due 30 Sep, passed
…

This week
1. Voltframe: …
```

Link each client's pipeline `/pipelines?pipeline={pipelineId}` and deals
`/pipelines?pipeline={pipelineId}&deal={dealId}`. Say what was read and what wasn't (calendar not
connected; comments read only for commit and stalled deals).

## Rules specific to this skill

- Read-only. Fixes go to `/update-deal`, chasing to `/draft-outreach`; never write from here.
- Same measures for every client, so they compare. Never add or convert currencies.
- A client the user can't see deals for (a project they're not a member of) is listed as "no
  access", not skipped silently.

## What comes next

- `/pipeline-review` or `/round-forecast` on the client that needs you most.
- `/close-plan` for a client whose timeline is overdue; `/daily-briefing` for today's part of the
  plan. The founder reports: `/yt-raise:client-update` for each client.
