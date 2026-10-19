---
name: find-contacts
description: Yellowtail Club — Find the people worth contacting at one investor firm — partners, principals, platform or IR — ranked by fit with the active project's stage and sector, each with title, background, recent investments and posts, and a contact search — LinkedIn, work email (Hunter through Yellowtail, then a published address, then a pattern only when the firm's own addresses show it; never a role inbox), X, work phone and location — each sourced and marked unverified or verified, or not found with where it looked. Checks who Yellowtail already has; a firm it lacks is looked at first and offered to be added if it invests. Then proposes the new people and details as one card and files it on approval. Use when someone says "who should we talk to at [firm]", "find contacts at", "find the partners at", "get me emails for Northwind", "who covers climate at", or when /add-to-list or /add-to-pipeline meets a firm with no contacts.
argument-hint: "<firm name, website or Yellowtail firm> [focus, e.g. \"climate partner\"]"
---

# Find contacts

One firm, the right people at it. "Right" means the people who would champion **this** project's
round: the partner whose sector, stage and recent deals look like it, not the managing partner by
default, and always a named person, never a role or shared inbox. The output is a ranked list with
reasons, and one card that adds the new ones to Yellowtail.

Read `${CLAUDE_PLUGIN_ROOT}/references/house-rules.md` first. People research follows
`${CLAUDE_PLUGIN_ROOT}/references/research-an-investor.md` § People; the card and bundle are there too. Worked
example: `${CLAUDE_PLUGIN_ROOT}/references/examples-investors.md` § find-contacts.

## Tools

| Tool | For | Required |
| --- | --- | --- |
| `whoami` | project, `workspaceId` | yes |
| `get_project`, `list_tags` | the project's interests, named (what "fit" is measured against) | yes |
| `search_companies` | the firm, by `q`, `domain`, `linkedinUrl` (`kind: "investor"`) | yes |
| `get_company` | the firm's scope, tags and the people Yellowtail already has | yes |
| `search_people` | `companyId` alone lists everyone at the firm; `q` / `linkedinUrl` per new person, to catch job moves | yes |
| `search_signals` | recent signals by `companyId` / `personId`: who is saying what | yes |
| `file_investor` | `dryRun: true` before the card, then file with `workspaceId` | to write |
| Web search / fetch | team page, LinkedIn, posts, portfolio announcements | yes |
| `domain_search`, `find_email`, `verify_email` | Hunter, through Yellowtail (`enrichment` toolset, the workspace's own key in Settings → Integrations). Each call spends the workspace's quota | when the workspace has a key |
| Apollo, Clay (own connectors) | phones, emails; `verified` only when the tool reports the address deliverable | if connected |

## Steps

1. **Orient.** `whoami`; `get_project` → interests, named with `list_tags`. Note the round's stage
   and sector in one line; it is what you rank against.
2. **The firm.** `search_companies` by every key (name, domain, LinkedIn), with and without
   `kind: "investor"`. Ambiguous → ask. Found, but not an investor (an account, a client) → stop and
   say what it is filed as; this skill only adds people at investor firms.
   **Not in Yellowtail → don't stop there. Look at the business first**: its website (home, about,
   portfolio or team page) and LinkedIn, enough to say what it does. Spend nothing else yet: no
   people research, no Hunter calls. Then say which of three it is, with the evidence:
   - **It invests** (a VC, angel network, CVC, family office or accelerator that writes cheques,
     shown by a portfolio, a fund, "we invest in…") → "**<Firm> isn't in Yellowtail yet.** It looks
     like an investor: <one line, with the page>. Add it to Yellowtail and then find contacts there?
     `yes` / `no`". On `yes`, research the firm as `/create-investor` does (profile, every applicable
     tag, house rules §12), then steps 3–7 for its people, and put the new firm and its people on
     **one** card, filed together on approval. Nothing is written on the `yes`; the card is still
     the approval.
   - **It doesn't look like an investor** (an agency, a startup, a service firm) → say so plainly,
     never file it as an investor to make the command work, and ask what was meant, naming the
     command that fits: "**<Firm> isn't in Yellowtail, and it doesn't look like an investor**: it's
     <what it does> (<page>). Did you mean:
     - `/find-investors` — investors *for* <Firm>, if it's raising (or `/create-project` to set it
       up as a client first);
     - something else? If <Firm> does invest and I've missed it, say so and I'll add it as an
       investor and find contacts."
     The last line is the override: on the user's say-so, carry on as for an investor, with what
     they told you as the firm's evidence.
   - **Can't tell** (a holding company, a site that says little) → give what you found and ask the
     same question as for an investor.
3. **Who Yellowtail has.** `get_company` and `search_people` (`companyId`). List them with title and
   which contact details they carry.
4. **Research the team.** Team page first, then each candidate's LinkedIn, then what they publish.
   For each candidate: current title at this firm, city, background in a line, 2–3 investments they
   led or sit on the board of (with dates), a recent post or talk that shows their focus.
   `search_people` by name and LinkedIn URL: someone on file at another firm is a possible **job
   move** (`moveToCompany`), never assumed.
   **With Hunter**, start the candidate list with one `domain_search { domain, seniority:
   ["senior", "executive"], limit: 20 }` on the firm's website domain: it lists the people Hunter
   knows there with title, LinkedIn and address, and the firm's address `pattern`. Check each against
   who Yellowtail has (step 3) and the team page; Hunter's list is a lead, the team page is what says
   they still work there.
