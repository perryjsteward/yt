---
name: add-to-pipeline
description: Yellowtail Club — Put investor firms on a Yellowtail pipeline as deals, one deal per firm, in the first stage or a named one, with the primary contact chosen from Yellowtail's people at that firm when the user names none, and the choice explained. Skips firms already on the pipeline. Works on one firm, a batch (a shortlist from /find-investors, a pasted list, a CSV), or a whole Yellowtail list copied in one go. Proposes cards and creates deals only on approval. Use when the user says "add X to the round", "put these on the pipeline", "start deals for these funds", "move my Tier 1 list onto Seed 2026", "import the list into the pipeline", or picks firms from a shortlist to pursue.
argument-hint: "[pipeline name] <firms… | list name> [stage]"
---

# Add to a pipeline

A deal is one opportunity with one firm on one pipeline: its own title, stage, value and currency,
owner, expected close, notes, and contacts (people, one primary). This skill opens deals. Moving
them afterwards is `/update-deal`. It works on any pipeline: a fundraising round, or an advisor's own
client pipeline where the "firm" is a prospective client.

Read `${CLAUDE_PLUGIN_ROOT}/references/house-rules.md` first.

## Tools

| Tool | For | Required? |
| --- | --- | --- |
| `whoami` | User id (default owner), active project | yes |
| `list_pipelines`, `get_pipeline` | Find the pipeline; its stages in order (ids for `stageId`) | yes |
| `list_deals` | Firms already on it: `{ pipelineId, query: { companyId, status? } }` | yes |
| `search_companies`, `get_company` | Resolve each firm; its people and recent signals | yes |
| `search_people` | Everyone at a firm (`companyId`), or a named contact | yes |
| `get_project`, `list_tags` | The project's interests, for choosing the primary contact by sector | yes |
| `list_workspace_members`, `list_project_members` | An owner the user names who is not them (`userId`) | when named |
| `create_deal` | `{ pipelineId, deal: { companyId, stageId?, title?, value?, currency?, ownerId?, expectedCloseOn?, nextStep?, nextStepOn?, notes?, contactIds? } }` | yes, on approval |
| `add_deal_contact` | Give a contact a `role` ("intro via Sarah"): `{ dealId, contact: { personId, isPrimary?, role? } }` | only when a role was given |
| `list_lists`, `get_list`, `import_list_to_pipeline` | The whole-list path | for a list |

## Steps

1. **Orient.** `whoami`. Project from `activeProjectId` unless named.
2. **Pick the pipeline.** `list_pipelines { projectId }`. The one the user named; else the only one
   that is not `closed` (statuses: `planning`, `raising`, `closed`); else ask. `get_pipeline { id }` for the stages. The default stage is the
   **first** (lowest `position`); a stage the user names must match one of these by name, or ask.
3. **A whole list?** If the user named a list ("move Tier 1 onto Seed 2026"), go to
   **Importing a list** below.
4. **Resolve each firm.** `search_companies` by name, domain, LinkedIn. Ambiguous → a question.
   Not in Yellowtail → **Not added**, offering `/create-investor`. An angel with no firm is a
   person (`search_people`), and their deal has no `companyId` (step 7).
5. **Already on the pipeline?** `list_deals { pipelineId, query: { companyId } }` returns **open
   deals only** unless you ask, so check `status: "won"` and `status: "lost"` too. An open deal →
   **Already on the pipeline**, with its stage. A won or lost one → a question ("Northwind was
   lost on 3 Aug. Open a new deal?"). Opening one anyway is the user's call.
6. **Choose contacts.** If the user named the person, use them. Otherwise:
   - `search_people { companyId }` for everyone at the firm, and the project's interests
     (`get_project`, then `list_tags` per kind for the names).
   - **Primary:** the most senior person whose sector matches the project's (a title, a profile
     note, a signal naming the sector; `get_company` shows recent signals and who they name).
     A Partner who leads the sector beats a Managing Partner who doesn't. Say why on the card.
   - Add a second contact only when there is a clear reason (the Principal who runs that Partner's
     deal flow). `contactIds` order matters: **the first id is the primary**.
   - No evidence to choose between people → a question listing them. No people at the firm →
     create the deal with no contacts only if the user wants it now; otherwise offer
     `/find-contacts <firm>` first. Say which on the card.
