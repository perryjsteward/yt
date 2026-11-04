---
name: create-investor
description: Yellowtail Club — Research one investor firm or a short list in depth (thesis, stages, cheque sizes, sectors, geography, fund size and vintage, recent deals, website, LinkedIn, HQ) plus the partners worth contacting with their LinkedIn, work email, X, phone and location, proposing any missing industry or focus tag. Shows each firm as a card with every field sourced and every contact detail marked unverified or verified, and on approval files it to Yellowtail's shared investor catalog for the project's workspace. Also enriches a firm Yellowtail already has (fill gaps, correct, retag, add people). Needs no round or pipeline. Use when someone says "add [firm]", "create these investors", "research and add", "put Kiln Climate in Yellowtail", "enrich Northwind", "fill in the gaps on these funds", or replies `create 2 7` to /find-investors.
argument-hint: "<firm name, website or LinkedIn URL — or a list / CSV of them>"
---

# Create investor

Turns a firm name into a Yellowtail record anyone can trust: researched field by field, every
value with a source, empty where there was none. Investors are shared: the card files to
Yellowtail's **shared catalog**, which every customer reads, for the project's workspace. A guessed
field is a bug for all of them.

Read `${CLAUDE_PLUGIN_ROOT}/references/house-rules.md` first. Research, the card and the bundle are all in
`${CLAUDE_PLUGIN_ROOT}/references/research-an-investor.md`; follow it exactly. Worked examples:
`${CLAUDE_PLUGIN_ROOT}/references/examples-investors.md` § create-investor.

## Three cases, one card

| The firm is… | The card is… | `company` in the bundle |
| --- | --- | --- |
| not in Yellowtail | **new investor**: full profile + people + profile note | `{ "new": …, "confirmNew": … }` |
| in the **shared catalog** or **this workspace** | **fill gaps**: Fill / ⚠ Change / Tags lines + people | `{ "id", "fill", "change", "tags" }` |
| only in **another workspace's** private catalog (the dry run's `duplicates[].elsewhere`) | no card: "already in Yellowtail, not filed again" | — |

Any member may fill, change and retag a firm in the shared catalog, and add or update its people;
every change is in the firm's history with who made it. `scope: "workspace"` (a private copy in this
workspace) only when the user asks for it.

**Filing needs no round.** A firm is filed to the catalog, not to a pipeline: never set up a round
or a pipeline first, and never ask for one. Putting the firm on a round is a separate step, offered
afterwards (`/add-to-pipeline`).

## Tools

| Tool | For | Required |
| --- | --- | --- |
| `whoami` | project, `workspaceId`, `key.canWrite`, `isPlatformAdmin`, `toolsets.enabled` (is `enrichment` there?) | yes |
| `get_company_kind` | investor field keys, options, which tag kinds apply and their cardinality | once |
| `list_tags` | real tag ids, one kind at a time; near-duplicates before proposing a new tag; the industry groups for `parentId` | per kind |
| `create_tag` | `{ kind, name, parentId? }`: an approved **New tags** line, created before its card is filed. Anyone: industry (with its group as `parentId`) and focus; other kinds: platform admins only | on approval |
| `search_companies` | dedupe by `q`, `domain`, `linkedinUrl` (`kind: "investor"`) — before and after research | yes |
| `get_company` | an existing firm's fields, tags and people, before proposing changes | existing firms |
| `search_people` | dedupe each person by `q` and `linkedinUrl`; `companyId` alone lists everyone at the firm | yes |
| `domain_search`, `find_email`, `verify_email` | Hunter, through Yellowtail (`enrichment` toolset, the workspace's own key): the firm's people and pattern, one person's address, a check on an address found elsewhere. Spends quota | when the workspace has a key |
| `file_investor` | `dryRun: true` before the card; `dryRun: false` after approval, with `workspaceId` | yes |
| Web search / fetch | research only | yes |
| Apollo, Clay (own connectors) | phones, emails; `verified` only when the tool reports the address deliverable | if connected |

## Steps

1. **Orient.** `whoami`. `key.canWrite` false → say the connection is read-only; you can still
   research and show the cards, as a checklist. Note the project and its `workspaceId`: every card
   is filed for it (house rules §4). Note whether `enrichment` is in `toolsets.enabled`.
2. **Read the input.** One firm, a list, a CSV, or rows handed over from `/find-investors` (reuse what
   it already read). A CSV: say in one line how you read the columns.
3. **Triage a list before researching** (over ~5 firms). Yellowtail searches only, no web: "12
   firms: 7 already in Yellowtail, 5 new. Research the 5 new ones? I can also fill gaps on the 7
   (2 have no cheque sizes) and add partners." Wait for the answer.
4. **Resolve each firm** with `search_companies` by every key you have. A `domain`, `linkedin` or
   `name_exact` reason, or `name_similar` at 0.8 or more, is a match; a name-only look-alike below
   0.8 is not, and is mentioned. Ambiguous → a question under **Needs your input**, not a card.
5. **Research** per research-an-investor.md: pin the firm, **re-search Yellowtail with the domain
   and LinkedIn you found**, then the site, a second source, the people. For an existing firm,
   `get_company` first and research only what is empty or looks wrong.
6. **People.** Two to five decision-makers for the project's stage and sector, as `/find-contacts`
   does (see its SKILL.md for ranking). `search_people` with `companyId` first: people already there
   are `(existing)`, and only a new contact detail or title is proposed for them. **Run the contact
   search for every person** (research-an-investor.md § Contact search; house rules §5): LinkedIn,
   work email, X, work phone, location, each with status and source or `not found — searched:
   <where>`. The firm's website, LinkedIn page and X handle likewise.
   - **Hunter first** (the workspace has a key): `find_email { fullName, domain }` for each person
     you chose, then `verify_email { email }` on its answer or on an address from the firm's site;
     one `domain_search { domain, seniority: ["senior", "executive"] }` per firm while you are still
     choosing. Put each address on the card with **exactly** the `emailStatus` Hunter's answer
     gives, and its `source` in `sources.email`. Never call it for people you won't propose.
   - **Then by hand**: an address the person publishes, then a pattern evidenced by a published
     address at the firm's domain, `unverified`. No key (the tools refuse) → say once: "I searched
     by hand; for verified emails connect Hunter in Settings → Integrations (lemlist is being
     considered)."
   - **Never a role inbox as a person's email** (`info@`, `hi@`, `hello@`, `contact@`, `team@`,
     `office@`, `invest@`, `media@` and the like): it can go in the firm's profile note, never on a
     person. A work email on a personal mailbox is dropped before the card (the server refuses it).
