# /round strategy

The `strategy` action of `/round`. Arguments: `[pipeline name] [--target 2.5m EUR] [--legal-close 2027-03-31]`.

A round is won or lost before the first email. Good advisors decide the size from the plan, not
from what sounds normal; find a lead before courting followers; work a tiered list in a short,
deliberate window so meetings cluster and momentum is real; and have the data room ready before
anyone asks for it. This skill turns the project's own facts and Yellowtail's catalog into that
plan, and writes it down where the founder and the advisor both work from it.

Read `${CLAUDE_PLUGIN_ROOT}/references/house-rules.md` first. The playbook behind every section is
`${CLAUDE_PLUGIN_ROOT}/skills/round/references/round-playbook.md` (in this skill's folder); read it before step 4.

The round's dates live on its **timeline** (`list_milestones`): one milestone per key, shown on
the pipeline page, read by every other command. The plan proposes them and quotes them; it never
keeps its own set of dates in prose.

## Tools

| Tool | For | Required? |
| --- | --- | --- |
| `whoami` | Active project and workspace | yes |
| `get_project` | The startup's description and its interest tag ids (stage, industry, geography) | yes |
| `list_tags` | Names for those interest ids; investor-side tag ids (`stage`, `investor_type`) for filtering | yes |
| `list_pipelines`, `get_pipeline` | The round: `target` and `currency`, status, stages, `attributes` | yes |
| `list_milestones` | The round's timeline: dated milestones with `status` and `note` | yes, when there is a round |
| `list_deals` | Firms already on the round (`status` omitted = open only) | yes |
| `get_company_kind` (`investor`) | Real custom-field keys (e.g. `leads_rounds`, `min_check`, `max_check`, `fund_size`) | yes |
| `list_companies` | The project's slice of the catalog, by `tags` facets and `attr` (e.g. `attr.leads_rounds=true`) | yes |
| `get_company` | One firm's tags, attributes, people and recent signals, for tiering | for the shortlist |
| `search_signals` | Recent activity (`projectId`, `companyId`, `from`) — a firm that just raised a fund or led a comparable round moves up | recommended |
| `list_project_notes` | What the team has already captured about investors | optional |
| `update_pipeline` | Record the plan on the round (description, `target`, `currency`, `attributes`) — card first | optional |
| `create_milestone`, `update_milestone` | Put the plan's dates on the timeline; move one with the reason as `note` — card first | optional |
| `capture_note` | A project note for an open decision about a date, next to the milestone's note — card first | optional |
| Docs (Google Drive `create_file`) | The plan document | optional; markdown otherwise |
| Upload / Drive read | Deck, financial model, cap table, last investor update | optional, strongly preferred |

## Steps

1. **Orient.** `whoami`, then `get_project` for the active project. `list_pipelines`; if the project
   has several, use the one the user named or ask. No pipeline yet → plan anyway and offer
   `/pipeline create` at the end.
2. **Gather the facts the plan stands on.** Ask once, in one message, for whatever the material
   doesn't give: monthly net burn, cash in bank, current revenue or the traction metric that
   matters, the milestones the next round will be judged on, the last round (instrument, amount,
   valuation or cap), where the company is incorporated, and any date pressure (and whether "close
   by" means lead committed or money in). Read an uploaded deck, model or cap table first and
   ask only for what is still missing. **Never fill a figure yourself**; an unknown stays
   `[to confirm]` in the plan.
3. **Read the round.** `get_pipeline` (`target`, `currency`, status, stages, attributes),
   `list_milestones` (the dates already set, with their notes) and `list_deals` (and again with
   `status: "won"` and `status: "lost"` if the round is already running), so the plan starts from
   where the round is, not from zero.
4. **Feasibility first: date it backwards** (playbook §4, "Dating it backwards"). Before sizing
   anything, take the date that matters (the founder's, or `legal_close` / `lead_committed` on the
   timeline) and work back: legal lead time for this jurisdiction and instrument (a Dutch B.V.
   priced round needs a notarial deed, a German GmbH round is notarised, a UK SAFE needs neither),
   term sheet to signing, partner meetings, the outreach window, prep. Say at the top of the plan
   whether the date holds, and if not, the realistic date and what would have to change. A date
   that waits on the founder's decision is an **open decision**, said plainly.
5. **Size it** (playbook §1). Show the arithmetic: months of runway to reach the milestones ×
   monthly burn (with the hiring plan if given) + buffer = the raise. Give a range, the
   milestones it buys, and what a smaller amount would and wouldn't reach, in the round's own
   currency (never converted). If the pipeline already has a `target`, compare and say which you'd
   change.
6. **Instrument and valuation context** (playbook §2). Recommend priced round, SAFE or
   convertible note for this stage and geography, with the trade-off in a sentence each. For
   valuation, use only what the user gives or a source you name (link and date). No market
   medians from memory; if there is no source, say "set with your lead; benchmark from a named
   dataset" and list which ones.
7. **Build the tiered target list** (playbook §3). Start with firms already on the pipeline, then
   `list_companies` with `projectId` and the investor facets that match the round (stage tag,
   `attr.leads_rounds=true` for lead candidates), then `search_signals` for the last 90 days.
   `get_company` the strongest 15–25. Say fit as a band with reasons, per
   `${CLAUDE_PLUGIN_ROOT}/references/fit-scoring.md`, then tier each firm:
   - **Tier 1 — lead candidates**: leads at this stage, cheque fits the round, thesis and sector
     fit, recent activity. 8–15 firms.
   - **Tier 2 — strong fit, likely follow or co-lead**. 15–30 firms.
   - **Tier 3 — fill**: angels, operators, smaller funds, strategic money; for once a lead is in.
   For every firm: why it fits, likely role (lead / follow), the evidence (tag, attribute, signal
   date), and a **portfolio-conflict check** against the startup's competitors when the user has
   named them. Mark what Yellowtail doesn't know as unknown, not as a fit. Firms not in
   Yellowtail are named only if the user brought them; to find more, offer `/investors find`.
8. **Timeline** (playbook §4): prep (deck, model, data room, references, warm-intro paths),
   a pre-marketing window, a concentrated outreach window (Tier 1 first, Tier 2 a week or two
   behind), meetings, partner meetings, term sheet, diligence and legal, close, dated from step 4.
   Each dated step that has a milestone key (`launch`, `outreach_start`, `calls_start`,
   `lead_term_sheet`, `lead_committed`, `legal_close`) becomes a proposed milestone on the card;
   the plan quotes those dates rather than keeping its own. An open decision becomes the
   milestone's `note` ("open decision: 31 Dec is lead committed or legal close; Hayden to decide")
   and a project note, not only a sentence in the doc.
9. **Readiness checklist** (playbook §5): each item **ready / in progress / missing**, from the
   material read. Missing items become the founder's to-do list.
10. **Write the plan.** Sections: Summary (the raise, the instrument, the window, the first ten
   firms) · Sizing · Instrument and valuation · Sequencing · Target list (tier tables) ·
   Timeline (quoting the milestones, each with its key) · Readiness · Risks and open questions ·
   Sources. Docs connected → create it in
   Drive titled `<Project> — <Round> strategy (<date>)` and return the link; otherwise output
   markdown. Creating a new document needs no card; editing someone's existing doc does.
11. **Offer to record it: the pipeline and its timeline.** One card (format below). Apply on approve.

## Card format

```
1. Update pipeline "Seed 2026" — project Fernway (workspace Northlight Advisory)
   target:      2000000 → 2500000 EUR      (sizing §, your burn €95k/mo × 22 months + buffer)
   attributes:  + round: "seed"  + instrument: "priced"
                (sent as the whole object: today's keys from get_pipeline, plus these)
   description: "Seed, €2.0–2.5m priced, lead first. Plan: <Drive link>"

2. Timeline for "Seed 2026" (the Timeline panel on the pipeline page; Fernway's founders see it)
   + launch           2027-01-11
   + lead_term_sheet  2027-02-26
   + lead_committed   2027-03-12
   ~ legal_close      2027-03-31 → 2027-04-30   note "Dutch B.V. priced: notarial deed and
                      investor KYC need ~7 weeks after the term sheet (feasibility §)"
   + calls_start      2027-01-25   note "open decision: Tier 2 a week behind Tier 1, or
                      together? Priya to decide"

3. Project note "Open decision: Tier 2 timing" — the note above, for the founder to answer

Reply: approve all · 2: <feedback> · reject 3
```

`update_pipeline` **replaces** `attributes` whole: send today's keys plus the new ones, never just
the new ones, and only the other fields that change. `target` is whole units of the round's
`currency`; never convert one currency into another, and never put a target, currency or date in
`attributes` (`targetUsd`, `targetCloseDate`, `close_by` are gone). On the timeline card, `+` is
`create_milestone` (`{ pipelineId, milestone: { key, dueOn, label?, note? } }`) and `~` is
`update_milestone` (`{ id, changes: { dueOn, note } }`) with the reason as the note, so the history
keeps from → to. A custom key needs a `label`.

## Output

Display: document — a private, Yellowtail-branded artifact; cards inline (`${CLAUDE_PLUGIN_ROOT}/references/display.md` §5). Yellowtail's reads draw nothing on the way; this output is the one thing shown (§ Only the answer is drawn).

The plan link (or markdown), then a six-line summary in chat: whether the dates hold (the
feasibility check), the raise, the instrument, the window, the first five Tier 1 firms, the top three
readiness gaps. Then the cards: the pipeline, the timeline, any open-decision note.

## What comes next

- `/investors find` to widen Tier 1 or 2 beyond what Yellowtail holds.
- `/pipeline add` for tiered firms not yet on the round.
- `/round pitch` on the deck, and `/outreach draft` for Tier 1 once the deck is ready.
