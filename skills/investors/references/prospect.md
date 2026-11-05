# Investor prospecting

The `prospect` action of `/investors`. Arguments: `<segment, e.g. "UK climate pre-seed funds"> [max firms, default 30]`.

`/investors find` asks "who fits this round?". This asks "who are we missing?": it maps a segment
on the web, skips everything Yellowtail already has, and turns the rest into researched records in
the shared investor catalog, where every future round (in this workspace and every other) can find
them. It is
web-first because its whole point is what Yellowtail doesn't have yet.

Read `${CLAUDE_PLUGIN_ROOT}/references/house-rules.md` first. Research, the card and the bundle:
`${CLAUDE_PLUGIN_ROOT}/references/research-an-investor.md`. Filing follows `${CLAUDE_PLUGIN_ROOT}/skills/investors/references/research.md` from its
step 4 on. Worked example: `${CLAUDE_PLUGIN_ROOT}/references/examples-investors.md` § investor-prospecting.

## Tools

| Tool | For | Required |
| --- | --- | --- |
| `whoami` | the workspace filed for, `key.canWrite`, `isPlatformAdmin`, whether `enrichment` is in `toolsets.enabled` | yes |
| `list_tags` | turn the segment into real tags (stage, industry, geography, investor_type, focus) | yes |
| `create_tag` | `{ kind, name, parentId? }`: an approved **New tags** line, before the cards that need it. Anyone: industry (its group as `parentId`) and focus; other kinds: platform admins only | on approval |
| `list_companies` | how much of the segment Yellowtail already has (`kind`, `tags` by kind, `workspaceId`) | yes |
| `search_companies` | triage each web find by `q`, `domain`, `linkedinUrl` (`kind: "investor"`) | yes |
| `get_company_kind` | investor field keys and options | once |
| `search_people` | dedupe people on each card | yes |
| `file_investor` | `dryRun: true` per card, then file with `workspaceId` | to write |
| Web search / fetch | discovery, research, and each person's contact search | yes |
| `domain_search`, `find_email`, `verify_email` | Hunter through Yellowtail, for the people on the cards only (it spends the workspace's quota) | when the workspace has a key |
| Apollo, Clay (own connectors) | phones, emails | if connected |

## Steps

1. **Orient.** `whoami`. The workspace is the active project's `workspaceId`; a user in several
   workspaces who didn't say → ask which. Every card files to the shared catalog **for** it (house
   rules §4).
2. **Pin the segment.** Map it onto tags with `list_tags`: "UK climate pre-seed funds" → stage
   Pre-seed · focus Climate Tech · geography United Kingdom · investor_type Early-stage VC. Say the
   mapping in one line. A part with no tag ("DACH") → list the countries it means and say so. A part
   that is a real classification with no tag (after checking case, plural and synonyms) is proposed
   under **New tags (created on approve)** (house rules §12) with the triage report, so it exists
   before the cards that need it. Industry (with its group from `list_tags { kind: "industry" }`)
   and focus tags: `create_tag` on approval, by anyone; `created: false` → the existing tag it
   returns is used, and you say so. Any other kind is the Yellowtail team's unless you are a
   platform admin: list it under **Tags to ask the Yellowtail team for**, and search the segment
   without it.
3. **What Yellowtail has.** `list_companies` with `kind: "investor"`, `tags` grouped by kind and
   `workspaceId`; report `meta.totalMatches`. A tag filter includes the tag's children ("Europe"
   finds firms tagged a European country too). (For a customer this read is also held to the active
   project's interests, so treat it as a floor, not a census.)
4. **Discover on the web.** Fund-close announcements, "most active <segment> investors" round-ups,
   accelerator and government co-investment lists, the investors on recent rounds in the segment,
   association member directories (BVCA, Invest Europe, regional angel networks). Collect name +
   website + one line of why each is in the segment. Stop at the max (default 30) and say there is
   more if there is.
5. **Triage before researching.** `search_companies` for every find, by `q` and `domain`. A
   `name_similar` match below 0.8 with no `domain` or `linkedin` reason is not a match: the firm is
   new, with the look-alike mentioned. Report the split and stop:

   > "UK climate pre-seed funds": 34 firms found. **19** already in Yellowtail, **12** new, **3**
   > unclear (two "Ember"s, one site down). Research the 12 new ones? I can also fill gaps on the
   > 19 (3 have no cheque sizes). Five cards per round.

   Firms already in Yellowtail are **skipped**, not re-proposed: list their names under "Already in
   Yellowtail" so the user can see the map is complete. Unclear ones are questions.
6. **Research and propose** the approved set, five cards per round, exactly as `/investors research`:
   full research per research-an-investor.md, the people worth contacting with the **contact
   search done for each** (LinkedIn, work email, X, work phone, location, or `not found — searched:
   …`; house rules §5), any **New tags** a firm's sources call for (§12), profile note, dry run
   with `workspaceId`, the investor card. A dry run that puts the firm under `elsewhere` (another
   workspace's private catalog) drops the card: already in Yellowtail.
   Order rounds by fit with the segment (strongest evidence of activity first), so a user who
   stops after one round got the best five.
7. **Take the reply, file, report** (house rules §2, §11). After each round: "8 of 12 filed; next 4?"

## Rules that are easy to break

- **No research before the triage answer.** Mapping 40 firms on the web before the user has seen
  that 25 are already there wastes their time and yours.
- **A segment list is a lead, not evidence.** "Top 50 climate VCs" puts a firm on the triage list;
  the firm's own site is what puts tags on its card.
- **Skip means skip.** A firm already in Yellowtail is not filed again under a slightly different
  name. If research finds the domain or LinkedIn matches, it's a skip (or a fill-gaps card, if the
  user asked for that).
- **A person line without all five contact fields is unfinished.** Prospecting at volume is not a
  reason to skip the LinkedIn search or the email order.
- Everything in `/investors research`'s rules applies to every card, including filing an approved new
  tag first.

## Output

Display: inline — `show_shortlist` with every row, grouped as below, `actions` for the replies; one or two lines after it. The block below is what goes on the card, and the text when no card can be drawn (`${CLAUDE_PLUGIN_ROOT}/references/display.md` §2). Yellowtail's reads draw nothing on the way; this output is the one thing shown (§ Only the answer is drawn).

The triage report (step 5), then card rounds, then after each round:

```
Filed to the shared catalog for Atlas Advisory — "UK climate pre-seed funds", round 1 of 3:
1. Kiln Climate — **new** · /investors/… · Hana Ito **new**
2. Fernway Capital — **new** · /investors/… · Dan Okafor **new**, Priya Shah **new**
…
7 new firms still to research. Next five?
```

## What comes next

`/investors find` for a live round now that the segment is mapped, or `/investors contacts` on the firms
that were filed with no people.
