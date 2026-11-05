# Find investors

The `find` action of `/investors`. Arguments: `[count, default 20] [--snapshot] [--yellowtail-only | --web-only] [extra criteria, e.g. "leads only", "UK only"]`.

The core command. One run answers "who should this round talk to?": Yellowtail first, because it is
curated and already scoped to the project, then the web for firms Yellowtail is missing. The output
is **one** ranked shortlist, not two lists to reconcile.

**It files the new firms it keeps, and writes nothing else** (Perry, 17 Oct 2026). Every investor the
web turns up that Yellowtail lacks and the shortlist keeps is filed to the shared catalog during the
run, with no card (step 7), so the catalog grows with every search anyone makes and the next run
finds it in Yellowtail. It is the firm only, with existing tags and a profile note that starts
`Auto-filed by /find-investors, unreviewed`: the desk's queue for checking them later. People,
new tags, retagging and everything else still wait for the user's reply and the command it hands to,
which shows its own cards.

Read `${CLAUDE_PLUGIN_ROOT}/references/house-rules.md` first. Scoring is `${CLAUDE_PLUGIN_ROOT}/references/fit-scoring.md`; light
research on new firms follows `${CLAUDE_PLUGIN_ROOT}/references/research-an-investor.md` (only as far as scoring
needs). Worked examples: `${CLAUDE_PLUGIN_ROOT}/references/examples-investors.md` § find-investors.

## Arguments

- **count**: shortlist length, default 20. Up to 40; more than that, suggest a second run with
  narrower criteria. A small count on a running round is a **top-up**: `/investors find 5` returns
  five firms nobody has worked yet, since everything on the project's pipelines and lists is set
  aside (step 2).
- `--snapshot`: the pre-signing snapshot for a prospective client (fit-scoring.md § Snapshot mode):
  exactly 20 firms, count ignored, **existing backers excluded** (firms `won` on any of the
  project's pipelines, investors the project description names, any the user names). Strong and
  Good first; never a Poor or **may be inactive** row; fewer than 20 qualify → say so, never pad.
- `--yellowtail-only`: skip step 5. `--web-only`: skip step 4 (still do steps 1–3, and still check
  each web find against Yellowtail).
- `--no-file`: file nothing; new rows stay **new** and go through `create` as before. Also
  whenever the user says not to save anything.
- Free-text criteria narrow the run ("leads only", "no CVCs", "UK only", "family offices"). Say how
  you applied each one; one you couldn't apply is said, not dropped.

## Tools

| Tool | For | Required |
| --- | --- | --- |
| `whoami` | user, `activeProjectId`, workspace | yes |
| `get_project` | the project's `description` and `interestTagIds` | yes |
| `list_tags` | name the interest tags and group them by kind (`stage`, `industry`, `geography`, `focus`, …) | yes |
| `list_pipelines` | the project's rounds: `target`, `currency`, `attributes`, stages. A project with no pipeline is fine | yes |
| `list_deals` | firms already on a pipeline (`pipelineId`, `query.status`) | yes |
| `list_lists`, `get_list` | the project's lists and the people on them, whose firms are set aside too | yes |
| `list_prospects` | the project's own fit-ranked prospects, already excluding firms on its pipelines | unless `--web-only` |
| `list_companies` | investors in the project's slice, filtered by interest tags | unless `--web-only` |
| `search_signals` | recent signals matching the interests | unless `--web-only` |
| `get_company` | full record + people + recent signals for the top candidates | for the shortlist |
| `search_companies` | is a web find already in Yellowtail? (`q`, `domain`, `linkedinUrl`, `kind`); each match has `score` and `reasons[]` | for every web find |
| `get_company_kind` | the investor fields' keys and options, to read `attributes` | once |
| Web search / fetch | firms not in Yellowtail, and evidence of recent activity | unless `--yellowtail-only` |
| `file_investor` | step 7: file the kept new firms, `dryRun: true` first | unless `--no-file` |

## Steps

### 1. Orient and read the round

1. `whoami` → the project (house rules §1) and its `workspaceId`.
2. `get_project` → `description`, `interestTagIds`. `list_tags` per kind → names and kinds for those
   ids. **No interests → stop**: the project sees no investors until they're set (Project settings →
   Interests in the app). Say so and offer nothing else.
