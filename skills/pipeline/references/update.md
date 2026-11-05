# Update a deal

The `update` action of `/pipeline`. Arguments: `<firm or deal> <what changed> [why]`.

One deal, several fields, one card. Every change goes with a **comment on the deal** saying what
moved and why, because the comment thread is the deal's activity log and a stage that changed with
no reason is a question someone asks later. (The stage history itself is recorded by Yellowtail in
the event log, `list_history`; the comment carries the *why*.)

Read `${CLAUDE_PLUGIN_ROOT}/references/house-rules.md` first.

## Tools

| Tool | For | Required? |
| --- | --- | --- |
| `whoami` | User, active project | yes |
| `list_pipelines`, `get_pipeline` | The project's pipelines; stage ids and names | yes |
| `list_deals` | Find the deal: `{ pipelineId, query: { companyId?, q?, status? } }` | yes |
| `search_companies`, `search_people` | Resolve a firm or a contact the user names | yes |
| `get_deal` | The deal now: fields (including `nextStep`, `nextStepOn`), contacts, comments | yes |
| `list_workspace_members`, `list_project_members` | A new owner's `userId` | for an owner change |
| `update_deal` | `{ id, changes: { stageId?, value?, currency?, status?, ownerId?, expectedCloseOn?, nextStep?, nextStepOn?, notes?, title?, companyId? } }` | yes, on approval |
| `add_deal_contact` | Add a contact or make one primary: `{ dealId, contact: { personId, isPrimary?, role? } }` | for a contact change |
| `comment_on_deal` | The log line: `{ dealId, comment: { body } }`, markdown | yes, on approval |

## Steps

1. **Orient.** `whoami`; project from `activeProjectId` unless named.
2. **Find the deal.** Resolve the firm with `search_companies`, then `list_deals { pipelineId,
   query: { companyId } }` on each of the project's pipelines (`list_pipelines`). `list_deals` returns
   open deals only by default; ask for `status: "won"` / `"lost"` when the user is reopening one.
   One match → use it. Several (the same firm on two rounds) → ask which. A deal with no firm is
   found by `q` on its title.
3. **Read it.** `get_deal { id }`: the current stage, value, currency, owner, close date, next step
   and its date, notes, contacts and recent comments. Every line on the card shows **old → new**.
4. **Map the request to fields.**
   - **Stage:** a stage name from `get_pipeline`, matched exactly or near-exactly. If the user's word
     is not a stage ("diligence" when the stages say "DD"), say which you matched. No match → ask;
     never create a stage.
   - **Status:** `open`, `won` or `lost`, nothing else. "Passed", "declined", "dead", "no" → `lost`.
     "Committed", "signed", "in" → usually `won`, but on a round a soft commit is not a close:
     ask if it is unclear. Reopening a lost deal is `status: "open"`.
   - **Value:** integer, whole units ("€500k" → `500000`). **Currency:** a deal carries the
     round's currency by default, so send `value` alone when the figure is in it. Pass `currency`
     only when the user gives the figure in another one (`USD` `EUR` `GBP` `CHF` `CAD` `AUD` `SGD`
     `HKD` `JPY` `INR` `SEK` `NOK` `DKK` `ILS` `AED` `BRL`), and say on the card that it differs
     from the round's. Changing currency relabels the amount; it converts nothing, so say so if
     only the currency changes.
     `null` clears the value ("we don't know the ticket yet").
   - **Owner:** a member's `userId` from the member lists, or the user's own id from `whoami`.
   - **Expected close:** `YYYY-MM-DD`. "End of Q4" → ask for a date or propose the last day of the
     quarter and say so.
   - **Notes:** markdown, and `notes` **replaces** the whole field. Show the full new text, normally
     the old notes with the addition. Never drop what was there unless the user asked.
   - **Primary contact:** `add_deal_contact` with `isPrimary: true` (the database demotes the old
     one). A person not yet on Yellowtail: `/investors contacts` first. Taking someone off a deal can't
     be done from this connection; say so and point to the app.
   - **Next step:** `nextStep` (short text: what, and who does it) and `nextStepOn` (`YYYY-MM-DD`,
     the date it is due). Both replace what was there; `null` clears them when the step is done and
     nothing follows. A deal written before these fields existed may carry its next step only as a
     `**Next:**` line in a comment: show that as the old value. Moving to `won` or `lost` → propose
     clearing the next step unless the user gave one (a signing, a polite close-out).
5. **Write the comment.** One short markdown entry, in the user's words where possible:
   `**Stage:** Pitched → Diligence. Asked for the data room after the partner meeting (8 Oct).`
   Then any other changes, then **Next:** if there is one (the same words as `nextStep`, so the log
   says what was agreed even after the field moves on). The *why* comes from the user or from a
   source you cite; if there is none, the comment states the change only. Never invent a reason.
6. **Propose** the card. Several deals in one request get one card each.
7. **Write** on approval: `update_deal` (only the fields that change), then `add_deal_contact` if
   needed, then `comment_on_deal`. If `update_deal` fails, don't post the comment; say why.

## Card

```
**Update a deal** in Acme Robotics · Seed 2026

### 1. Northwind Capital
- Stage: Pitched → **Diligence**
- Value: none → **€500,000** (EUR)
- Close: none → **2026-12-15**
- Next step: none → **Send data room access (us)** · due **2026-10-10**
- Primary contact: Sarah Lee → **Tom Reid**
- Comment:
  > **Stage:** Pitched → Diligence. Asked for the data room after Tuesday's partner meeting.
  > **Value:** €500k ticket discussed. **Next:** send data room access by Friday.

Reply with any of: `approve all` · `1: no value yet` · `1: close is Jan 31` · `1: next step due Mon` · `reject 1`
```

## Output

Display: inline — one Yellowtail widget in the chat, cards under it (`${CLAUDE_PLUGIN_ROOT}/references/display.md` §2). Yellowtail's reads draw nothing on the way; this output is the one thing shown (§ Only the answer is drawn).

One line per deal: what changed, linked `/pipelines?pipeline={pipelineId}&deal={dealId}`, and
"comment logged". Then any failure with the tool's own message.

## What comes next

- `/meetings schedule` when the next step is a meeting; `/outreach draft` when it is an email.
- `/pipeline log` to record the conversation behind the change in more detail.
- `/pipeline review` to see the whole round after a run of updates.
