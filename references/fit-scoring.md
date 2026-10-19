# Fit scoring

How `/find-investors`, `/investor-brief`, `/call-prep` (through `meeting-prep-brief.md`) and
`/investor-prospecting` say how well an investor fits.
The score is a **band with its reasons**, never a number: "Strong — leads seed climate rounds in the
UK, $1M–$3M cheques, led two grid-software seeds this year." A founder can check every word of that;
"87/100" invites them to trust arithmetic on guesses.

## What we score against

Gather the round once per run, before scoring anything:

| What | From |
| --- | --- |
| Sector, stage, geography | `get_project` → `interestTagIds`, named via `list_tags` (the project's interests) |
| What the startup does, its competitors | the project's `description`; the user; the startup's own site when the description is thin. Competitors: ask once if not obvious, or research 3–5 and say they are your list |
| Round size, round type | No pipeline is needed. In this order: the open round's `target` and `currency` (`list_pipelines`, `projectId`; several open rounds → ask which), else a size the project's `description` states, else **one** question ("Roughly how much is the round, and in what currency?"). No answer → leave cheque fit and lead appetite out and say so. Round type from the round's `attributes` (`round`) or the description |
| Lead or follow wanted | the user, or the round's notes. Unknown → treat a lead as a plus, not a requirement |

Cheques are compared in the round's own currency. A firm's cheque range in another currency is
compared roughly and said ("£250K–£1M against a €2M round"), never converted to a figure.

Anything missing is said once at the top of the shortlist ("No round size on the pipeline, so cheque
fit isn't scored"), and that factor is left out rather than guessed.

## Evidence, not tag overlap

Tags say what a firm was filed as; they don't say what it is doing. A firm whose stage, industry and
geography tags all match the project's interests has passed the filter, nothing more: every firm in
the project's slice does. Rank on **evidence**, read in this order:

1. **Recent deals and signals**: a dated round it led or joined (a signal "Invested in…" / "Led…",
   a portfolio announcement, trade press), a fund close, a partner's post saying what they want now.
   This is the strongest evidence and the only evidence of activity.
2. **Notable investments** (`notable_investments`, the portfolio page): companies like the client,
   at its stage, in its market. Also where conflicts show up.
3. **The description** and the firm's own thesis page: what it says it backs, in its own words.
4. **Tags and attributes**: the filter, and the cheque and lead fields. They confirm; on their own
   they never make a firm Strong.

Each evidence line on a row is one of 1–3 with its date and source. A row whose only evidence is
"tags match" is at most **Possible**, and says so ("tags only; no dated deal found").

## The factors

Each factor is **match**, **partial**, **miss** or **unknown**, with one evidence line and its source.
Unknown is not a miss; say it.

| Factor | Match | Partial | Miss (and is it a blocker?) |
| --- | --- | --- | --- |
| **Stage** | firm's `stage` tags or stated stages include the round | adjacent stage (does seed, round is pre-seed) or "opportunistic" | no overlap — **blocker** |
| **Cheque vs round** | `min_check`–`max_check` brackets a sensible ticket: ~20–50% of the round for a lead, ~5–25% for a follower | the range touches it at one end | cheque floor above the whole round, or ceiling under 5% of it |
| **Sector / thesis** | a source names the project's industry, or the focus is Generalist and the stage matches | an adjacent sector, or a portfolio pattern with no stated thesis | thesis excludes it ("no hardware", `software_only: true` for a hardware company, a negative signal) — **blocker** |
| **Geography** | invests where the startup is based (`geography` tags, stated) | its region but not the country named, or "global" with no deals there | explicit exclusion — **blocker** |
| **Recent activity** | deals in the project's stage and sector in the last 18 months, or a Yellowtail signal in the last 90 days pointing this way | active, but not in this stage or sector | last known deal more than 18 months ago → flag **may be inactive** (caps the band at Possible); "not making new investments" stated → **blocker**. No deal date known at all is **unknown**, not inactive |
| **Lead vs follow** | `leads_rounds: true` when a lead is wanted; either when a follower is wanted | unknown appetite to lead | follow-only when the round has no lead yet |
| **Portfolio conflict** | none found (say what you checked: the portfolio page, `notable_investments`, deal signals) | an adjacent company in the portfolio | a direct competitor of the client in the portfolio — **blocker** unless the user says otherwise. Shown first on the row, ⚠ |
| **Fund vintage / dry powder** | a fund closed in the last ~3 years, or a fresh close announced | a fund 4–5 years old (likely reserving for follow-ons) | the last fund is 6+ years old and nothing new — say "probably not deploying" |

Recent signals in Yellowtail (`search_signals`) count twice: as recent activity, and because a
**negative** tag on a recent signal ("− Pre-seed", "− Hydrogen") is the firm saying no. A negative
signal on the project's stage or sector is a blocker until a later signal says otherwise.

## Bands

| Band | Rule |
| --- | --- |
| **Strong** | Stage, sector and geography match; a dated deal or signal in the last 18 months matches; no blocker; cheque fits or is unknown |
| **Good** | Stage matches; no blocker; at most two partials or unknowns among the rest |
| **Possible** | No blocker, but several partials or unknowns, or flagged **may be inactive**, or tags are the only evidence: worth a look, not a first wave |
| **Poor** | Any blocker. Shown only when the user asked about that firm, or in a "Ruled out" list with the blocker |

Within a band, order by: the most recent matching deal or signal, then how close its portfolio
companies are to the client, then leads-when-a-lead-is-wanted, then cheque fit. A "new"
firm (not in Yellowtail) is scored on the same factors from its web research; mark the factors you
couldn't verify as unknown rather than letting a thin profile look like a clean one.

## How it reads

One line for the band and the strongest reason, then up to three evidence lines, each with its
source. Blockers and warnings come first.

```
3. Northwind Capital — Strong · in Yellowtail
   Leads UK pre-seed and seed climate rounds; $250K–$1M cheques fit a $2M round as lead
   · Signal 2 Oct 2026: "now leading pre-seed climate in UK + Nordics" (Yellowtail)
   · Led Gridly seed, Jun 2026 (northwind.vc/portfolio)
   · Fund III, $120M, closed Jan 2025 (press release)
```

```
9. Halcyon Ventures — Poor · in Yellowtail
   ⚠ Portfolio conflict: backs Heatloop, a direct competitor (halcyon.vc/portfolio)
```

```
14. Brightwater Ventures — Possible · in Yellowtail · ⚠ may be inactive
   Last known deal: Mar 2025 (brightwater.vc/portfolio), over 18 months ago; no Yellowtail signal since
```

Never present a band as more certain than its evidence: three unknowns and a stage match is
**Possible**, however famous the firm.

## Snapshot mode

`/find-investors --snapshot` is the short list shown to a prospective client before they sign: it
must stand on its own. Exactly **20** firms, Strong first, then Good; Possible only to reach 20, and
never a Poor or a **may be inactive** row. **Existing backers are excluded**: firms on any of the
project's pipelines with status `won`, firms the project description names as investors, and any
the user names. Fewer than 20 qualify → show what qualifies and say why there are fewer; never pad.
