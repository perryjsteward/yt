---
name: investor-update
description: Yellowtail Club — Write the founder's monthly investor update — highlights, key metrics, lowlights, what's next, and specific asks — for existing investors and for investors in the pipeline. Metrics come only from what the founder provides, never invented. Recipients come from Yellowtail (deal contacts at the stages the user picks on a round, or the people on a list), with each address's verification status shown. Creates Gmail drafts only, after a card. Use when someone says "investor update", "monthly update", "update my investors", "write the October update", "send an update to the pipeline", or "keep warm investors in the loop".
argument-hint: "[month] [--to existing|pipeline|both] [--list <name>] [--stages \"In Conversation,Committed\"]"
---

# /investor-update

The monthly update is the cheapest fundraising tool there is. Existing investors who hear regularly
help more (intros, hires, follow-on); pipeline investors who watch a company hit what it said it would
hit come to the next round already convinced. The format that works is short and consistent: the
same metrics every month, the bad news alongside the good, and asks specific enough that a reader can
act on one in a minute. This skill writes it from the founder's own numbers and addresses it from
Yellowtail.

Read `${CLAUDE_PLUGIN_ROOT}/references/house-rules.md` first.

## Tools

| Tool | For | Required? |
| --- | --- | --- |
| `whoami`, `get_project` | Active project, the startup's name and description | yes |
| `list_pipelines`, `get_pipeline`, `list_deals` | Pipeline recipients: deals at the chosen stages (and `status: "won"` for investors already in) | for pipeline recipients |
| `get_deal` | The deal's contacts (`isPrimary`) | for pipeline recipients |
| `list_lists`, `get_list` | List recipients (e.g. "Existing investors") — a list holds people | for list recipients |
| `get_person` | Each recipient's `email` and `emailStatus` | yes |
| Upload / Docs (Drive read) | Last month's update, the metrics sheet, board pack | strongly preferred |
| Email (Gmail `search_threads`, `get_thread`, `create_draft`) | Last update sent (format and metrics to repeat); the drafts | optional; paste-ready text otherwise |

## Steps

1. **Orient.** `whoami`, `get_project`.
2. **Find last month's update** (Gmail sent mail, Drive, or ask for a paste). Reuse its structure and
   **its metric set**, so readers can compare month to month. First update → propose a small metric
   set for the stage and let the founder pick.
3. **Collect this month's content from the founder**, in one ask: the numbers for each metric (with
   last month's for comparison if not in the old update), highlights, lowlights, what's next, asks
   (intros to specific kinds of people, hires, customers), and cash/runway if they share it. Read any
   upload first. **Never fill a metric yourself**: a missing one stays `[metric: …]` in the draft, and
   is named in the report.
4. **Pick recipients** (`--to`, or ask):
   - **Existing investors**: a list the user names (`get_list` → each entry's person) or the won deals on
     a closed round's pipeline (`list_deals` `status: "won"` → `get_deal` contacts).
   - **Pipeline investors**: `list_deals` on the round filtered to the stages the user picks (by
     `stageId` from `get_pipeline`; one call per stage, since the filter takes one) → `get_deal` → primary contact (all contacts only if asked).
   - `get_person` for each: no email → listed under "No email", not dropped silently; `unverified`
     email → shown with that status.
   Dedupe people who appear twice.
5. **Two versions when both audiences get it.** Existing investors get the full update. Pipeline
   investors get a shorter version: highlights, the metrics the founder is happy to share outside the
   cap table, what's next, and a light ask. Ask the founder which metrics are existing-investors-only;
   default to leaving cash and runway out of the pipeline version.
6. **Write it** in the founder's voice: subject `<Company> — <Month YYYY> update`; a three-line TL;DR;
   Highlights; Metrics (a small table, this month vs last, with definitions the first time); Lowlights
   (honest, with what's being done); What's next; Asks (numbered, specific: "Intro to heads of ops at
   mid-size logistics firms in the Nordics", not "intros welcome"); thanks. Under ~400 words.
7. **Card, then drafts.** Default delivery: one draft per version, addressed to the founder with
   recipients in **BCC**, so no one sees the list. Individual drafts with a personal first line → only
   if asked (batch at most 10 per round, each line from that investor's deal or signals, never filler).

## Card format

```
1. Draft — "Fernway — October 2026 update" (existing investors, full version)
   To: sam@fernway.io  BCC (6): Maya Okafor <maya@northwind.vc> verified · …
   No email (1): Lena Brandt (Ember Ventures)
   Metrics missing: [metric: gross margin]
   <the full email>

2. Draft — "Fernway — October 2026 update" (pipeline, short version)
   To: sam@fernway.io  BCC (11): … from Seed 2026 stages In Conversation, Committed
   <the full email>

Reply: approve all · 2: <feedback> · reject 2
```

On approve: Gmail `create_draft` per card; return the links. Nothing is sent.

## Output

Display: document — a private, Yellowtail-branded artifact; cards inline (`${CLAUDE_PLUGIN_ROOT}/references/display.md` §5). Yellowtail's reads draw nothing on the way; this output is the one thing shown (§ Only the answer is drawn).

Draft links (or paste-ready text), recipient counts per version, people with no email, metrics left as
placeholders. This skill writes nothing to Yellowtail. Offer, as a card, a one-line `comment_on_deal`
on each pipeline deal that received it ("October update sent") — only after the user says it was sent.

## What comes next

- `/pipeline-review` to see who engaged; `/draft-outreach` for a personal follow-up to anyone who replied.
