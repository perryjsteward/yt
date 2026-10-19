# Researching an investor

Used by `/create-investor`, `/find-contacts`, `/investor-prospecting` and the "new" rows of
`/find-investors`: whenever a firm or person isn't in Yellowtail yet, and when filling the empty
fields of one that is. The aim is a profile as complete as the public web supports, and **not one
field more**. An empty field is honest; a guessed one is a bug every Yellowtail customer sees,
because investors are filed to the **shared (global) catalog** (§ Where it files).

The second half of this file is the **investor card** and the `file_investor` bundle it becomes.
Every skill that files an investor uses exactly that card.

## Order of work

1. **Pin the firm down first.** Find its own website. Two firms with one name are common
   ("Lantern Ventures" London vs "Lantern Capital" Berlin); the website domain and LinkedIn company
   page tell them apart, and they are what `file_investor` dedups on. If you can't tell which firm
   the user means, ask; don't research both.
2. **Re-check Yellowtail with what you found.** `search_companies` again with `domain` and
   `linkedinUrl` (and `kind: "investor"`). Research often turns a "new" firm into an existing one.
   Note the match's scope: `workspaceId` and `projectId` both null = the **shared catalog**;
   `workspaceId` = this workspace's = **this workspace's private copy**. A firm that only exists in
   another workspace's private catalog comes back from the dry run under `duplicates[].elsewhere`
   (name and website only): it is already in Yellowtail, so don't file it again.
3. **Read the firm's own site**: home, about/thesis, team, portfolio, "what we invest in", FAQ, news.
   This is the primary source for everything below.
4. **Then a second source** for what the site doesn't say: the LinkedIn company page, the fund-close
   press release, Crunchbase / Dealroom / PitchBook public profile pages, reputable trade press
   (Sifted, TechCrunch, the FT, UKTN, EU-Startups…), the firm's Companies House / SEC Form D filing
   for a fund size.
5. **People**: the team page first, then each person's LinkedIn, then what they publish (posts,
   blog, podcast appearances). Professional facts only (§ People). For every person, run the
   **contact search** (§ Contact search) before the card: LinkedIn, work email, X, work phone,
   location, each found or `not found — searched: <where>`.

Stop when the remaining fields have no source. Five to ten minutes per firm is plenty; the user would
rather wait than review guesses.

## What to fill, and from what

Read the live definitions every session with `get_company_kind` (`kind: "investor"`) and
`list_tags` (one kind at a time). Field keys, select options and tag names change; never trust the
lists below over the tools. They are a guide to mapping evidence, as of Oct 2026.

### Firm — core

| Field | Fill from | Rule |
| --- | --- | --- |
| `name` | site / LinkedIn | the name the firm uses ("Northwind Capital", not "Northwind Capital Partners LLP") |
| `websiteUrl` | the site | the root, `https://northwind.vc` |
| `linkedinUrl` | LinkedIn company page | the company page, never a person's. Always searched: `"<firm>" site:linkedin.com/company`, or the link in the site's footer |
| `hqCountry` | site footer, about page, LinkedIn | ISO code. Where the firm is based, not where it invests |
| `description` | the firm's own words, rewritten | 1–2 plain factual sentences: who they back, at what stage, where. Every Yellowtail customer reads it; no adjectives they didn't earn, nothing said in confidence |
| X handle (no field) | linked from the site's header or footer | on the card and in the profile note only; a firm has no X field |

### Firm — tags

Tags are what matching runs on: a firm with the wrong industry, or none, never reaches the
founders it should. So tagging is a required step with its own check, not what is left over.

**Read the whole tree first.** `list_tags` for every kind `get_company_kind` says applies, the
industry kind with its children (`parentId`). Most sectors a thesis names are already there one
level down: "therapeutics" is *Biotech*, "chips" is *Semiconductors*, "agbio" is *Ag biotech*.

**Every applicable kind is answered.** For each kind, the card shows its tags, or `not stated —
searched: <where>`. Never a kind left off silently. The dry run's `tagGaps` lists the kinds still
empty; each must be on the card as not stated (with where you looked), or mapped before the card
is shown.

