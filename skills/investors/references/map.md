# /investors map

The `map` action of `/investors`. Arguments: `<firm or deal> [pipeline name]`.

A deal with one friendly associate is not a deal with a fund. This skill models the firm the way it
decides: who can say yes, who has to agree, who we actually have a relationship with as opposed to
a name in the database, and the shortest honest path to the people we're missing. Every role and
every "warm" carries its evidence.

Read `${CLAUDE_PLUGIN_ROOT}/references/house-rules.md` first.

## Tools

| Tool | For | Required? |
| --- | --- | --- |
| `whoami` | Project, `workspaceId` | yes |
| `search_companies`, `list_pipelines`, `list_deals`, `get_deal` | The deal: contacts (`isPrimary`, `role`), comments, `lastActivityAt` | yes |
| `get_company` | The firm: people on file, tags, cheque and fund fields, recent signals | yes |
| `search_people` | `companyId` (everyone at the firm); `q` / `linkedinUrl` / `email` for people found elsewhere | yes |
| `search_signals` | `companyId` and per `personId`: who has said what lately | yes |
| `list_project_notes` | `q` = the firm's or a partner's name: what was heard in meetings | yes |
| `list_history` | `{ entityType: "person", verb: "company_changed", workspaceId }`: people who moved to or from this firm | optional |
| Email (Gmail `search_threads`, `get_thread`) | Per person: last exchange, who replied | optional |
| Calendar (Google Calendar `search_events`) | Who has actually attended a meeting | optional |
| Meeting notes (Granola, Fireflies, Otter, Wispr Flow) | Who spoke and what stance they took | optional |
| Web search / fetch | Team page, IC process, who led which deals, portfolio founders | yes |
| `add_deal_contact` | Put a known person on the deal or make them primary, with a `role` | on approval |

## Steps

1. **The deal.** `whoami`; resolve the firm (`search_companies`) and its deal on the project's
   pipelines; `get_deal`. No deal → map the firm anyway and offer `/pipeline add`.
2. **How the firm decides.** From `get_company` (cheque fields, fund) and research (team page, the
   firm's own words about process, interviews), each with a source: the partners; whether there is
   an investment committee and who sits on it; who has led deals at this stage and cheque size
   (2–3 examples with dates); who leads in this project's sector. Unknown stays unknown: "IC process
   not published".
3. **Who we know.** Everyone at the firm in Yellowtail (`search_people { companyId }`), plus the deal
   contacts. Per person: email thread history (last exchange, did they reply), meetings attended
   (calendar, notes), comments that mention them, signals, project notes.
4. **Classify** each person: **role** — decision maker (partner who can lead / IC member),
   sponsor (the partner who would take it to IC), champion, evaluator (associate, principal doing
   the work), gatekeeper (platform, IR), unknown; **engagement** — active (met or replied in 14
   days), warm, cold, never met; **stance** — positive, neutral, negative, unknown, with a quote.
   A **champion** has *done* something for the deal (pushed a partner meeting, shared how IC sees it,
   made an intro); enthusiasm in one call doesn't make one.
5. **Gaps.** No decision maker engaged; the sponsor unknown; single-threaded (one person engaged in
   30 days); the partner who covers the sector never met; a contact who has gone quiet.
6. **Paths in**, for each missing person, best first, each with its evidence:
   - someone at the firm we already know well (the associate who can bring the partner in);
   - a founder the firm backed whom the client or advisor knows (portfolio from research, matched
     with `search_people` and the user's email);
   - someone in Yellowtail who used to work there or moved from there (`list_history`
     `company_changed`, or a person's history on file);
   - a project note or signal that names a connection;
   - cold, anchored on something they said (a signal, a post) — always the last option.
   Never claim a relationship the evidence doesn't show; "the founder of Gridly is backed by Raj"
   is a fact, "the founder of Gridly will intro us" is a question for the user.
7. **Recommend** the one person to contact next, through which path, and what to ask for.
8. **Cards** for people already in Yellowtail who belong on the deal (`add_deal_contact` with a
   `role` such as "Sponsor partner", or `isPrimary: true` when the champion should lead). People not
   in Yellowtail → `/investors contacts`; never filed from here.

## Output

Display: inline — one Yellowtail widget in the chat, cards under it (`${CLAUDE_PLUGIN_ROOT}/references/display.md` §2). Yellowtail's reads draw nothing on the way; this output is the one thing shown (§ Only the answer is drawn).

```
**Northwind Capital** · Seed 2026 (Partner meeting) · map read 9 Oct 2026

How they decide: 4 partners; Monday partner meeting, IC of all partners (northwind.vc/process).
Seed leads $500k–$1.5m by Sarah Lee or Raj Patel (Gridly Jun 2026, Voltgrid Jan 2026).

| Person | Role | Engagement | Stance | Evidence |
| Tom Reid, Associate | Champion | active | positive | set up the partner meeting (email 3 Oct) |
| Sarah Lee, Partner | Sponsor? | warm | neutral | met 8 Oct; asked about margins (notes) |
| Raj Patel, Partner (climate) | Decision maker | never met | unknown | led Gridly seed |

Coverage: 2 of 4 partners met; sponsor not confirmed; single-threaded through Tom until 8 Oct.
Paths to Raj: Tom (works with him) · Gridly's founder Mia Holt, whom Jack emailed in Aug (Gmail)
Next: ask Tom whether Raj will be in the partner meeting, and offer Raj the grid-flex case study.

### 1. Add to the deal
- Sarah Lee → contact, role "Sponsor partner"

Reply with: `approve 1` · `1: make Sarah primary` · `reject 1`
```

Link the deal `/pipelines?pipeline={pipelineId}&deal={dealId}` and people `/investors/people/{personId}`.
Say which sources were read and which weren't.

## What comes next

- `/investors contacts` for the people missing from Yellowtail; `/outreach draft` for the intro ask.
- `/meetings prep` before the meeting with the decision maker.