7. **Tags: every applicable kind, answered.** Read the whole tree (`list_tags` per kind, industry
   with its children) and map each sourced statement to the **most specific existing tag plus its
   parent** (research-an-investor.md § Firm — tags: "therapeutics" is Biotech + Health, "chips" is
   Semiconductors + Robotics & hardware). Each kind `get_company_kind` lists ends up with tags or on
   the card's **Tags not stated** line with where you searched; none is skipped. A sector with no
   tag under any spelling or synonym is checked against the gate in house rules §12, which is
   strict: it passes → **New tags (created on approve)** with kind, name, the group for an
   industry tag, the source sentence and the near misses; it fails → the nearest existing tag
   (usually the parent) and the word in the description.
   - **Who may create which.** Anyone: an `industry` tag under its parent group, or a `focus` tag.
     Platform admins (`whoami.isPlatformAdmin`): any kind. Near-duplicates are refused and answered
     with the existing tag, which you then use.
   - **Alongside the parent, never instead.** A new specific tag goes on the firm **next to** its
     broad parent (keep "AI & data" when adding "Industrial AI", and add the parent if the firm
     lacks it). Other firms don't carry the new tag yet, so matching must never get worse for them.
   - **A kind the user can't create** (stage, geography, investor type, … for a customer): no
     "created on approve" line. List it on the card, and in the report, under **Tags to ask the
     Yellowtail team for**, with the firm and the source, and suggest the user mentions it to the
     team. Nothing notifies them automatically.
8. **Profile note.** Required for every new firm or person; also for any enrichment. Where each value
   came from, what wasn't found, and the facts with no field (fund vintage, recent deals).
9. **Preflight.** Build the bundle (with `sources` on every person) and call `file_investor` with
   `dryRun: true` and `workspaceId`. Strong `duplicates` → a question, with the candidates.
   `weak: true` look-alikes → **Worth a look** on the card; it goes ahead. `elsewhere` → drop the
   card: already in Yellowtail. `problems` → fix and dry-run again, or show on the card. Only cards
   that pass, or wait on a question, are shown. A proposed new tag has no id yet: dry-run without
   it, and again with its id once it is created. `tagGaps` → look again for each kind before
   accepting it, then put what is left on **Tags not stated**. A new firm with no tags is refused.
10. **Propose.** The investor card format in research-an-investor.md. **At most 5 cards per round**;
    say how many remain.
11. **Take the reply** (house rules §2), plus `1: not Tomas` (drop a person), `1: no emails`,
    `1: no new tag`, `2: keep HQ as UK`, `1: keep it private` (adds `scope: "workspace"`). Re-run
    the dry run on changed cards and re-show only those.