| Tag kind (Oct 2026) | Evidence | Notes |
| --- | --- | --- |
| `investor_type` | how the firm describes itself: VC, CVC, angel network, family office, university fund… | many-of. Almost always findable: a corporation's fund is *Corporate VC*; a university's fund is typed by what it does (usually *Early-stage VC*); a state body is *Government* |
| `stage` | stated stages, or the rounds they say they lead | Pre-seed, Seed, Series A, Series B+. A stated range covers the stages inside it ("seed to Series B" is Seed, Series A, Series B+); "all stages" / "multi-stage" is all four. "Early stage" alone is not a stage: look for the rounds or the cheque size and say which |
| `focus` | Generalist only if they say so; Climate Tech / Deep Tech if that is the thesis | |
| `industry` | sectors named on the thesis page | the **most specific** tag the source names, **and its parent** |
| `geography` | where they say they invest | countries when named; a region tag only when the region is named. A fund that backs one university's or one country's companies invests in that country. "DACH", "Nordics" are not tags: list the countries only if a source lists them |
| `business_model` | B2B, B2C, marketplace… if the thesis states it | |

Mapping rules, each one a mistake made in a real filing:

- **Most specific, plus its parent.** A semiconductor and photonics fund is *Semiconductors* +
  *Robotics & hardware*, not *Industry*. A life-sciences fund is *Health* + *Biotech* (and
  *Medtech* if devices are named), not untagged because no tag says "life sciences". A parent
  alone, when a child the source names exists, is under-tagging.
- **The main sector first.** "Mainly therapeutics, also agbio" is *Health* + *Biotech* first, and
  *Food & agriculture* + *Ag biotech* second. Never the secondary sector alone.
- **Words, not vibes.** Every tag has a sentence that states it. Portfolio patterns are not a
  thesis: "three of their companies are batteries" does not make them a batteries investor.
- **No padding.** A firm naming twelve sectors gets the ones its thesis leads with, not every
  child in the tree. Eight or more industry tags on one firm means you are tagging its portfolio,
  not its thesis: cut back to what it says it focuses on.
- **Synonyms map; they don't create.** "Life sciences", "biopharma", "therapeutics" → Biotech.
  "Cleantech", "energy transition" → Energy (and Climate Tech focus if that is the thesis).
  "Mobility" → Mobility & logistics or Transportation, whichever the thesis means. "Quantum",
  "photonics" → Semiconductors only if that is how the firm groups them; otherwise the parent
  (Robotics & hardware) and the word in the description.

If a source names a sector with no tag under any spelling or synonym, check it against the gate
in house rules §12 (it is strict). A term that passes goes under the card's **New tags (created
on approve)**; one that fails gets the nearest existing tag at the level that is true (usually
the parent) and the specific word goes in the description. **Never substitute a vaguer tag when
a specific one exists, and never leave the sector untagged.**

- **industry** and **focus** tags anyone may create. An industry tag needs its group as `parentId`
  (one of the groups `list_tags { kind: "industry" }` returns; never a hard-coded id). On
  approval, `create_tag { kind, name, parentId? }` runs first and the card files with the returned
  id. `created: false` means the kind already has that tag under another spelling: use the returned
  tag's id and say so in the report ("used existing *Grid management*: same tag, plural").
- **Other kinds** (`stage`, `geography`, `investor_type`, `business_model`) are the Yellowtail
  team's. A platform admin's `create_tag` works; for anyone else, don't propose it as created on
  approve: the card files without it and lists it under **Tags to ask the Yellowtail team for**
  (kind, name, the source), suggesting the user mentions it to the team.
- **Alongside the parent.** A new specific tag goes on the firm next to its broad parent, never in
  its place, so matching on the parent keeps working for every firm.

### Firm — custom fields (`attributes`)

Investor fields as of Oct 2026 (confirm with `get_company_kind`):

| Key | Type | Evidence | Rule |
| --- | --- | --- | --- |
| `fund_size` | select: Not applicable · Below $10M · $10M to $50M · $50M to $200M · Over $200M | the latest fund's close announcement or filing | convert to USD at a rough current rate; within ~10% of a bucket edge → leave empty and say why. "Not applicable" only for evergreen / balance-sheet investors that say so |
| `min_check` / `max_check` | select: Below $250K · $250K to $1M · $1M to $3M · $3M to $10M · Over $10M | a stated cheque range | same conversion rule. A single "typical cheque" fills neither; say it in the description |
| `leads_rounds` | boolean | "we lead", "we take board seats" → true; "we co-invest / follow" → false | silence → empty, never false |
| `climate_investments` | select: 0 · 1 to 5 · 6 to 10 · 11 to 20 · 21 to 50 · Over 50 | count of climate companies on the published portfolio page | only if you actually counted; otherwise empty |
| `software_only` | boolean | an explicit "software only" / "no hardware" | silence → empty |
| `parent_company` | text | CVCs and fund-of-funds arms: the parent | |
| `notable_investments` | text | 3–5 portfolio names from their own site | names only, comma-separated |