5. **Contact search, for every candidate** (research-an-investor.md § Contact search; house
   rules §5). Not optional, and done before ranking so the card is complete:
   - **LinkedIn**: web search `"<full name>" "<firm>" site:linkedin.com/in`, the team page's links,
     their own site. Always.
   - **Work email**, in this order:
     1. **Hunter, through Yellowtail, first** when the workspace has a key: `find_email { fullName,
        domain }` (or `{ personId }` for someone on file) for the named partner, or the address
        `domain_search` already listed, then `verify_email { email }` on it. It goes on the card
        with **exactly** the `emailStatus` Hunter's answer gives, and its `source` line as the
        source (`sources.email`).
     2. **A published address of their own**: the team or contact page naming them, their site,
        a press release. With a key, `verify_email` it.
     3. **A pattern, by hand**: only if a real address at the firm's domain is published, filed
        `unverified` with "pattern from <address> (<page>)".

     No key (the tools refuse: "Hunter is not connected for this workspace…") → steps 2 and 3, and
     say once in the run: "I searched by hand; for verified emails connect Hunter in Settings →
     Integrations (lemlist is being considered)."
   - **Never a role inbox.** `info@`, `hi@`, `hello@`, `contact@`, `team@`, `office@`, `invest@`,
     `media@` and the like are never a person's email, and never the evidence for a pattern. A
     firm's general inbox can be noted on the firm (its profile note), not on the person.
   - **X handle**, **work phone**, **location** (city, country).
   Each value carries its status and source; each miss is `not found — searched: <where>`.
   `verified` only from the user's own material, their say-so, or Hunter's answer saying so (its
   `valid` at 90 or more). Never a personal address or number: `file_investor` refuses a work email
   on a personal mailbox, so drop it before the card.
6. **Rank** by fit with this round: sector and stage of the deals they personally led > a stated
   personal focus > recent deals > seniority (can they get a deal to IC?). The top row is the partner
   most likely to take this round, and the card says why. Platform / IR people go last,
   flagged "routes intros". Analysts only if the firm says they source deals.
7. **Signals.** `search_signals` with `companyId`, last 180 days: a partner who said something
   relevant goes up, and the signal is quoted.
8. **Card.** One `file_investor` card for this firm (research-an-investor.md § The investor card):
   `company: { "id" }` plus the people. New people with full profiles; existing people only with the
   contact details or title that are new. Any member may add or update people at a firm in the
   shared catalog. `dryRun: true` with `workspaceId` (the workspace filed for) first; strong
   duplicates → a question; `weak: true` look-alikes → **Worth a look**, not blocking. This skill
   doesn't change the firm's profile or tags; a gap you notice goes to `/create-investor`.
9. **Take the reply, file, report** as in house rules §2 and §11. `1: only Sarah and Raj`,
   `1: no emails` are normal replies.
   Write every field the search found: `linkedinUrl`, `linkedinStatus`, `email`, `emailStatus`,
   `twitterHandle`, `phone`, `phoneStatus`, `locationCity`, `locationCountry`, and `sources`
   (`{ email, phone, linkedinUrl, twitterHandle }`, one line each for the values you send) (a new
   person's `new` profile; an existing person's `update` with only what is new). The report adds
   how many Hunter calls the run made.

## Output

Display: inline — one Yellowtail widget in the chat, cards under it (`${CLAUDE_PLUGIN_ROOT}/references/display.md` §2). Yellowtail's reads draw nothing on the way; this output is the one thing shown (§ Only the answer is drawn).

```
**Contacts at Northwind Capital** for Voltframe (Seed · Climate Tech · Grid management · UK)
In Yellowtail already: Sarah Lee, Partner (LinkedIn, no email)

Ranked
1. **Raj Patel** — Partner · **new**
   Ex-Halcyon; led Gridly seed (Jun 2026) and Voltgrid pre-seed (Jan 2026)
   Signal 2 Oct 2026: "Raj Patel has joined Northwind as a Partner"
   LinkedIn linkedin.com/in/raj-patel-vc (unverified; northwind.vc/team link)
   Email raj@northwind.vc (verified; Hunter verifier 2026-10-10 (valid, 96))
   X @rajpatel (linked from his LinkedIn)
   Phone +44 20 7946 0000 (unverified; office line, northwind.vc/contact)
   Location London, GB (LinkedIn)
2. **Sarah Lee** — Partner · (existing)
   Leads the climate practice (northwind.vc/team); posts on grid flexibility monthly
   LinkedIn on file · Location London, GB (on file)
   Add: email sarah@northwind.vc (unverified; listed on northwind.vc/team)
   X not found — searched: northwind.vc/team, LinkedIn · Phone: office line as above
3. **Ola Berg** — Platform Lead · **new** · routes intros (northwind.vc/founders)
   LinkedIn linkedin.com/in/ola-berg (unverified; web search, firm and title match)
   Email ola@northwind.vc (unverified; pattern from sarah@northwind.vc, northwind.vc/team)
   X not found — searched: northwind.vc/founders, LinkedIn · Phone not found — searched: northwind.vc/contact
   Location Stockholm, SE (LinkedIn)

---

### 1. Northwind Capital — people (shared catalog, filed for Atlas Advisory)
- **People** · Raj Patel — **new** … · Sarah Lee (existing): add email … · Ola Berg — **new** …
- **Profile note** · 3 sources

Reply with any of: `approve all` · `1: only Raj and Sarah` · `1: no emails` · `reject 1`
```

## What comes next

`/add-to-list` to track these people, `/add-to-pipeline` with the top one as primary contact, or
`/draft-outreach` to the person ranked first.
