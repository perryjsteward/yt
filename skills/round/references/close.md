# Close plan

The `close` action of `/round`. Arguments: `[pipeline or deal] [--legal-close 2026-12-15]`.

A round is not closed when the lead says yes. Between the term sheet and the money there are
weeks of steps on both sides, and every one that nobody owns slips. This skill writes the plan
backwards from the round's **legal close** milestone (money in) and checks it against **lead
committed**, names an owner on our side and theirs for every step, and
tracks each investor in the round separately, because the lead's lawyers and an angel's wire are
different problems. It works on any pipeline: a round, or the practice's Clients pipeline, where
the close is a signed engagement letter.

Read `${CLAUDE_PLUGIN_ROOT}/references/house-rules.md` first. This is a plan, not legal advice; the company's
lawyers own the documents (`/round term-sheet` explains the terms).

## Tools

| Category | Tools | For | Required? |
| --- | --- | --- | --- |
| Yellowtail | `whoami`, `get_project`, `list_pipelines`, `get_pipeline` | The round: stages, `target` and `currency`, attributes (instrument, round) | yes |
| Yellowtail | `list_milestones { pipelineId }` | The round's timeline: `lead_term_sheet`, `lead_committed`, `legal_close` with `status`, `daysUntilDue` and `note` | yes |
| Yellowtail | `update_milestone { id, changes: { dueOn, note } }`, `create_milestone` | Move a date the plan shows is unrealistic, with the reason; add `legal_close` if the round has none | on approval |
| Yellowtail | `list_deals` (open and `status: "won"`), `get_deal` | Deals in a soft-commit stage or later: value, currency, contacts, comments, `nextStep`, `nextStepOn`, `expectedCloseOn` | yes |
| Yellowtail | `list_history { entityType: "deal", entityId }` | When each deal reached its stage; slips | yes |
| Yellowtail | `list_project_notes { projectId, query: { q } }`, `get_note` | Terms and closing details the team captured; on the practice, the Yellowtail Raise profile | recommended |
| Yellowtail | `update_deal { id, changes: { nextStep, nextStepOn, expectedCloseOn } }`, `comment_on_deal` | Per-deal next step and the plan's link | on approval |
| Docs | Drive search/read, `create_file` | The term sheet and documents already drafted; the plan doc | no: markdown |
| Email | Gmail `search_threads`, `get_thread`, `create_draft` | Commitments in writing; open legal threads; the note sharing the plan | no: paste-ready text |

## Steps

1. **Orient.** `whoami`; the round (`list_pipelines`; ask if several are raising); `get_pipeline`.
   Scope: the whole round (default) or one deal the user names.
2. **Who is in.** `list_deals`, then keep deals in a soft-commit stage or later (classified as
   `/pipeline review` does) and won deals. For each, `get_deal` and its events: amount and currency,
   lead or follow (from comments or notes; unknown stays unknown), the signatory contact, what is
   already done (term sheet signed, side letter asked for, KYC sent) with its source. Total
   committed against the round's `target`, in its `currency`; deals in another currency are
   totalled separately, never converted.
3. **The target date: the timeline.** `list_milestones` → `legal_close` (money in) is what the plan
   works back from, and `lead_committed` / `lead_term_sheet` are checked on the way. Quote them with
   their notes; an open decision in a note ("open decision: …") is shown at the top. No
   `legal_close` milestone → `--legal-close`, else the lead's `expectedCloseOn`, else ask, and propose
   creating the milestone on the cards. Never invent one, and never treat "close by" as one date:
   lead committed and legal close are separate.
4. **The steps.** Start from this default round sequence, keep what applies to the instrument
   (a SAFE or ASA skips most of it) and the user's or the term sheet's own steps; say it is a default:
   1. Term sheet signed (lead; company) · 2. Confirmatory diligence: data room, legal, financial,
   references (lead's team; founder) · 3. Long-form documents drafted and negotiated: subscription or
   investment agreement, articles, shareholders' agreement, disclosure letter (lead's counsel;
   company counsel) · 4. Follow-on investors confirm and receive documents (each investor; advisor)
   · 5. Board approval and shareholder consents, waivers of pre-emption (company; existing
   investors) · 6. KYC and investor details (each investor; company secretary) · 7. Signatures
   (everyone) · 8. Funds wired and received (each investor; founder confirms) · 9. Filings and share
   issue (company counsel) · 10. Closing announcement (founder; lead, agreed embargo).
   **Practice close:** proposal accepted → engagement letter from the advisor's template
   (`/create-contract`) → founder's comments → signed → first invoice → `/onboard-client`.
