# Export a list

The `export` action of `/lists`. Arguments: `[list name] [csv | hunter]`.

A list is the set of people a round will write to. This turns it into something an outreach tool can
send from: a CSV any tool can import, or a Hunter Leads list in the workspace's own Hunter account.
Nobody is emailed from here. The export is a write (it leaves Yellowtail), so it is a card first,
and the people with no email are shown before anything goes out, because an outreach list with
holes is a batch that silently skips the best names.

Read `${CLAUDE_PLUGIN_ROOT}/references/house-rules.md` first.

## Tools

| Tool | For | Required? |
| --- | --- | --- |
| `whoami` | The project, `workspaceId`, whether `enrichment` is in `toolsets.enabled` | yes |
| `list_lists`, `get_list` | Find the list; its entries (person, firm) for the card | yes |
| `get_person` | An entry's email and `emailStatus`, when the list row doesn't carry them | for the card |
| `export_list` | `{ listId, format: "csv" }` returns `{ format, filename, rows, csv, missingEmail[] }`; `{ listId, format: "hunter" }` returns `{ format, leadsListId, leadsListName, url, pushed, missingEmail[], failed[] }` | on approval |
| Google Drive `create_file` | Save the CSV: as a file (`contentMimeType: "text/csv"`, `disableConversionToGoogleType: true`) or as a Google Sheet (`contentMimeType: "text/csv"`, converted) | on approval, if asked |

`export_list` is in the `enrichment` toolset. Missing from your tools → `whoami.toolsets.hint` gives
the URL to reconnect with (`CONNECTORS.md`); say so and stop. The `csv` format needs no Hunter key;
`hunter` does, and without one it is refused ("Hunter is not connected for this workspace…"): offer
`csv` instead.

## Steps

1. **Orient.** `whoami`; the project is `activeProjectId` unless the user named another.
2. **Find the list.** `list_lists { projectId }`, matched to the user's name; several plausible →
   ask. `get_list { id }` for its entries. Format from the argument; not given → ask: `csv` (a file
   for any tool) or `hunter` (a Hunter Leads list in the workspace's Hunter account).
3. **Check the emails.** For each entry, the person's email and `emailStatus` (from `get_list`, else
   `get_person`). Sort into: **ready, verified** · **ready, unverified** · **no email**. Unverified
   addresses are exported (the CSV's `email_status` column says so) but the card counts them, so the
   user can decide to verify first.
4. **No email first.** If any entry has no email, list them by firm and offer `/investors contacts <firm>`
   for each firm before exporting (it runs the contact search, with Hunter when the workspace has a
   key, and files what it finds on its own cards). The user may also say "export anyway": those
   people are left out of a Hunter push and have an empty email in the CSV.
5. **Card** (below), then on approval: `export_list { listId, format }`.
6. **After a CSV export**, offer where it goes (a second card if they choose Drive):
   - **Drive, as a file**: `create_file { title: <filename>, textContent: <csv>, contentMimeType:
     "text/csv", disableConversionToGoogleType: true, parentId? }`.
   - **Drive, as a Google Sheet**: the same without `disableConversionToGoogleType`, so Drive
     converts it.
   - **In the chat**: the CSV in a code block (over ~50 rows, say it is long and offer Drive).
   The folder is the one the user names, else My Drive.
7. **Report.** Rows exported, the Drive or Hunter link (`url` for Hunter), `missingEmail` (names),
   and for Hunter any `failed[]` with the reason. A Hunter push creates or reuses a Leads list named
   after the list; it **never starts a campaign**: say that the sequence is set up and sent from
   Hunter by the user.

## Card

```
**Export list** · Tier 1 — Seed 2026 · Voltframe · format: hunter

### 1. Push 23 people to a Hunter Leads list "Tier 1 — Seed 2026" (Atlas Advisory's Hunter account)
- Ready, verified: 17 · ready, unverified: 6 (exported, marked unverified)
- No email (left out): Raj Patel (Northwind Capital) · Mira Osei (Lantern Ventures)
  → `/investors contacts Northwind Capital`, `/investors contacts Lantern Ventures` first?
- No campaign is started; you send from Hunter.

Reply with any of: `approve 1` · `find contacts first` · `1: csv instead` · `reject 1`
```

## Output

Display: inline — one Yellowtail widget in the chat, cards under it (`${CLAUDE_PLUGIN_ROOT}/references/display.md` §2). Yellowtail's reads draw nothing on the way; this output is the one thing shown (§ Only the answer is drawn).

```
Exported 23 of 25 to Hunter · Leads list "Tier 1 — Seed 2026" · https://hunter.io/leads/…
Left out (no email): Raj Patel, Mira Osei · Failed: none
```

or, for a CSV: the file name and rows, then the Drive link or the CSV itself.

## What comes next

- `/investors contacts <firm>` for the people left out, then export again.
- `/outreach draft` for the opening email the sequence will use.