Omit a key you have no value for; never send `null` or `""` for "unknown".

### Facts that have no field

Fund vintage, dry powder, recent deals, partner theses, portfolio conflicts: these drive fit
(`fit-scoring.md`) and belong in the **profile note** and on the card, not in invented attributes.

### People

Who to find: the **partner most likely to champion this round**: the one whose sector, stage and
recent deals look like the client's, not the managing partner by default. Then other partners and
principals who make or champion decisions at the client's stage, then the platform /
investor-relations person if the firm routes intros through one. Analysts only when the firm says
they source deals. Five people is a lot; two well-chosen ones beat a team page dump. Always a named
person: a role or shared inbox is never a contact (below).

| Field | Fill from | Rule |
| --- | --- | --- |
| `fullName` | team page / LinkedIn | as they write it |
| `roleTitle` | team page first, LinkedIn second | their title at **this** firm; if the two disagree, prefer the newer and say so |
| `linkedinUrl` + `linkedinStatus` | LinkedIn | **always searched** (below); the dedup key, so get the right profile (match firm, title, city). `unverified` unless the user gave or confirmed it |
| `email` + `emailStatus` | the contact search's email order (below) | **work address only**, its source or evidence pattern on the card |
| `phone` + `phoneStatus` | the firm's site / contact page, a work number they publish, an enrichment connector | work numbers only (office, a published work mobile). Never a personal number |
| `twitterHandle` | linked from their site, the team page or LinkedIn | only if clearly theirs; `@handle` without the URL |
| `locationCity` / `locationCountry` | LinkedIn / team page | the city they work from; ISO country code |

### Contact search

Required for every person on a card (house rules §5). Run it in this order and write down where you
looked; that list is what `not found — searched: …` reports.

1. **LinkedIn.** Web search `"<full name>" "<firm>" site:linkedin.com/in`; the team page's LinkedIn
   links; the person's own site or Substack. Take a profile only when firm, title and city match.
   A person card with no LinkedIn line is incomplete.