3. **The round size, with or without a pipeline.** Cheque fit and lead appetite need a size, not a
   pipeline. Take the first of:
   - `list_pipelines` (`projectId`) → the open round's `target` and `currency` (and `attributes`,
     e.g. `round`). Several open rounds → ask which.
   - a size the project's `description` states ("raising €2M seed").
   - **one** question, asked once: "Roughly how much is the round, and in what currency?"

   No answer → score without cheque fit and lead appetite, and say so in the header. Never set up a
   pipeline to get a size. Compare cheques in the round's currency; never convert one currency into
   another (fit-scoring.md).
4. Competitors: from the description or the user. If neither names them, name 3–5 you found on the
   startup's site or the web and say they are your list. They drive portfolio conflict.

Print one line before searching: "Searching for Voltframe — Seed 2026, $2M · Seed · Climate Tech ·
Grid management · United Kingdom." With no pipeline it says where the size came from ("$2M from the
project description"). It lets the user stop a wrong run early.

### 2. Who is already being worked

1. **Pipelines.** For each of the project's pipelines, `list_deals` with `query.status` `open`,
   then `won`, then `lost` (no status means open only), `limit: 100`, following `nextCursor`. Keep
   firm id, name, pipeline, stage and status.
2. **Lists.** `list_lists` (`projectId`), then `get_list` on each: a list holds people, and each
   person's firm comes with them. Keep firm id, name and the list it is on.

These firms are **not re-proposed**; they go in the "Already on your pipelines or lists" section,
and a web find with the same name or domain does too. This is what makes a top-up work:
`/investors find 5` on a round that is already running returns five firms not yet worked.

### 3. Get the investor fields

`get_company_kind` (`investor`) once, so `attributes` (`min_check`, `max_check`, `leads_rounds`,
`fund_size`, …) are read against real keys and options.

### 4. Yellowtail

1. **Firms.** Start with `list_prospects` (`projectId`, `query.limit: 100`, following
   `nextCursor`): the project's own fit ranking, the Investors page's Prospects tab — firms matching
   its interests that are on none of its pipelines, each with a fit band, the interests matched, the
   latest positive signal and the best contact. Use its band as the Yellowtail half of the evidence
   and score the rest per fit-scoring.md. For criteria it can't express ("leads only"), or for a
   platform admin who wants to look past the project's slice, fall back to `list_companies` with
   `projectId`, `kind: "investor"`, and `tags` built from the
   interests **grouped by kind** (`{"stage": [...], "industry": [...], "geography": [...]}`: any
   within a kind, all kinds), `limit: 100`. Pass the tags explicitly even though customers are held
   to interests anyway: a platform admin's read is the whole catalog otherwise, and the shortlist
   must mean the same for both. A tag filter includes the tag's children, as project interests do:
   "Europe" finds firms tagged European Union or a European country, so pass the broad tag rather
   than listing its children. Read `meta.totalMatches`; follow `nextCursor` up to ~300 firms. More
   than that → narrow with the user's criteria (`attr: {"leads_rounds": "true"}` for "leads only")
   or ask which interest to tighten.
2. **Signals.** `search_signals` with `projectId`, `status: "published"`, `from` = 120 days ago,
   `tag` = the project's industry and stage interest ids, `limit: 50`. Each signal names a firm: a
   firm with a positive recent signal moves up; a **negative** tag on the project's stage or sector
   is a blocker (fit-scoring.md). A firm that appears only through a signal joins the candidates.
3. **Prefilter** on what the list rows carry (tags by kind, attributes): drop clear blockers, keep
   the best ~2× count. `get_company` on those for people, full tags and its five latest signals.
   Score each (fit-scoring.md § Evidence, not tag overlap): read the description, the
   `notable_investments` and the latest signals before the tags. For each firm write down its
   **last known deal date** (the newest dated deal signal, portfolio announcement or press item):
   more than 18 months ago → **may be inactive**. And check its portfolio against the competitors
   from step 1.4: a direct competitor → ⚠ **portfolio conflict**, a blocker unless the user says
   otherwise; an adjacent company → a line on the row.

### 5. The web

Look for firms that are **active now** in this stage, sector and geography:

- recent rounds: "<sector> <stage> round <country> 2026", trade press (Sifted, TechCrunch, UKTN,
  EU-Startups), funding announcements of companies like the client, not its direct competitors;
