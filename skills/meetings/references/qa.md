# /meetings qa

The `qa` action of `/meetings`. Arguments: `[firm] [--stage first-meeting|partner-meeting|diligence]`.

Investors ask the questions their own fund's situation makes inevitable: a climate fund with a
hardware portfolio asks about capital intensity, a fund with a competitor in its portfolio asks
how you're different, a Series A partner asks about retention and channel repeatability whatever
the deck says. A founder who has rehearsed those answers, with numbers they can stand behind,
reads as in control. This skill predicts the questions, drafts answers from the founder's own
material, and is blunt about the answers that aren't there yet.

Read `${CLAUDE_PLUGIN_ROOT}/references/house-rules.md` first. Answers come from the data room index:
`${CLAUDE_PLUGIN_ROOT}/references/data-room-index.md`.

## Tools

| Tool | For | Required? |
| --- | --- | --- |
| `whoami`, `get_project` | Active project, the startup's description, interest tags (stage, sector) | yes |
| `list_tags` | Names for those tag ids | yes |
| `search_companies`, `get_company` | The firm: thesis (description), tags, attributes (`leads_rounds`, cheque, fund size, notable investments), people, recent signals | with a firm |
| `search_signals` | The firm's and partner's recent statements and deals (`companyId`, `personId`, last 12 months) | with a firm |
| `list_pipelines`, `list_deals` (`companyId`), `get_deal` | The deal's stage, contacts and comment thread — questions already asked, concerns already raised | when a deal exists |
| `list_project_notes`, `get_note` | The **data room index** first (`query: { q: "Data room index" }`, body starts `# Data room index`); then what the team captured from earlier conversations | yes |
| Upload / Docs (Drive read) | Deck, model, metrics, data room docs — the source of every answer | strongly preferred |
| Email (Gmail `search_threads`, `get_thread`) | Threads with the firm: questions asked by email | optional |
| Web | Portfolio companies and the partner's public writing, sourced | optional |

## Steps

1. **Orient.** `whoami`, `get_project`, the round from `list_pipelines`. Work out the stage being
   raised and the meeting type (`--stage`, or the deal's stage name, or ask).
2. **Gather the founder's material.** The **data room index first** (`data-room-index.md` § Finding
   it): its documents, key metrics and known gaps are the base of every answer. Then every upload,
   and the documents the index links when an answer needs more than its summary
   (`read_file_content`). No index → say so, offer `/round data-room`, and search Drive for the deck,
   model and metrics if Docs is connected. `list_project_notes` for notes on the firm. No material
   at all → ask for the deck (and model if Series A); answers without material become "to draft
   with you", not invented ones.
3. **Know the firm** (skip for a generic stage prep). `get_company`; `search_signals` for the firm
   and the partner on the deal. Portfolio: the `notable_investments` attribute, the firm's signals,
   and its own portfolio page on the web (cite it). Flag **portfolio companies that compete with or
   sit next to the startup**. What has the firm said it looks for, or worries about?
4. **Read the deal.** `get_deal`: comments and notes from earlier meetings, questions asked, concerns
   logged, and the next step (`nextStep`, `nextStepOn`; on an old deal, the newest `**Next:**` comment
   line): a promised follow-up still open is a question waiting to be asked. Email threads with the firm if connected. A question already asked and answered weakly is
   the most likely one to come back.
5. **Build the questions**, 15–25, grouped: Team & founder-market fit · Problem & customer · Product &
   defensibility · Market · Traction & metrics · Business model & unit economics · Go-to-market ·
   Competition · Round, valuation & use of funds · Risks. Each question carries **why this firm asks
   it** (thesis, portfolio, signal, earlier comment, or stage norm — cited).
   Stage norms to cover at minimum: pre-seed — why you, why now, what you've learned from customers;
   seed — evidence it works, first channel, what this round proves; Series A — growth and retention by
   cohort, CAC and payback, sales motion repeatability, the plan to the next round.
6. **Draft answers** only from the index and the client's files, in the founder's voice, 2–4
   sentences each, citing the document in brackets (`[Deck v9, s.6]`, `[Model v4, Revenue tab]`,
   `[note 3 Oct]`). A number appears only if a source shows it. No source → write the answer's shape
   and `[needs: …]`; a gap the index already lists says "known gap".
7. **Flag weak spots.** For each: the question, why the current answer is weak (missing data,
   contradicts the deck, a known red flag for this firm), and what to prepare (a metric, a customer
   reference, a slide). Rank them; the top three go first in the output.

## Output

Display: document — a private, Yellowtail-branded artifact; cards inline (`${CLAUDE_PLUGIN_ROOT}/references/display.md` §5). Yellowtail's reads draw nothing on the way; this output is the one thing shown (§ Only the answer is drawn).

```
Prep: Northwind Capital · partner meeting · Seed 2026 (deal stage: In Conversation)
Firm lens: leads seed climate software; portfolio includes Coldline (adjacent, not competing) [signal 14 Sep]

Top weak spots
1. Net revenue retention — asked about at Northwind's last two seed deals [signals]; no cohort data in the
   material. Prepare: cohort table from billing, even if 3 cohorts.
…

Questions and draft answers
### Traction & metrics
Q. What's your month-on-month revenue growth over the last six months?
   Why they ask: stage norm; Northwind's thesis note "capital-efficient growth" [firm description]
   A. <draft> [model: Revenue tab]
…
```

This skill writes nothing to Yellowtail. Offer to save the prep as a Google Doc (a card, then
Drive `create_file` with `contentMimeType: "text/html"`), and — on request only — to log a one-line comment on the deal ("Q&A prep done for partner meeting;
weak spots: retention, CAC") as a card, applied with `comment_on_deal` on approve.

## Rules specific to this skill

- Every draft answer is the founder's to own; mark anything you inferred rather than read.
- Don't pad with generic questions: each one cites why *this* firm or *this* stage asks it.
- Never characterise a partner's views from memory; only from a signal, the firm record or a cited page.

## What comes next

- `/meetings prep` the day before the meeting (attendees, threads, deal history).
- `/round pitch` if the weak spots trace back to the deck.
