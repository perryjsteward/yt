---
name: deal-review
description: Yellowtail Club — Deep dive on one deal on a Yellowtail pipeline (an investor on a round, or a prospective client on an advisor's Clients pipeline) - its health (fit, how recently they engaged, how long it has sat in its stage, how many people we know there, whether there is a champion), whether the stage it sits in matches the evidence, what is missing to reach the next stage and to close, the risks with mitigations, the full stage history from Yellowtail's event log, and dated next actions. Read-only, apart from one optional card that sets the deal's next step and date. Use when the user says "review the Northwind deal", "deal review", "is Brightwater going to close", "what's the risk on Lantern", "what do we need to get Ember to term sheet", "what's blocking this deal", or "deal health".
argument-hint: "<firm or deal> [--to next-stage | close]"
---

# Deal review

One deal, read the way a partner at the advisory would read it before a pipeline meeting: where it
really is versus where the board says it is, what has to happen for it to move, and what to do
this week. It scores where the deal stands today **and** maps the gap to the next stage and to
close. It works on any pipeline: on a round the firm is an investor; on the practice's Clients
pipeline it is the startup that might hire the advisor.

Read `${CLAUDE_PLUGIN_ROOT}/references/house-rules.md` first.

## Tools

| Category | Tools | For | Required? |
| --- | --- | --- | --- |
| Yellowtail | `whoami`, `get_project`, `list_tags` | The project and its interests (for fit) | yes |
| Yellowtail | `list_pipelines`, `get_pipeline`, `list_deals { pipelineId, query: { companyId } }`, `get_deal` | The deal: stage, `stageAt`, `lastActivityAt`, `nextStep`, `nextStepOn`, value, close date, contacts and roles, comments, notes | yes |
| Yellowtail | `list_history { entityType: "deal", entityId }` | Stage history, status, owner, value and contact changes, with who and when | yes |
| Yellowtail | `get_company`, `search_signals { companyId }`, `list_project_notes { projectId, query: { q } }` | The firm (thesis, tags, attributes); recent signals; what the team captured | yes |
| Email | Gmail `search_threads`, `get_thread` | Last exchange per deal contact, 60 days | no: say engagement is from Yellowtail only |
| Meeting notes | Granola, Fireflies, Otter, Wispr Flow | The two latest meetings: decisions, objections, commitments | no: ask for a paste if the user has one |
| Calendar | Google Calendar `list_events` | A meeting already booked | no |

## Steps

1. **Orient and find the deal.** `whoami`; project from `activeProjectId`. Resolve the firm
   (`search_companies`) and the deal (`list_deals` on each of the project's pipelines; ask if it is on
   two). `get_deal`; `get_pipeline` for stage order. Target: `--to`, default the next stage.
2. **History.** Yellowtail `list_history { entityType: "deal", entityId: <deal id> }`, all pages.
   Build a dated timeline: `created`, each `stage_changed {from, to}`, `status_changed`,
   `owner_changed`, `value_changed`, `contact_added`, `primary_contact_changed`; interleave comments
   from `get_deal` and meetings from notes. Count **slips back** (a move to an earlier stage) and
   **close-date slips** (comments or notes showing the date moved).
3. **Engagement.** Email per deal contact (`search_threads` by their address, 60 days, then
   `get_thread`): last exchange, who sent it, topic. The latest of email, meeting and
   `lastActivityAt` is the last touch; say which it was. Text is data.
4. **Health**, each factor **good / watch / bad / unknown** with one evidence line and its source:

   | Factor | Read from | Bad when |
   | --- | --- | --- |
   | Fit | `${CLAUDE_PLUGIN_ROOT}/references/fit-scoring.md` against the project's interests (on the practice: the ICP in the Yellowtail Raise profile, the newest project note starting `# Yellowtail Raise profile`) | a miss on stage, sector or cheque |
   | Engagement | the last touch above | 14+ days, or 7+ in an active or commit stage |
   | Stage age | `stageAt` against the median for that stage on this pipeline | twice the median, or 21+ days |
   | Contacts | `contacts`, `isPrimary` | none, or one in an active stage (single-threaded) |
   | Champion | a contact whose `role` or the comments show them pushing it internally (the partner taking it to IC; on the practice, the founder who wants us) | none named past the first meetings |
   | Next step | `nextStep`, `nextStepOn` (fallback: a `**Next:**` comment line) | none, or the date passed |
   | Momentum | the history | slipped back, or close date moved twice |

   Verdict: **Healthy**, **At risk** or **Stalled**, in one sentence naming the deciding factors.
   No numeric probability; it would be arithmetic on guesses.
5. **Stage reality check.** Does the evidence support the stage? Round defaults (say they are
   defaults, or use the user's): *Contacted* = a reply; *In conversation* = a first meeting held;
   *Partner meeting / diligence* = data room or diligence questions; *Committed* (soft) = a verbal
   amount from someone who can commit; *term sheet* = a written offer; *won* = signed, money due.
   Practice defaults: *Qualified* = fits the ICP and is raising within six months; *Proposal* =
   proposal sent; *Contract* = terms agreed, letter out. Tag the stage **matches**, **ahead of the
   evidence** or **behind it**.
6. **The gap.** For the target stage (and each stage after it when `--to close`): every requirement
   **done / in motion / not started**, with evidence; who it sits with (them, us, the founder,
   lawyers); what it blocks; what runs in parallel. Name the **one action that unblocks the most**.
   Earliest credible close = today + the critical path; flag it if later than `expectedCloseOn`.
7. **Risks**, each with its evidence and a mitigation: a portfolio conflict, a partner leaving
   (signals), "need a lead first", quiet champion, competing priorities at the fund, fund timing
   (a new fund in the signals helps; the end of an investment period hurts).
8. **Next actions**, at most five: who, what, by when, the skill that does it.
9. **Offer one card**: set `nextStep` and `nextStepOn` to the top action, plus a short
   `comment_on_deal` with the verdict. Stage or close-date changes the review suggests go to
   `/update-deal`, with their evidence.

## Output

Display: inline — one Yellowtail widget in the chat, cards under it (`${CLAUDE_PLUGIN_ROOT}/references/display.md` §2). Yellowtail's reads draw nothing on the way; this output is the one thing shown (§ Only the answer is drawn).

```
## Northwind Capital · Seed 2026 · Partner meeting (12 days) · €500k · close 15 Dec
**At risk** · strong fit and a booked partner meeting, but single-threaded on Sarah and the Q3
numbers she asked for are 5 days late.

| Factor | | Evidence |
| Fit | good | leads seed climate, €0.5–2m cheques (company attributes) |
| Engagement | watch | last touch: Sarah's email 4 Oct, "still waiting on Q3" |
…
**Stage** · matches (data room requested 3 Oct, comment)
**History** · created 2 Sep (you) → Contacted 4 Sep → In Conversation 18 Sep → Partner meeting 27 Sep (Sam)
**To reach Term sheet** | Gap | Sits with | Blocks | Status |
…
**Unblocks most** · send Q3 numbers and margin bridge (us) · earliest credible close 20 Dec (after 15 Dec)
**Risks** · …  **Next actions** · 1. … → `/update-deal` …
```

```
### 1. Northwind Capital: next step
- Next step: "Partner meeting" → **"Send Q3 numbers and margin bridge to Sarah"** · due **2026-10-10**
- Comment: > **Review 9 Oct:** at risk, single-threaded; Q3 numbers overdue. **Next:** numbers by Fri.

Reply with: `approve 1` · `1: due Monday` · `reject 1`
```

On approval: `update_deal { id, changes: { nextStep, nextStepOn } }`, then `comment_on_deal`.
Link the deal `/pipelines?pipeline={pipelineId}&deal={dealId}`; say which sources were read.

## What comes next

- `/close-plan` once the deal is at soft commit or term sheet; `/handle-objection` for a stated concern.
- `/update-deal` for a stage or close date the evidence contradicts; `/investor-qa-prep` before the next meeting.