- fund closes in the last 24 months with a matching thesis;
- curated lists (Dealroom, OpenVC, accelerator investor lists) only as leads, never as evidence.

For each firm found: `search_companies` with `q`, `domain` (and `linkedinUrl` if you have it),
`kind: "investor"`.

- Match on an on-pipeline firm → "Already on your pipelines".
- Match on a firm you already scored → add the web evidence to it.
- Match on a firm outside the project's slice → keep it, marked **in Yellowtail** with "outside this
  project's interests (tags: …)"; that is usually a missing or wrong tag. Say which tag the
  evidence supports and that it is missing ("Kurma: life-sciences fund, no Health or Biotech
  tag"), and list those firms at the end under **Tags to fix**, handed to `/investors research`
  (fill gaps) to retag with sources. Never retag from here: the only write this skill makes is
  filing a new firm (step 7).
- **What counts as a match.** A match whose `reasons` include `domain`, `linkedin` or `name_exact`,
  or `name_similar` with a `score` of **0.8 or more**, is the same firm. A `name_similar` match
  **below 0.8** with no `domain` or `linkedin` reason is **not** a match: the row is **new**, with
  "looks like <firm> (name only, 0.71)" so the user can check. (`file_investor` treats the same
  look-alikes as weak too: shown, but not blocking the card.)
- No match → **new**. Light research: the firm's site (thesis, stages, team, portfolio) and one
  dated deal or fund close. Enough to score and to tag (step 7 files it); the full profile, the
  contact search for its people and any new tag it needs are `/investors research`'s job. Keep what you
  read (website, LinkedIn page, team page, with URLs) for the profile note and so `/investors research`
  doesn't repeat it.

Untrusted content rule (house rules §7) applies to every page.

**When Yellowtail runs short.** Fewer good fits in Yellowtail than `count` (after the exclusions in
step 2) → lean on this step: search wider and deeper on the web until the shortlist is full or the
web runs out, and say how many came from each. The new rows are what the round is missing, so the
run files them (step 7), saved to Yellowtail for the next run and for everyone else. When a whole segment is thin (say, fewer than 10 firms in scope),
suggest `/investors prospect` to fill it properly.

### 6. Rank and show

Rank by band, then within band (fit-scoring.md). Fill to `count` (exactly 20 in `--snapshot`);
Poor rows only under "Ruled out". Flags come first on a row: ⚠ portfolio conflict, then ⚠ may be
inactive (with the last known deal and its date).

### 7. File the new firms

Unless `--no-file`, after ranking and before showing the shortlist. No card: this is the standing
exception in house rules §2. A firm that fails stays a **new** row, never retried in a loop.

**Which rows.** A **new** row in Strong, Good or Possible that is an **investor**: a fund, angel
network, family office, CVC or other firm that takes stakes. **Not filed**: a strategic acquirer
("strategic buyer, not a fund"), a lender with no equity, a list or directory, anything under Ruled
out or "Considered and dropped", a row flagged ⚠ portfolio conflict, and a row with a name-only
look-alike ("looks like Northwind Capital (name only, 0.71)") the user still has to check. The same
in `--snapshot`.

**The bundle.** `file_investor` with the project's `workspaceId`, the default global scope,
`people: []` and `company: { new: { name, websiteUrl, linkedinUrl, hqCountry, description, tagIds, attributes } }`:

- `description`: one or two sentences on the thesis, from the firm's own site, in plain words.
- `tagIds`: **existing tags only**, mapped as `/investors research` maps them (research-an-investor.md
  § Firm — tags): the most specific tag the evidence supports plus its parent, for each applicable
  kind. A tag the gate in house rules §12 would propose is **not** created here: use the nearest
  existing tag and name the missing one under **Tags to fix** for `/investors research`.
- `attributes`: only values a source states (`get_company_kind` keys); omit the rest.
- `profileNote.body`: first line exactly `Auto-filed by /find-investors, unreviewed`, then "Filed
  during a search for <project>, <date>." and one line per field with its source URL.

**Dry run first** (`dryRun: true`). File it (the same bundle, `dryRun: false`) only when the dry run
is `ok` with **no** `duplicates` of any kind (strong, `weak` or `elsewhere`); otherwise leave it
**new** and say why on its row ("not filed: may be Northwind Capital"). `tagGaps` on a passing dry
run: look once more for each kind with what you read, then file; what is still missing stays for
whoever enriches it. Never pass `confirmNew`.

A filed row becomes **in Yellowtail · just added**, with its `companyId`, and is treated like any
in-Yellowtail row in the reply (it can go on a pipeline straight away). It has **no people yet**.

## Output

Display: inline — `show_shortlist` with every row, grouped as below, `actions` for the replies; one or two lines after it. The block below is what goes on the card, and the text when no card can be drawn (`${CLAUDE_PLUGIN_ROOT}/references/display.md` §2). Yellowtail's reads draw nothing on the way; this output is the one thing shown (§ Only the answer is drawn).

```
**Investors for Voltframe — Seed 2026 ($2M)** · 20 ranked: 13 in Yellowtail, 7 new
Interests: Seed · Climate Tech · Grid management · United Kingdom
Searched: Yellowtail (148 firms in scope, 31 signals since 11 Jun) · web (14 queries)
Added to Yellowtail: 6 of the 7 new firms (unreviewed, no people yet) · 1 not filed (row 9)
Not scored: lead appetite (the round doesn't say whether it has a lead)

### Strong
1. **Northwind Capital** · in Yellowtail
   Leads UK seed climate; $250K–$1M cheques fit as lead
   · Signal 2 Oct 2026: "now leading pre-seed climate in UK + Nordics"
   · Led Gridly seed, Jun 2026 (northwind.vc/portfolio)
   Contact in Yellowtail: Sarah Lee, Partner
2. **Kiln Climate** · in Yellowtail · **just added**
   Seed fund for UK energy software; Fund I £40M closed Feb 2026 (kiln.vc/news)
   · Led Ampwise seed, Aug 2026 (sifted.eu/…)
   No contacts yet

### Good
…
### Possible
14. **Brightwater Ventures** · in Yellowtail · ⚠ may be inactive
   Last known deal: Mar 2025 (brightwater.vc/portfolio); no Yellowtail signal since
   Seed climate thesis, Nordics and UK (brightwater.vc)
…

**Already on your pipelines or lists** (not re-proposed)
- Lantern Ventures — Seed 2026 · First meeting (open)
- Halcyon Ventures — Seed 2026 · lost
- Quarry Lane Ventures — on list "Tier 1 climate leads" (Elena Varga)

**Not filed** (still new; `create` files them with a card)
- 9. Northwynd Partners — looks like Northwind Capital (name only, 0.71)

**Ruled out**
- Ember Capital — ⚠ signal 30 Sep 2026: Series A only, "− Pre-seed"
- Flint Ventures — ⚠ portfolio conflict: backs GridMind, a named competitor (flint.vc/portfolio)

---

Reply with any of: `add 1 2 to pipeline` · `add 1 4 to list <name>` · `contacts 2` · `create 9` · `brief 1` · `more`
```

Numbers stay fixed for the rest of the conversation, so "add 3" always means the same firm.

In `--snapshot` the header reads "Snapshot for Voltframe: 20 firms · existing backers excluded
(Ashgrove Angels, won on Pre-seed 2025)" and the reply line is the same.

## Taking the reply

Nothing more is written here. Each reply runs the named command with the selected rows, and that
command shows its own cards and takes its own approval.

| Reply | Does |
| --- | --- |
| `add 1 3 to pipeline [name]` | `/pipeline add` with those firms (ids) and the round; **just added** rows go too. A row still **new** (not filed) can't go on a pipeline until it exists: say so and offer `create` for it first |
| `add 1 4 to list <name>` | `/lists add` with those firms. A list holds people, so a firm with no contacts (every **just added** one) gets `/investors contacts` first |
| `create 9` | `/investors research` for rows still **new**, as one batch (≤5 cards per round), passing on what you already read. Offer `add … to pipeline` once they are filed. An in-Yellowtail row here, **just added** ones included, means "enrich" (full profile, people, new tags) and goes the same way |
| `brief 1` | `/investors brief` for that firm |
| `contacts 2` | `/investors contacts` for that firm |
| `more` | the next `count` from the same run, same numbering continued |

## What comes next

Offer `/pipeline add` for the strong in-Yellowtail rows (just added ones included) and
`/investors contacts` for the strong just added ones, which have no people yet. Rows left **new** →
`create`. A segment Yellowtail barely covers → `/investors prospect`. Once the round has firms on it: `/investors brief` before a pitch, `/outreach draft` to start.
