# /round pitch

The `pitch` action of `/round`. Arguments: `<deck upload or Drive link> [pipeline name] [--for <firm>]`.

A deck is read before it is presented, usually in a few minutes, often by an associate deciding
whether a partner should take the meeting. It must tell the story without the founder in the room:
every slide's headline states its point, and the stage's key question (pre-seed: why this team and
this problem; seed: early proof it works; Series A: a repeatable engine) is answered before slide
five. This skill reviews the deck as those readers would, and as the specific firms on the round
would, and rewrites what doesn't land.

Read `${CLAUDE_PLUGIN_ROOT}/references/house-rules.md` first. The slide-by-slide criteria are in
`${CLAUDE_PLUGIN_ROOT}/skills/round/references/slide-criteria.md`.

## Tools

| Tool | For | Required? |
| --- | --- | --- |
| Upload, or Docs (Google Drive `search_files`, `read_file_content` / `download_file_content`) | The deck | yes — one or the other |
| `whoami`, `get_project` | The startup's description and interest tags (stage, sector) | yes |
| `list_tags` | Names for the interest tag ids, so you know the stage being raised | yes |
| `list_pipelines`, `get_pipeline` | The round: target, stage tag or `attributes.round` | yes |
| `list_deals` | The firms on the round (open), to review against | recommended |
| `get_company` | Each firm's description, tags, attributes and recent signals: its thesis | for firm fit |
| `search_signals` | What those firms have said or done lately (`companyId`, last 180 days) | optional |

## Steps

1. **Get the deck.** An upload is complete input; read it all. A Drive link or name → read it from
   Drive. PDF or slides export: read text and, where possible, the visuals (charts, screenshots,
   logos). No deck → ask for it; never review from a description.
2. **Orient.** `whoami`, `get_project`, `list_pipelines` → the round. Work out the **stage** being
   raised (pipeline `attributes`, the project's stage interest tag, or the deck's own ask). Unclear →
   ask once; the criteria differ by stage.
3. **Know the audience.** `list_deals` on the round (or the firm in `--for`). For the 5–10 most
   advanced or Tier 1 firms, `get_company`: description (thesis), stage and industry tags,
   `leads_rounds`, cheque attributes, recent signals. Summarise each firm's lens in one line ("climate
   infra, leads seed, wants hardware-light"). No firms on the round → review against the stage only
   and say so.
4. **Map the deck.** List slides as they are: number, title, what it actually says. Note the order and
   which of the expected sections (criteria §1) are missing, merged or buried.
5. **Review slide by slide** against `${CLAUDE_PLUGIN_ROOT}/skills/round/references/slide-criteria.md`:
   - **Verdict**: strong / fix / rewrite / cut.
   - **What a reader takes away** in one line — compare with what the founder meant.
   - **The fix**: a rewritten headline (as a claim, not a label: "Contractors lose [n] hours a week to
     paperwork", not "Problem") and specific body changes. Use the founder's facts only; where the
     slide needs a number they haven't given, write `[metric: e.g. monthly revenue growth, last 6
     months]`, never a figure.
   - **Firm lens** where it matters: "Northwind's thesis is software-only; slide 7's hardware BOM will
     raise a flag — lead with the software margin."
6. **Check the whole.** Story order; the stage's key question answered early; consistency (the market
   number, the ask and the use of funds agree across slides and with the pipeline target); jargon;
   density (one idea per slide); a send-ahead version vs a presenting version.
7. **Report.**

## Output

Display: document — a private, Yellowtail-branded artifact; cards inline (`${CLAUDE_PLUGIN_ROOT}/references/display.md` §5). Yellowtail's reads draw nothing on the way; this output is the one thing shown (§ Only the answer is drawn).

```
Deck: Fernway seed deck v7 (14 slides) · Round: Seed 2026, $2.5m target · Stage lens: seed
Reviewed against: Northwind Capital, Ember Ventures, Halden Partners (thesis lines below)

Top five changes
1. …  (each names the slide and the rewrite)

Slide by slide
| # | Slide | Verdict | Reader takes away | Fix |
…

Missing: why now · use of funds
Inconsistent: market $40bn on slide 4, $12bn on slide 11; ask $3m vs pipeline target $2.5m
Firm notes: one line per firm — what will land, what will worry them
```

Rewrites for the slides marked rewrite go in full below the table. Offer to write the review into a
Drive doc beside the deck if Docs is connected (a new doc, no card needed). This skill writes nothing
to Yellowtail.

## Rules specific to this skill

- Never invent traction, market sizes, customer names or logos. A market size in the deck that has
  no source on the slide gets "add the source", not a replacement number.
- Criticise the slide, not the founder. Be direct; a soft review costs the round.
- A firm's thesis comes from its Yellowtail record or a sourced page, never from memory of the firm.

## What comes next

- `/meetings qa <firm>` for the questions the weak slides will provoke.
- `/outreach draft` once the deck is fixed; `/round strategy` if the review shows the ask is off.
