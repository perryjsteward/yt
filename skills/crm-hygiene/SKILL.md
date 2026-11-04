---
name: crm-hygiene
description: Yellowtail Club — Data-health check for one Yellowtail project — deals with no primary contact, no or overdue next step, stuck, or no value; primary contacts missing email, a verified email, LinkedIn, X, phone or location; people with no firm; likely duplicates; deals on firms outside the project's interests; and plan vs record (overdue milestones, deals closing after legal close, a raising round with no timeline). Reports with counts and links, then proposes fixes as cards (next step, value, primary contact, contact details, tags). Never deletes or merges: duplicates are reported for the Yellowtail team to merge. Works on any project, including the practice. Use when someone says "clean up the CRM", "CRM hygiene", "data health", "what's missing on my deals", "audit the pipeline data", "find duplicates", "which deals have no contact", or before a round review or a hand-over.
argument-hint: "[pipeline name | --all-pipelines] [--stale 14] [--fix]"
---

# /crm-hygiene

Every other skill trusts the data: `/round-forecast` needs values and next steps, `/draft-outreach`
needs a primary contact with an email, `/find-investors` needs the catalog free of two copies of the
same fund. This skill finds what's missing or wrong, says how much of it there is, and fixes what
can be fixed from evidence — on a card, after approval. Deletes and merges are not available to
customers; duplicates are reported, not removed, and the Yellowtail team merges them across the
whole catalog (its desk plugin's `/catalog-hygiene`).

Read `${CLAUDE_PLUGIN_ROOT}/references/house-rules.md` first.

## Tools

| Tool | For | Required? |
| --- | --- | --- |
| `whoami`, `get_project`, `list_tags` | Project, `workspaceId`, `isPlatformAdmin`, interests (`interestTagIds`) named | yes |
| `list_pipelines`, `get_pipeline`, `list_deals` | Every open deal; page `nextCursor` | yes |
| `list_milestones` | Each round's timeline, for the plan-vs-record check | yes |
| `update_milestone` | Mark a milestone done (`doneOn`) or move it with the reason as `note` | on approval |
| Docs (Google Drive `read_file_content`) | A plan doc linked from the pipeline's description or a deal comment, to compare its dates | when one is linked |
| `get_deal` | Contacts (`isPrimary`), `nextStep`, `nextStepOn`, `lastActivityAt`, comments | every open deal |
| `get_person` | Each primary contact's `email`/`emailStatus`, `linkedinUrl`/`linkedinStatus`, `twitterHandle`, `phone`/`phoneStatus`, `locationCity`/`locationCountry`; their `companyId` | yes |
| `get_company` | The deal firm's tags and scope (shared catalog or a workspace's own) | yes |
| `search_companies`, `search_people` | Duplicate candidates: by name, domain, LinkedIn; `reasons` and `score` | yes |
| `update_deal` | `nextStep`, `nextStepOn`, `value`, `currency`, `expectedCloseOn` | on approval |
| `add_deal_contact` | `{ dealId, contact: { personId, isPrimary: true } }` | on approval |
| `file_investor` / `file_account` | Fill or verify a person's contact details, attach a person to a firm, tag a firm: `workspaceId`, `company: { id, tags? }`, `people: [{ id, update: { email, emailStatus, linkedinUrl, linkedinStatus, twitterHandle, phone, phoneStatus, locationCity, locationCountry, sources } }]`; `dryRun: true` first. Any member may do this on an investor in the shared catalog; an account card only touches the workspace's own | on approval |
| `create_tag` | `{ kind, name, parentId? }`: an approved **New tags** line (house rules §12). Anyone: industry (with its group) and focus; other kinds: platform admins only | on approval |
| `find_email`, `verify_email` | Hunter through Yellowtail, for a primary contact's missing or unverified email (spends the workspace's quota) | when the workspace has a key |
| Email (Gmail `search_threads`) | Evidence for a next step, or a verified email from a signature | optional |
| Web search; enrichment (Hunter, Apollo, Clay) | The contact search for a primary contact's missing fields | when filling contacts |

## Steps

1. **Orient.** `whoami`; `get_project` (interests, named with `list_tags`); the pipeline named, else
   every pipeline with status `planning` or `raising` (`--all-pipelines` adds closed ones).