2. **Work email**, stopping at the first that gives one:
   1. **Hunter, through Yellowtail, first** (the `enrichment` toolset, when the workspace has a
      Hunter key in Settings → Integrations): `find_email { fullName, domain }` (or `{ personId }`
      for someone on file) for the named partner, then `verify_email { email }` on what it returns
      or on an address found anywhere else. `domain_search { domain, seniority:
      ["senior","executive"] }` once per firm when you are still choosing who to contact: it lists
      the people Hunter knows, with their addresses and the firm's pattern. File the address with
      **exactly** the answer's `emailStatus` and put its `source` in `sources.email`; never upgrade
      it. Only Hunter's `valid` at a score of 90 or more comes back `verified`. Each call spends the
      workspace's Hunter quota, so call it for the people on the card, not the whole team page.
      Apollo or Clay, if connected as their own connectors, work the same way: `verified` only
      when the tool reports the address deliverable, naming the tool;
   2. **a published address of the person's own**: the firm's team, contact or "pitch us" page
      naming them, a press release that names them as the contact, or an address they publish
      (their site, newsletter, a talk's slides). With a Hunter key, run `verify_email` on it;
   3. **pattern inference, by hand**: only if at least one real address at the firm's own domain
      is published (step 2 or a press release). Apply its pattern, file `unverified`, and on the card
      write `pattern from hana@kiln.vc (kiln.vc/team)`. No published address at the domain, or two
      that disagree → no email.

   **No Hunter key** (the tools refuse: "Hunter is not connected for this workspace…"): do steps 2
   and 3, and say once in the run, not per person: "I searched by hand; for verified emails connect
   Hunter in Settings → Integrations (lemlist is being considered)."

   **A person's email is a person's.** Never file a role or shared inbox as someone's `email`:
   `info@`, `hi@`, `hello@`, `contact@`, `team@`, `office@`, `invest@`, `investments@`, `pitch@`,
   `deals@`, `media@`, `press@`, `admin@` or anything like them, whoever's name is next to it. A
   firm's general inbox can go in the firm's profile note ("general inbox: hello@kiln.vc, from
   kiln.vc/contact"), never on a person, and never as the evidence for a pattern.
3. **X / Twitter.** Linked from their site, the team page or LinkedIn. A search hit counts only if
   the X profile names the firm or their role.
4. **Work phone.** The firm's contact page (office line), a work number they publish, enrichment.
5. **Location.** LinkedIn, then the team page or the office they're listed under.

Never a personal email or a personal number, even if a source shows one. `file_investor` refuses a
work email on a personal mailbox (gmail.com, googlemail.com, icloud.com, me.com, proton.me,
protonmail.com, aol.com, and hotmail / outlook / live / yahoo / gmx on any domain), so drop it
before the card rather than letting the dry run catch it.

Every contact value on a card carries a **source**, and the bundle sends it: `sources { email?,
phone?, linkedinUrl?, twitterHandle? }`, one short line each ("kiln.vc/team", "pattern from
hana@kiln.vc (kiln.vc/team)", Hunter's own `source` line). Yellowtail stores it per field with a
date and shows it on hover in the app.

What a person card also carries, for the user and the profile note (not fields): background in a
line (ex-operator, ex-founder of X), the 2–3 most relevant recent investments they led, and a recent
post or talk that shows what they want now, each with a link.

A person is always filed **at a firm**. If you can't establish their current firm, they aren't
filed; ask. A same-named person at another firm in Yellowtail is either someone else or a job move
(`moveToCompany`); never assume which.

## Where it files

Investors are shared. `file_investor` files to the **global catalog** for anyone, admin or not;
`workspaceId` is the workspace the card is filed **for**: required unless you are a platform admin,
recorded as provenance, and where the profile note is kept. The result says where it went
(`filedTo: "global"`). `scope: "workspace"` files to the workspace's own private catalog instead:
only when the user asks for that ("keep this one private to us"), and the card says so.

(`file_account`, for startups and other prospects, is different: it files to the workspace's own
catalog, as before.)

## Existing firms: filling gaps, not rewriting

Any member may enrich an investor in the shared catalog, as on one in their own workspace; every
change is in the firm's history with who made it, so be as careful as you would be on someone
else's record.

- Empty fields with a source → **Fill** (`fill` is refused on a field that has a value).
- A filled field that disagrees with a source → **⚠ Change** on its own line, `old → new`, with the
  source. Change only what a source contradicts, and only what the user approves.
- Tags: add what a source states; remove only on explicit evidence ("we no longer invest in…"), as
  its own line.
- People: add new ones, or update the contact details and title of people on file.
- Never remove a tag because you didn't find it mentioned.

## Provenance: the profile note

Every new firm, and every firm you enrich, carries a **profile note** (`profileNote.body`): where each
value came from and what wasn't found. Required for new records. It is how a teammate checks a value
later without redoing the work.

```
Profile researched 9 Oct 2026 for /create-investor (project Voltframe).

- Website, HQ, description, stages, leads rounds: https://quarrylane.vc/about
- Fund size (Fund II, €80m ≈ $87m → $50M to $200M; vintage 2025): https://quarrylane.vc/news/fund-ii
- Industries, notable investments: https://quarrylane.vc/portfolio
- Investor type: https://www.linkedin.com/company/quarry-lane-ventures
- Not found: climate investments, parent company, software only.
- Elena Varga — General Partner (team page), Vienna (LinkedIn): https://www.linkedin.com/in/…
  Email elena@quarrylane.vc (verified): listed on https://quarrylane.vc/team; Hunter verifier
  2026-10-09 (valid, 96)
  X @elenavarga: linked from her LinkedIn · Phone: not found (searched quarrylane.vc/contact, Hunter)
- Tomas Novak — Principal (team page), Prague (LinkedIn): https://www.linkedin.com/in/…
  Email tomas@quarrylane.vc (unverified): pattern first@quarrylane.vc, from elena@quarrylane.vc
  on https://quarrylane.vc/team · X: not found (site, LinkedIn) · Phone: not found
- Recent deals (not filed as fields): Robotiq seed lead, Mar 2026 (https://…)
```

---

## The investor card

Plain markdown in the chat, readable on a phone: short lines, no wide tables, no ids. At most
**5 investor cards per round**; say how many remain.

```
**Investors for Voltframe** · shared catalog, filed for Atlas Advisory · 1 new, 2 to enrich

---

### 1. Quarry Lane Ventures — new investor
- quarrylane.vc · HQ Austria · linkedin.com/company/quarry-lane-ventures · X @quarrylanevc
- "Leads pre-seed and seed rounds in deep tech companies across Central Europe."
- Type: Early-stage VC · Focus: Deep Tech · Stage: Pre-seed, Seed
- Industries: Industrial automation (**new tag**, below) · Invests in: Austria, Czechia, Poland
- Leads: yes · Cheque: $250K to $1M – $1M to $3M · Fund: $50M to $200M (Fund II, 2025)
- Notable: Robotiq, Fabrik, Lumen
- Not found: climate investments, parent company, software only
- Tags not stated: business model — searched: quarrylane.vc/about, /thesis
- Sources: site (about, team, portfolio), Fund II release, LinkedIn
- **New tags (created on approve)**
  - industry · "Industrial automation" (group: Robotics & hardware) — quarrylane.vc/thesis:
    "robotics and industrial automation" · near misses: Robotics (kept as well), Deep Tech (focus;
    broader, not a sector)
- **People**
  - Elena Varga — **new**, General Partner · led Robotiq seed (Mar 2026); posts weekly on industrial AI
    LinkedIn linkedin.com/in/elena-varga (unverified; linked from quarrylane.vc/team)
    Email elena@quarrylane.vc (verified; listed on quarrylane.vc/team · Hunter verifier 2026-10-09 (valid, 96))
    X @elenavarga (linked from her LinkedIn)
    Phone not found — searched: quarrylane.vc/contact, Hunter
    Location Vienna, AT (LinkedIn)
  - Tomas Novak — **new**, Principal
    LinkedIn linkedin.com/in/tomas-novak (unverified; web search, firm and title match)
    Email tomas@quarrylane.vc (unverified; pattern from elena@quarrylane.vc, quarrylane.vc/team)
    X not found — searched: quarrylane.vc/team, LinkedIn
    Phone not found — searched: quarrylane.vc/contact
    Location Prague, CZ (LinkedIn)
- **Profile note** · 5 sources

### 2. Northwind Capital — fill gaps (shared catalog)
- Fill: LinkedIn page · min cheque $250K to $1M · leads rounds: yes (northwind.vc/thesis)
- ⚠ Change: HQ United Kingdom → Ireland (northwind.vc/contact: "Dublin office, est. 2019")
- Tags: + Sweden · + Norway (northwind.vc/thesis)
- **People** · Sarah Lee (existing) · LinkedIn on file · add email sarah@northwind.vc (unverified;
  northwind.vc/team) · add X @sarahlee_vc (linked from northwind.vc/team) · phone not found —
  searched: northwind.vc/contact · location London, GB (on file)
- **Profile note** · 2 sources

### 3. Lantern Ventures — add people (shared catalog)
- Nothing to fill: every field the sources support is already on file.
- **People** · Mira Osei — **new**, Partner
  LinkedIn linkedin.com/in/mira-osei (unverified; lanternvc.com/team link)
  Email not found — searched: lanternvc.com/team and /contact, no published address at the domain
  X @miraosei (linked from lanternvc.com/team) · Phone not found — searched: lanternvc.com/contact
  Location London, GB (LinkedIn)

---

**Needs your input**
- **A.** "Brightwater": *Brightwater Ventures* (Stockholm) is already in Yellowtail as
  brightwater.vc (same domain) — same firm, or a new one?

**Worth a look** (name-only look-alikes; they don't stop a card)
- Card 1: "Quarrylane Partners" (London, quarrylanepartners.com) — different domain; treated as a
  different firm unless you say otherwise.

Reply with any of: `approve all` · `approve 1 3` · `reject 2` · `2: keep HQ as UK` · `1: not Tomas` · `A: <answer>`
```

Card rules:

- Mark every entity `(existing)`, **new**, or **moves job**, and the firm's scope: **new investor**
  (files to the shared catalog), **(shared catalog)**, or **(this workspace)** for a private copy.
- Every person shows all five contact lines (LinkedIn, email, X, phone, location), each with its
  status and source, or `not found — searched: <where>`. A missing line is a search not done.
- **New tags (created on approve)** name kind, name, the group (`parentId`) for an industry tag,
  the source sentence and the near misses ruled out (house rules §12). A tag of a kind you may not
  create is still shown, marked "the Yellowtail team's; won't be created from this card" unless you
  are a platform admin.
- Name-only look-alikes (`weak: true`) go under **Worth a look**, not **Needs your input**.
- **Not found** is not optional on a new firm: it says the empty fields are deliberate.
- **Tags not stated** is not optional either: every kind in the dry run's `tagGaps` is on it,
  with where you searched. A new firm with no tags at all is refused by `file_investor`.
- Fields that came from inference rather than a stated source are not shown and not filed.
- The header names the project, the catalog the card files to and the workspace it is filed for
  (house rules §1).

## The `file_investor` bundle

Built from your working state, not from the rendered card. Send it with `dryRun: true` before the
card is shown; send it again with `dryRun: false` only after approval.

```jsonc
{
  "dryRun": true,
  "workspaceId": "…",                       // the workspace filed for (the project's); required
                                             // unless you are a platform admin
  // "scope": "workspace",                  // only when the user asked for a private copy
  "company": { "new": <company profile>, "confirmNew": false }
          // existing (shared catalog or this workspace):
          //   { "id": "…", "fill": {…}, "change": {…}, "tags": { "add": [], "remove": [] } }
          //   { "id": "…" }                    — people only
  ,
  "people": [
    { "new": <person profile>, "confirmNew": false },
    { "id": "…", "update": { "email": "…", "emailStatus": "unverified", "twitterHandle": "…",
                             "sources": { "email": "northwind.vc/team", "twitterHandle": "northwind.vc/team" } } },
    { "id": "…", "update": { "roleTitle": "Partner" }, "moveToCompany": true }   // job move to this firm
  ],
  "profileNote": { "body": "…" }            // required when the firm or any person is new
}
```

```jsonc
// company profile — no kind: file_investor files an investor
{
  "name": "Quarry Lane Ventures",
  "websiteUrl": "https://quarrylane.vc", "linkedinUrl": "https://www.linkedin.com/company/quarry-lane-ventures",
  "hqCountry": "AT", "description": "Leads pre-seed and seed rounds in deep tech companies across Central Europe.",
  "tagIds": ["…"],                          // real ids from list_tags (parents with children), plus
                                             // the id create_tag just returned for an approved new tag
  "attributes": { "leads_rounds": true, "min_check": "$250K to $1M", "max_check": "$1M to $3M",
                  "fund_size": "$50M to $200M", "notable_investments": "Robotiq, Fabrik, Lumen" }
}

// person profile — companyId is never sent; the server sets it to the card's firm
{
  "fullName": "Elena Varga", "roleTitle": "General Partner",
  "linkedinUrl": "https://www.linkedin.com/in/elena-varga", "linkedinStatus": "unverified",
  "email": "elena@quarrylane.vc", "emailStatus": "unverified",
  "twitterHandle": "elenavarga",
  // "phone": "+43 1 234 5678", "phoneStatus": "unverified"   — when a source gives one
  "locationCity": "Vienna", "locationCountry": "AT",
  "sources": {                              // one line per contact value, where it came from
    "linkedinUrl": "linked from quarrylane.vc/team",
    "email": "Hunter verifier 2026-10-09 (valid, 96)",   // Hunter's `source`, verbatim
    "twitterHandle": "linked from her LinkedIn"
  }
}
// a field the search found nothing for is omitted; never "" and never a guess
```

Reading the dry run:

- `ok: true` → show the card.
- `duplicates[].companies` (strong: same domain, LinkedIn or exact name, or a name at similarity
  0.8 or more) → the card becomes a question under **Needs your input** with the candidates.
  `confirmNew: true` only after the user says "new" (house rules §4).
- `duplicates[]` with `weak: true` (a name-only look-alike, 0.6–0.8) → shown under **Worth a look**;
  the card still goes ahead.
- `duplicates[].elsewhere` → the firm is in another workspace's private catalog. Say "already in
  Yellowtail (another workspace's list); not filed again" and drop the card.
- `problems` → fix what is yours to fix (an option string, a second single-cardinality tag, a
  `fill` on a field that already has a value, a field change on a shared firm) and dry-run again;
  otherwise show the problem on the card.
- `tagGaps` → each kind is either mapped now (go back to the tree: is there a tag the sources
  support that you missed?) or goes on the card's **Tags not stated** line with where you
  searched. A gap in `industry`, `stage` or `geography` on a new firm is rare: look again before
  you accept it.

Filing: one call per approved card, in card order. A card with approved **New tags**: call
`create_tag { kind, name, parentId? }` for each first and swap the returned ids into the bundle (a
dry run with the new ids before the real call). `created: false` → use the existing tag it returns,
and say so in the report. Refused (a kind other than industry or focus, and not a platform admin) →
file without it and list it under **Tags to ask the Yellowtail team for**. The answer's
`filedTo` says where it went and `written` gives `companyId`,
`personIds` and `peopleCreated`; report with `/investors/{companyId}` and
`/investors/people/{personId}` (house rules §11). A failed card wrote nothing; say why and carry on.
Never re-file a card that succeeded.
