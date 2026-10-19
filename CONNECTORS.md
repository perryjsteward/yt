# Connectors

The skills in this plugin name connector **categories** ("email", "the calendar", "meeting notes"),
not products, so any connector in a category works. Yellowtail is the CRM and the one connector that
is required. Every other skill works without the rest: paste the email, upload the transcript, give
the meeting time, and the skill works from that and says what it did not have.

`/setup` checks what this session has and tells you.

## Categories

| Category | Required? | Works with | Used by |
| --- | --- | --- | --- |
| CRM | **Required** | Yellowtail (this plugin's `.mcp.json`) | every skill |
| Hunter | Optional, advised | **Built into Yellowtail**: the workspace's own key, set in the app under Settings → Integrations. No connector to add | `find_email`, `verify_email`, `domain_search` in `/create-investor`, `/find-contacts`, `/investor-prospecting`, `/crm-hygiene`; `export_list` in `/export-list` |
| Email | Optional | Gmail | `/setup` (your writing voice), `/add-user` (a personal note, or the invite when Yellowtail didn't email it), `/draft-outreach`, `/inbox-sweep` (it cannot read Spam), `/handle-objection`, `/call-summary`, `/call-prep`, `/daily-briefing`, `/end-of-day`, `/follow-ups`, `/deal-review`, `/pass-analysis`, `/round-forecast` |
| Calendar | Optional | Google Calendar | `/call-prep`, `/schedule-meeting`, `/daily-briefing`, `/end-of-day`, `/weekly-wrap`, `/deal-review`, `/round-forecast` |
| Scheduling links | Optional | Calendly | `/schedule-meeting`: a single-use link for a named event type (Discovery, Snapshot walkthrough, Kickoff, Close-out, an investor call type) |
| Meeting notes | Optional | Otter, Fireflies, Granola, Wispr Flow, or a paste | `/call-summary`, `/call-prep`, `/end-of-day`, `/weekly-wrap`, `/deal-review`, `/pass-analysis` |
| Docs | Optional | Google Drive | decks for `/create-project` and `/pitch-review`, `/round-strategy`, `/close-plan`, `/investor-update`, `/weekly-wrap`, `/data-room` and its index, `/call-prep` (the brief as a Doc), `/crm-hygiene` (a linked plan doc), `/export-list` (the CSV) |
| E-sign | Optional | DocuSeal (planned), Docusign, SignNow, Jotform Sign | sending a document for signature, where a skill offers it. Detected by its tools, never assumed |
| Other enrichment | Optional | Apollo, Clay (people, emails, phones); Crunchbase (rounds), as their own connectors | the same contact search, after Hunter |
| Web | Optional, strongly advised | Claude's web search and fetch | `/create-project`, `/find-investors`, `/create-investor`, `/find-contacts`, `/investor-brief` |

What a missing connector means:

- **No email**: drafts come back as paste-ready text instead of Gmail drafts.
- **No calendar**: the skill asks for the meeting time and attendees.
- **No meeting notes**: paste or upload the transcript. A summary is never written from a meeting's
  title alone.
- **No Hunter key** (or no `enrichment` toolset): emails at firms come from a search by hand: an
  address the person publishes, or a pattern evidenced by a published address at the same domain,
  all marked unverified, and the skill says so once ("connect Hunter in Settings → Integrations for
  verified emails"). With a key, Hunter is tried first: the skills find the named partner's address
  and check it, and
  file each with exactly the status and source Hunter's answer gives: only Hunter's `valid` at a
  score of 90 or more is `verified` (house rules §5). Every lookup spends the workspace's Hunter
  quota, so `/setup` never tests the key; the first lookup shows whether it works.
- **No Calendly**: `/schedule-meeting` finds times in Google Calendar or proposes them by email.
- **No e-sign**: export the document to PDF and send it through your e-sign tool.
- **No Yellowtail**: the skill stops. Nothing in this plugin works without it.

Yellowtail's `list_history` is the event log (what moved on a deal, who changed what); Google
Calendar's `list_events` is the calendar.

## Connect Yellowtail

Yellowtail is an MCP server you sign in to with your Yellowtail account. There is no key to copy.

The URL is your Yellowtail app's address plus the server path:

```
https://www.yellowtail.club/api/mcp/admin
```

Customers connect through `/api/mcp/admin` for now and see only the tools their account may use.
With no `?toolsets=`, the server lists every tool group the plugin needs: `catalog`, `tenancy`,
`pipelines` and `enrichment` (Hunter). A connector added earlier with
`?toolsets=catalog,tenancy,pipelines` still works but has no Hunter tools and no `/export-list`:
`/setup` reads `whoami.toolsets` and shows the URL to reconnect with (`toolsets.hint`).

### Claude Code

1. Install the plugin (see `README.md`). Its `.mcp.json` already points at Yellowtail, so there is
   nothing to configure.
2. Run `/mcp`, pick **yellowtail**, and choose **Authenticate**. A browser opens on Yellowtail; sign
   in, select **Allow**, and you are back in Claude Code, connected.

### claude.ai and Claude Desktop

**Settings → Connectors → Add custom connector.** Name it "Yellowtail", paste the URL above, leave
the advanced settings empty, and select **Add**, then **Connect**. Sign in to Yellowtail and select
**Allow**. A connector added on claude.ai is in Claude Desktop too.

### Check it

Run `/setup`. It calls `whoami` and shows who you are signed in as, your workspaces and projects,
and whether writes are allowed. A sign-in connection acts as you, with exactly what you can do in the
app; disconnect it any time under **Account → Connected apps** in Yellowtail.

## Gmail, Google Calendar, Google Drive and Calendly

These are claude.ai connectors, not part of this plugin's `.mcp.json`. Connect them once in
**Settings → Connectors** in claude.ai or Claude Desktop (they are then available in Claude Code too
when you are signed in to the same account). Sign each in with your **work** account: `/setup`
shows which account each one uses and warns when it isn't on your Yellowtail email's domain.
Meeting-notes tools (Granola, Fireflies, Otter, Wispr Flow), e-sign tools and Apollo or Clay are
connected the same way, where they offer a connector; otherwise add the tool's own MCP server as a
custom connector. Hunter is not one of these: it is set up inside Yellowtail.

### Writing to Google Drive

The Drive connector can create files but not edit a Google Doc's body: `update_file` changes only a
file's title or folder. So skills that produce a document (a prep brief, a data room's `00 Index`)
**create** it from HTML (`create_file` with `contentMimeType: "text/html"`, which Drive converts to
a Google Doc), and a new version is a new Doc. Text that must stay current, such as the data room
index, lives in a Yellowtail project note, which can be updated in place. A CSV is saved as a file
(`contentMimeType: "text/csv"` with `disableConversionToGoogleType: true`) or converted to a Sheet.

## Permissions

What a connector may do is set on the connector: allow, ask or block for each tool. The skills never
go beyond it, and they never send an email or create a calendar event without you saying so.