2. **Read everything before judging anything.** Every open deal, `get_deal` for each, `get_person`
   for each primary contact, `get_company` for each firm. Say "N deals, P people, F firms read".
3. **Run the checks.**

   | Check | Flag if |
   | --- | --- |
   | Primary contact | none on the deal, or no contacts at all |
   | Next step | `nextStep` empty (old deals: no `**Next:**` line in comments or notes) |
   | Overdue | `nextStepOn` in the past, or `expectedCloseOn` in the past while open |
   | Stuck | `lastActivityAt` older than `--stale` days (default 14), or `stageAt` older than twice that |
   | Value | `value` empty on a deal past the first stages (can't be forecast) |
   | Contact email | primary contact with no email, or only an `unverified` one on a deal past first contact |
   | Contact details | primary contact with no LinkedIn (always a gap), or no X, phone or location on file |
   | No firm | a person on a deal with no `companyId` (fine for an angel; ask) |
   | Duplicates | a deal firm whose `search_companies` by name, domain or LinkedIn returns another firm with `domain`/`linkedin` reasons or `name_similar` ≥ 0.8; the same for each deal contact with `search_people` |
   | Outside interests | the firm carries tags of a kind the project chose, but none of the chosen ones (e.g. project Seed, firm tagged Series B only). A firm with no tag of that kind is *unclassified*, not outside |
   | Plan vs record | per round, against `list_milestones`: a milestone `overdue` and not done (did it happen? `doneOn` missing, or the date needs moving); an open deal whose `expectedCloseOn` is later than the round's `legal_close`; a pipeline with status `raising` and no milestones at all; a plan doc linked from the pipeline's description or a deal comment (a `/round-strategy` or `/close-plan` doc) whose dates disagree with the milestones, each difference quoted from both |

   On the Yellowtail Raise project, skip the interests check unless interests are set.
4. **Report** (output below) before proposing anything: the critical items first (overdue, no
   primary contact on an active deal), then attention items, then the clean count.
5. **Suggest fixes from evidence only.**
   - **Next step**: from the newest comment, the latest email thread with the contact, or a meeting
     booked — quoted with its source. No evidence → ask the user; never write a generic "follow up".
   - **Next-step date / close date passed**: propose a new date only if the evidence gives one;
     otherwise list it as a question.
   - **Primary contact**: the contact on the deal who has actually engaged (latest comment, email
     reply). Nobody on the deal → `/find-contacts` for the firm.
   - **Contact details** (house rules §5): for each primary contact with gaps, run the contact
     search for the missing fields only — LinkedIn always; email from the user's own mail (signature,
     sender → `verified`), then **Hunter** if the workspace has a key (`find_email { personId }`;
     `verify_email { email }` for an `unverified` address on a deal past first contact), filed with
     exactly the `emailStatus` and `source` it returns, then an address they publish, then a
     pattern only from a published address at the firm's domain (`unverified`, the evidence named);
     no key → say once "I searched by hand; for verified emails connect Hunter in Settings →
     Integrations". Never a role inbox (`info@`, `hello@`, `team@`, …) as a person's email: a
     primary contact whose only email is one is flagged as having no email. X, work phone, location. Each found value with its source, sent as `sources`; each miss as
     `not found — searched: <where>`. An unverified email the user's mail confirms → propose
     `emailStatus: "verified"` with the message as its source.
   - **Unclassified firm** (an investor in the shared catalog, or a firm in this workspace): a tag a
     source states for the kind the project filters on → propose it (`company: { id, tags: { add }
     }`). A tag the source supports that doesn't exist → **New tags (created on approve)** (house
     rules §12): `create_tag` first on approval; `created: false` → use the tag it returns and say
     so; refused → file without it and list it under **Tags to ask the Yellowtail team for**.
   - **No firm**: if their email domain or LinkedIn matches a firm in Yellowtail, propose attaching
     them through `file_investor` / `file_account` with `people: [{ id }]` on that firm.
   - **Value**: only a ticket a comment or email states.
   - **Plan vs record**: the timeline is the record, so a disagreement is fixed on the timeline or
     on the deal, never in the doc alone. An overdue milestone that happened → `update_milestone
     { doneOn }` from the evidence (a signed term sheet in email, a comment); one that didn't →
     ask for the new date and the reason, then `update_milestone { dueOn, note }`. A deal closing
     after `legal_close` → propose `expectedCloseOn` on or before it, or ask whether legal close
     should move. A raising round with no milestones → `/round-strategy` or `/create-pipeline` to
     set them. A doc whose dates differ → say which is right, per the milestone's history, and
     offer a fresh doc from `/close-plan`.
   - **Duplicates / outside interests**: no fix here. Duplicates are listed for the Yellowtail team
     to merge (the Yellowtail team merges them); a deal outside interests is a question for the
     user — it may be deliberate.