7. **Fill the deal.**
   - `title`: omit it, and it becomes the firm's name. That is the convention. Set a title only for
     a second deal with the same firm ("Northwind Capital: Fund II SPV"), or a deal with no firm
     (the angel's own name, "Jane Doe (angel)").
   - `stageId`: omit for the first stage, else the named stage's id.
   - `value` (whole units, integer, no decimals): only if the user gave a figure. A firm's cheque
     range is not a deal value. **Currency:** a new deal takes the round's `currency`
     (`get_pipeline`) when none is sent, so leave `currency` out. Pass it only when the user gives
     the figure in another currency (`USD` `EUR` `GBP` `CHF` `CAD` `AUD` `SGD` `HKD` `JPY` `INR`
     `SEK` `NOK` `DKK` `ILS` `AED` `BRL`), and never convert. The card shows the value in the
     currency it will carry.
   - `ownerId`: the user's own id from `whoami` by default, shown on the card as "Owner: you";
     a named colleague's `userId` from the member lists.
   - `expectedCloseOn` (`YYYY-MM-DD`) and `notes` (markdown): only from what the user said, e.g.
     "Intro via Sarah Lee (Northwind), 6 Oct".
   - `nextStep` and `nextStepOn` (`YYYY-MM-DD`): the first action, when the user gave one ("ask
     Sarah for the intro by Friday"). Don't invent one; a deal with no next step is what
     `/pipeline-review` flags, which is the point.
8. **Propose** at most 15 firms per round; say how many remain.
9. **Create** on approval, card order: `create_deal`. Then `add_deal_contact` only for contacts given
   a `role`. If one fails, say why and continue; never re-create one that succeeded.

## Importing a list

`import_list_to_pipeline { pipelineId, import: { listId, stageId?, skipExisting: true } }` copies a
list in one call: one deal per firm the list's people are at, those people as its contacts (the
first is primary), and a deal named after the person for anyone with no firm. Firms already on the
pipeline are skipped while `skipExisting` is true. It is a copy: later list changes don't follow. Imported deals take the round's currency.

There is no dry run, so preview it yourself: `get_list`, group the entries by `companyName`, check
each firm with `list_deals` as in step 5, and show **one card** with the counts and the firms. Set
`skipExisting: false` only if the user explicitly wants second deals with firms already there. Report
the call's own `created`, `skipped` and `contactsAdded`.

## Card

```
**Add to "Seed 2026"** in Acme Robotics: 3 deals in Researching, 1 already on it

### 1. Northwind Capital
- Primary: **Sarah Lee**, Partner: leads their climate deals (signal, 6 Oct)
- Also: Tom Reid, Principal: runs Sarah's pipeline (profile note)
- Owner: you · Value: none · Close: none

### 2. Brightwater Ventures → stage Intro booked
- Primary: **Anna Holm**, Founding Partner: the firm's only partner
- Notes: "Intro via Sarah Lee, 6 Oct"
- Next step: **Ask Sarah for the intro (us)** · due **2026-10-10**

### 3. Halcyon Ventures
- No people in Yellowtail. Opens with no contact; `/find-contacts Halcyon Ventures` after?

**Already on the pipeline** · Lantern Capital (Pitched)
**Needs your input** · **A.** Greyhound Partners was lost on 3 Aug. Open a new deal?

Reply with any of: `approve all` · `approve 1 2` · `1: primary Tom` · `3: skip` · `A: yes`
```

## Output

Display: inline — one Yellowtail widget in the chat, cards under it (`${CLAUDE_PLUGIN_ROOT}/references/display.md` §2). Yellowtail's reads draw nothing on the way; this output is the one thing shown (§ Only the answer is drawn).

One line per deal created, with its stage and primary contact, linked
`/pipelines?pipeline={pipelineId}&deal={dealId}`. Then failures with reasons, then firms waiting on
contacts or research.

## What comes next

- `/find-contacts <firm>` for deals opened with no contact.
- `/draft-outreach` or `/schedule-meeting` to start working the new deals.
- `/update-deal` as they move.