12. **File.** For each approved card, in order: its approved **New tags** first, `create_tag
    { kind, name, parentId? }` each, the returned ids swapped into the bundle and dry-run again.
    `created: false` → use the existing tag it returned and note it for the report. Refused → drop
    it from the bundle and list it under **Tags to ask the Yellowtail team for**. Then
    `file_investor` with `dryRun: false`, the same `workspaceId`, and `confirmNew: true` only where
    the user said "new". Duplicates on the real call (someone added it since) → stop that card and
    ask. A failure wrote nothing; carry on.
13. **Report** (house rules §11): one line per card, **new** marked, where it was filed (`filedTo`),
    links `/investors/{companyId}` and `/investors/people/{personId}`, tags created or reused; then
    failures; then **Tags to ask the Yellowtail team for**.
14. **Backfill a new tag (platform admins only).** After a tag is created, offer once: "Want me to
    look for other firms that should carry Industrial AI?" On yes, it is card first, like everything
    else:
    - **Candidates** from firms carrying the parent tag (`list_companies` with `kind: "investor"`
      and `tags` the parent's id; a parent filter includes its children) and from the research
      notes you already have (`list_project_notes` with the tag's words as `query.q`).
    - **Check each one** against the firm's own description and portfolio (`get_company`, its
      `notable_investments`, its site when they are thin). A tag goes on only with a sentence that
      supports it; "parent tag matches" is not a reason.
    - **Propose retag cards in batches of about 15**: one line per firm, `tags { add: [new id] }`
      (the parent stays), with the reason and its source. On approval, `file_investor` with
      `company: { id, tags }` and the project's `workspaceId`, as for any retag. Then the next batch,
      until the candidates run out or the user stops.

    Customers don't get this offer: they retag the firms they work on as they file them.

## Rules that are easy to break

- **Not one field more than the sources support.** `leads_rounds` empty on silence, never false.
  `climate_investments` only if you counted. A cheque bucket only from a stated range.
- **The description is read by every customer.** Factual, from the firm's own words. Nothing a
  user told you in confidence.
- **Contacts are a required search.** Every person shows LinkedIn, email, X, phone and location, or
  `not found — searched: …`, and every value has its `sources` line. Work details only. A
  pattern-built email only from a published address at the same domain, `unverified`, with that
  address named. A Hunter answer is filed with exactly its `emailStatus`, never upgraded; `verified`
  otherwise only from the user's own material or their say-so.
- **No vaguer tag substituted, and no gap left silent.** The most specific tag that exists, plus
  its parent; every applicable kind tagged or on **Tags not stated**. A missing tag is proposed
  only if it passes the §12 gate (a market, used by other firms, led with by the thesis, one level
  under a group, one per card); never a near-duplicate of a tag that exists.
- **No `confirmNew: true` on your own judgement**, and never because a look-alike was only `weak`
  (a weak one doesn't need it).
- **No private copies by default.** `scope: "workspace"` only when the user asks.
- **No deletes.** A duplicate or a stale record is reported, not removed.

## Output

Display: inline — one Yellowtail widget in the chat, cards under it (`${CLAUDE_PLUGIN_ROOT}/references/display.md` §2). Yellowtail's reads draw nothing on the way; this output is the one thing shown (§ Only the answer is drawn).

The cards, then after filing:

```
Filed to the shared catalog for Atlas Advisory (project Voltframe):
1. Quarry Lane Ventures — **new** · /investors/…  · Elena Varga **new**, Tomas Novak **new** ·
   tag "Industrial automation" (industry, under Robotics & hardware) created
2. Northwind Capital — filled LinkedIn, min cheque, leads rounds; HQ → Ireland; + Sweden, Norway
   (the change is in Northwind's history under your name)
3. Lantern Ventures — Mira Osei **new** · /investors/people/…

Tags: "Grid flexibility" (industry) already existed as "Grid flexibility services"; used that one.
"Industrial automation" was added next to its parent, Robotics & hardware.

Tags to ask the Yellowtail team for (geography tags are theirs; mention it to them):
- "Estonia" (geography) for Quarry Lane Ventures — quarrylane.vc: "Tallinn and the Baltics"

Hunter: 3 lookups (1 domain search, 2 verifications) on Atlas Advisory's key.
```

## What comes next

`/add-to-pipeline` for the firms just filed (if they came from `/find-investors`, name the rows):
this is where a round comes in, never before. `/add-to-list` to track their people,
`/investor-brief` before a first meeting.