6. **Cards** (one per deal or person), `dryRun: true` for every `file_*` card first. A person the dry
   run won't let this workspace update: say so, and leave it for the Yellowtail team.
7. **Write** on approval: `update_deal`, `add_deal_contact`, any approved new tags (`create_tag`,
   their ids into the bundle), then the `file_*` cards. Then a
   `comment_on_deal` only where the user asks for the change to be logged.

## Cards

```
**CRM hygiene fixes** · Fernway · Seed 2026 · for Atlas Advisory

### 1. Northwind Capital (deal)
- Next step: none → **"Send model v3 to Sarah"** · due **2026-10-14** (Gmail, Sarah 8 Oct: "model by Tuesday?")
- Primary contact: none → **Sarah Lee** (replied 8 Oct; Raj never has)

### 2. Sarah Lee (Northwind Capital, shared catalog)
- Email: sarah@northwind.vc unverified → **verified** (her reply of 8 Oct, signature)
- LinkedIn: none → linkedin.com/in/sarah-lee-vc (unverified; northwind.vc/team link)
- X: none → @sarahlee_vc (linked from northwind.vc/team)
- Phone: none → +44 20 7946 0000 (verified; her signature, 8 Oct)
- Location: on file (London, GB)

### 3. Timeline: Lead term sheet (due 3 Oct, overdue)
- Done: none → **2026-10-06** (Gmail, Sarah 6 Oct: "signed term sheet attached")

**Needs your input**
- **A.** Ember Capital: next step date passed (30 Sep, "partner meeting"). Did it happen? New date?
- **C.** Lantern's expected close is 15 Jan; the round's legal close is 19 Dec. Move the deal, or the round?
- **B.** Blue Fjord is tagged Series B only; the project is Seed. Keep the deal?

Reply with any of: `approve all` · `approve 1` · `2: leave unverified` · `A: met 2 Oct, next is IC 20 Oct`
```

## Output

Display: inline — one Yellowtail widget in the chat, cards under it (`${CLAUDE_PLUGIN_ROOT}/references/display.md` §2). Yellowtail's reads draw nothing on the way; this output is the one thing shown (§ Only the answer is drawn).

```
Data health — Fernway · 2 pipelines · 41 open deals, 63 people, 39 firms read (9 Oct 2026)

Critical  · 3 overdue next steps · 2 active deals with no primary contact · 1 close date passed
Attention · 9 no next step · 6 stuck > 14 days · 7 no value · 5 primary contacts with no email ·
            4 with no LinkedIn
Catalog   · 2 likely duplicate firms · 1 likely duplicate person · 3 people with no firm
Interests · 1 firm outside, 4 unclassified
Plan      · 1 overdue milestone not done · 2 deals closing after legal close (19 Dec) ·
            the close plan doc says legal close 12 Dec, the timeline says 19 Dec (moved 3 Oct, Sam)
Clean     · 22 of 41 deals

| Deal | Issues | Suggested fix | Source |
…
Likely duplicates (for the Yellowtail team to merge)
- "Northwind Capital" (this workspace) ↔ "Northwind Capital Partners" (shared) — same domain northwind.vc
```

Link deals `/pipelines?pipeline={pipelineId}&deal={dealId}`, people `/investors/people/{personId}`,
firms `/investors/{companyId}`.

## Rules specific to this skill

- Never deletes, removes a contact, merges, or moves a deal between pipelines. Customers can't
  merge, and this skill never routes around it: duplicates go in the report for the team.
- A fix needs evidence or the user's word. "Unknown" is a valid, reported state.

## What comes next

- `/find-contacts` for deals with nobody to email; `/update-deal` for anything left as a question.
- `/round-forecast` or `/pipeline-review` once the data is clean.
