# Worked examples: investors

All names are fictional. The setting for every example: workspace **Atlas Advisory**; project
**Voltframe**, a London startup selling grid-flexibility software to utilities; interests Seed ·
Climate Tech · Grid management · United Kingdom; pipeline **Seed 2026**, target $2,000,000, stages
Researching → Contacted → First meeting → Diligence → Term sheet. Competitors (from the project
description): Flexa, GridMind.

Tag names are as of Oct 2026. Always re-read `list_tags` and `get_company_kind`; don't trust this
file over them. These double as test cases: run each against a local Yellowtail and compare.

---

## find-investors

### 1. Default run

**Input**: `/investors find`

**What the tools return, in this example**

- `get_project` → 4 interest tags. `list_pipelines` → Seed 2026 open, `target` 2000000, `currency` USD.
- `list_deals` (open / won / lost) → Lantern Ventures (First meeting, open), Halcyon Ventures (lost).
- `list_companies` (`tags` = `{stage:[Seed], focus:[Climate Tech], industry:[Grid management],
  geography:[United Kingdom]}`) → `meta.totalMatches` 41.
- `search_signals` (last 120 days, industry + stage tags) → 9 signals; one is Ember Capital,
  30 Sep, "− Pre-seed · − Seed · + Series A".
- Web → 11 firms; `search_companies` matches 4 of them (one is Lantern, on the pipeline), 7 new.
  One of the 7, "Northwynd Partners", came back with a single `name_similar` match at 0.71 to
  Northwind Capital and a different domain: it stays **new**, with "looks like Northwind Capital
  (name only, 0.71)" on its row.
- `get_company` on Brightwater Ventures → notable investments up to 2024, no signal since; its last
  announced deal is Mar 2025.

**Expected output**

- Header line names the round, target and interests, and "Searched: Yellowtail (41 firms in scope,
  9 signals) · web".
- 20 rows across Strong / Good / Possible, each marked **in Yellowtail** or **new**, each with up to
  three sourced evidence lines (a dated deal or signal, the thesis sentence, the cheque); Northwind
  near the top on its 2 Oct signal and its Gridly seed.
- Brightwater Ventures flagged **may be inactive** (last known deal Mar 2025, over 18 months ago) and
  no higher than Possible.
- **Already on your pipelines**: Lantern Ventures (First meeting), Halcyon Ventures (lost).
- **Ruled out**: Ember Capital, with the 30 Sep signal as the blocker.
- 6 of the 7 new firms filed by `file_investor` (dry run, then real; `people: []`, existing tags,
  profile note starting `Auto-filed by /find-investors, unreviewed`), each row now **in Yellowtail ·
  just added**; the header says "Added to Yellowtail: 6 of the 7 new firms". Northwynd Partners is
  **not filed** (its dry run returns the weak look-alike) and is listed under **Not filed**.
- The reply line.

Things that must NOT happen: Lantern in the ranked list because the web found it again; a fit
"score" as a number; a new firm ranked Strong on a site visit alone with stage, cheque and activity
unknown; a firm ranked Strong on matching tags with no dated evidence; Northwynd treated as
Northwind on a name-only look-alike, or filed; a strategic buyer or a ruled-out firm filed; a
person, a new tag or a retag written; any write with `--no-file`.

### 1b. Snapshot

**Input**: `/investors find --snapshot`

**Expected**: exactly 20 rows, none of them a firm on any of the project's pipelines with status
`won` (existing backers) or already open on the round; the header says "Snapshot: 20 firms,
existing backers excluded". Fewer than 20 that pass → say how many and why, never pad with Poor rows.

### 2. The reply

**Input**: `add 1 2 to pipeline` · `create 9`, where 1 was in Yellowtail, 2 was just added and 9
was not filed.

**Expected**: `/pipeline add` runs for rows 1 and 2 on Seed 2026 and shows its own cards (row 2
has no people yet, so its card offers `/investors contacts` for the primary contact); `/investors research`
runs for row 9, reusing the pages already read, and shows its card with the look-alike under **Needs
your input**. Nothing is written until each command's approve. If the user had written
`add 9 to pipeline`, row 9 isn't in Yellowtail: say it must be created first and offer `create 9`.

---

## create-investor

### 3. A new firm with people

**Input**: `/investors research Kiln Climate`