5. **Date it backwards.** Each step gets an owner on each side (a named person from the deal's
   contacts or the project's members, else "their counsel", never an invented name) and a date,
   working back from the target using turnaround the user gives (default assumptions shown: legal
   drafting 2 weeks, consents 1 week, wires 3 working days; a notarised round, such as a Dutch B.V.
   or German GmbH priced round, adds notary scheduling and investor KYC). Flag any step whose date is
   already past or that makes the target impossible, and give the earliest credible close.
   **When the plan says a milestone is unrealistic** (legal close before the earliest credible
   close, lead committed after the documents would have to be agreed), propose an
   `update_milestone` card moving it, with the reason as the `note` ("legal drafting 3 weeks after a
   term sheet on 17 Oct; notary 2 weeks"). The plan doc quotes the timeline; it never keeps a
   different date of its own.
6. **Per investor.** A table: investor, amount, role, what is done, what is outstanding, the next
   step and its owner and date. The next outstanding step becomes that deal's `nextStep` and
   `nextStepOn`; a target close different from `expectedCloseOn` is proposed too.
7. **Cards** (below), then write on approval: the Drive doc first (`create_file`, titled
   `<Project> — <Round> close plan (<date>)`), then any milestone change, then per deal
   `update_deal` and a `comment_on_deal` linking the doc. A deal's `expectedCloseOn` is never later
   than the round's `legal_close`. Optionally a draft to the lead (thread participants only) sharing the plan and
   asking them to confirm owners on their side.

## Cards

```
**Close plan** · Acme Robotics · Seed 2026 · legal close **2026-12-12** (timeline) · lead committed 2026-11-07 · $1.9m of $2.5m committed

### 1. Plan doc → Drive "Acme Robotics — Seed 2026 close plan (9 Oct)"
| # | Step | Ours | Theirs | By | Status |
| 1 | Term sheet signed | Priya (CEO) | Sarah Lee, Northwind | 17 Oct | in motion (redline 8 Oct) |
| 3 | Long-form documents | company counsel | Northwind's counsel | 21 Nov | not started |
…
Earliest credible close: 12 Dec (holds if the term sheet is signed by 17 Oct)

### 2. Timeline: lead committed 2026-11-07 → 2026-11-14
- note "term sheet redline still open on 9 Oct; long-form can't start before 17 Oct (close plan)"

### 3. Northwind Capital (lead, $1.0m)
- Next step: "Term sheet redline" → **"Sign term sheet"** · due **2026-10-17**
- Comment: > **Close plan (9 Oct):** <doc link>. Outstanding: term sheet, diligence, long-form.

### 4. Brightwater (follow, $400k)
- Next step: none → **"Confirm allocation and send KYC details"** · due **2026-11-07**

**Needs your input** · A: who is the company's counsel? · B: Lantern's $250k: soft or confirmed?

Reply with: `approve all` · `approve 1 3` · `3: due 20 Oct` · `A: Fieldfisher` · `reject 2`
```

## Output

Display: dashboard — a live Yellowtail Dashboard artifact for the project, made once and reused; cards inline (`${CLAUDE_PLUGIN_ROOT}/references/display.md` §4). Yellowtail's reads draw nothing on the way; this output is the one thing shown (§ Only the answer is drawn).

One line per write with the doc link and `/pipelines?pipeline={pipelineId}&deal={dealId}`. Then
the steps due in the next seven days, owners named, and what was not read (no term sheet found,
email not connected).

## What comes next

- `/daily follow-ups` to chase the outstanding items; `/round term-sheet` for the terms themselves.
- On the practice: `/create-contract`, then `/onboard-client` once it is signed.