**Resolution**: `search_companies q="Kiln Climate"` → nothing. Research finds kiln.vc;
`search_companies domain="kiln.vc"` → nothing → new. Hunter is connected: `domain_search { domain:
"kiln.vc", seniority: ["senior","executive"] }` returns hana@kiln.vc (`emailStatus: "verified"`)
and james.rowe@kiln.vc (`emailStatus: "unverified"`, Hunter `accept_all`). The site says "We lead seed rounds in UK energy
software, cheques £500k–£1.5m"; Fund I £40M closed Feb 2026 (press release); team page lists Hana
Ito (Partner) with hana@kiln.vc; LinkedIn shows James Rowe, Principal.

**Expected card**

- **Kiln Climate — new investor** · kiln.vc · HQ United Kingdom · LinkedIn page
- Description from the site, one or two sentences.
- Type: Early-stage VC · Focus: Climate Tech · Stage: Seed · Industries: Energy, Grid management
  (if the thesis names grid) · Invests in: United Kingdom
- Leads: yes · Cheque: $250K to $1M – $1M to $3M (£500k ≈ $650k; £1.5m ≈ $1.95m) ·
  Fund: $50M to $200M (£40M ≈ $52M; within ~10% of the $50M edge → **left empty, and the card says
  why**)
- Not found: climate investments, parent company, software only
- People, each with all five contact lines (house rules §5):
  - Hana Ito — **new**, Partner · LinkedIn (unverified; kiln.vc/team link) · hana@kiln.vc
    (verified; listed on kiln.vc/team, and Hunter's `source` line) · X if linked, else
    `not found — searched: …` · phone · location
  - James Rowe — **new**, Principal · LinkedIn (unverified; web search, firm and title match) ·
    james.rowe@kiln.vc (unverified; Hunter's `source` line, accept_all) · X · phone · location.
    Without Hunter: james@kiln.vc (unverified; pattern from hana@kiln.vc, kiln.vc/team)
- Each person's bundle carries `sources` for every contact value; the email's is Hunter's `source`
  verbatim.
- Profile note · sources listed
- The dry run was called with `workspaceId` = Atlas Advisory before the card was shown, and the card
  says it files to the shared catalog for Atlas Advisory.

Things that must NOT happen: `fund_size` filled despite the bucket edge; James's address filed
`verified` (Hunter said `unverified`), or a pattern address shown without naming hana@kiln.vc as the
evidence; a pattern email at a firm where no address is published; a person line with no LinkedIn
line; `leads_rounds: false` for a firm that didn't say; the card filed without `workspaceId`; a
`domain_search` run on every firm in a list "to be safe" (it spends quota).

If the site says "we back climate robotics" and there is no such industry tag: the card's **New
tags (created on approve)** carries industry · "Climate robotics" (group: Robotics & hardware, its
id from `list_tags { kind: "industry" }`), with the sentence and the near misses. On approval, by
anyone, `create_tag { kind: "industry", name: "Climate robotics", parentId }` runs first and the
card files with the returned id. If it answers `created: false` (say, "Robotics" already exists and
the server judged it the same tag), the card uses that tag and the report says so. A proposed
**geography** tag on the same card, for a customer, is never proposed as created on approve: the
card files without it and lists it under **Tags to ask the Yellowtail team for**. The new
"Climate robotics" tag goes on next to its parent, Robotics & hardware, not instead of it.

### 4. A shared-catalog firm

**Input**: `/investors research Northwind Capital, and add Raj Patel`

**Resolution**: Northwind matches by domain; `workspaceId` and `projectId` are null → shared
catalog. Its HQ looks wrong (site says Dublin), and it has no LinkedIn page on file.

**Expected**: a **fill gaps (shared catalog)** card: Fill LinkedIn page; ⚠ Change HQ United Kingdom
→ Ireland with northwind.vc/contact quoted; Raj Patel **new**. Filed with `workspaceId` = Atlas
Advisory; the result's `filedTo` is `global`, and the report says the change is in Northwind's
history under the user's name.

Things that must NOT happen: a second, workspace-private Northwind created to hold the change;
`scope: "workspace"` used without the user asking for it.

---

## find-contacts

### 5. Ranking by fit, not seniority

**Input**: `/investors contacts Northwind Capital`

**Resolution**: Yellowtail has Sarah Lee (Partner, LinkedIn, no email). The team page lists a
Managing Partner (generalist, last deal 2023), Raj Patel (Partner, led Gridly seed Jun 2026) and
Ola Berg (Platform Lead). The team page shows `sarah@northwind.vc` and `raj@northwind.vc`.

**Expected contact lines**: LinkedIn for all three (searched, unverified); raj@ and sarah@ from the
team page; Ola's `ola@northwind.vc` as a pattern from those (unverified, evidence named); X, phone and
location found or `not found — searched: …` for each.

**Expected**: Raj first (grid deal this year + a recent signal), Sarah second with "add email",
Ola last flagged "routes intros", the Managing Partner listed below the card as "not proposed:
generalist, no climate deals since 2023" (the user can add him with `1: add <name>`). One card,
`company: { "id" }` plus the people (Northwind is in the shared catalog, and any member may add
people to it). With Hunter connected: one `domain_search` for northwind.vc, then `verify_email` on
the team-page addresses; each filed with the status Hunter returned and its `source`.

Things that must NOT happen: the Managing Partner ranked first because of his title; a personal
Gmail address found on a conference page (the server refuses it anyway); a contact line silently
missing; Raj filed without checking `search_people` for a Raj
Patel at Halcyon (a job move).

### 5b. A firm Yellowtail doesn't have

**Input**: `/investors contacts www.madx.digital - just 5 new contacts`

**Resolution**: `search_companies` finds nothing by domain or name. madx.digital's home and about
pages describe an SEO and digital-marketing agency; there is no portfolio, fund or investing page.

**Expected**: "MADX Digital isn't in Yellowtail, and it doesn't look like an investor: it's an SEO
and digital-marketing agency (madx.digital/about). Did you mean `/investors find` — investors for
MADX, if it's raising (or `/setup project` to set it up as a client first)? If MADX does invest and
I've missed it, say so and I'll add it as an investor and find 5 contacts there." Nothing else is
researched until the user answers.

Had the site shown a portfolio (say, "MADX Ventures backs early-stage B2B SaaS", with six companies
listed), the answer is instead "isn't in Yellowtail yet; it looks like an investor (…). Add it and
then find contacts? `yes` / `no`", and a `yes` leads to one card with the firm (fully tagged) and
its five people.

Things that must NOT happen: stopping at "not in Yellowtail" with nothing to do next; a list of
unrelated tools (ActiveCampaign, accounts) instead of the one question; MADX filed as an investor
without the user saying it invests; people researched or Hunter spent before the question.

---

## investor-brief

### 6. A firm with history

**Input**: `/investors brief Lantern Ventures`

**Resolution**: Lantern is on Seed 2026 at First meeting; `get_deal` shows primary contact Mira
Osei and two comments ("Mira wants churn data", "Partner meeting pencilled for 15 Oct"). Lantern's
portfolio page lists GridMind.

**Expected**: the Meeting Prep Brief layout (`meeting-prep-brief.md`). The header carries "⚠
Portfolio conflict: GridMind (a named competitor) is a Lantern portfolio company
(lanternvc.com/portfolio)", fit **Poor** on that blocker unless the user says otherwise, and the
history so far; sections 1–10 follow in order, with section 5 giving the conflict in full and
section 7 including "How would you work with us alongside GridMind?" with an answer drawn from the
data room index, or `[needs: …]`. The angle section says plainly that the conflict comes first:
raise it with Mira before the partner meeting.

Things that must NOT happen: the conflict buried below the angle; a fit band of Strong because the
deal is already moving.

---

## investor-prospecting

### 7. A segment

**Input**: `/investors prospect UK climate pre-seed funds`

**Expected**: one line mapping the segment to tags (Pre-seed · Climate Tech · United Kingdom ·
Early-stage VC); "Yellowtail has 23 firms matching"; web discovery; then the triage report
(found / already in Yellowtail / new / unclear) and a question. **No web research on any firm before
the user answers.** Then five cards per round, filed to the shared catalog for Atlas Advisory, with
"N still to research" after each. A firm whose dry run shows it under `elsewhere` (another
workspace's private list) is dropped with "already in Yellowtail".

Things that must NOT happen: a card for a firm `search_companies` matched by domain; "Nordics"
turned into a guessed country list; a "Climate" tag proposed when "Climate Tech" exists; a sixth
card in one round; people filed without the contact search because the run is large.
