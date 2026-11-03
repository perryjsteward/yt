# Changelog

## 0.9.0 (3 Nov 2026)

- **`/fulfil-orders`** (platform admins): work the order queue. When a customer buys their investor list or the annual licence, Yellowtail makes an order due in 48 hours, carrying a brief of the project and its interests frozen at purchase. This picks up the next one, builds a List in the customer's project of the 200 investors that fit the brief best (filing new firms from research where the catalog is thin, as `/find-investors` does), gives it a pass over, and delivers the order with that List, which emails the customer a link to it. Runs unattended on a schedule (`--deliver`), leaves an order it can't fill in progress with a note, and skips upgrades (the licence on top of a project's list), which have nothing to build. Uses the new order tools.

## 0.8.0 (3 Nov 2026)

- **`/invite-founder`** (platform admins): invite a founder you met to make their own free Yellowtail account. It saves the invite as a draft, reads their website for sector, stage and geography, checks how many investors in Yellowtail fit, researches and files more when coverage is thin (as `/find-investors` does), then sends the invite on your yes. Their ten free investors are picked once, at onboarding, so the research comes first. Uses the new join-invite tools.

## 0.7.6 (19 Oct 2026)

- Lives in its own private `perryjsteward/yt` repository, with the `yellowtail-club` marketplace at the root. Plugin source, validation and releases are independent of the app repository. Command names and existing practice files are preserved.

## 0.7.5 (19 Oct 2026)

- Managed-service command links now use `/yt-raise:`. The core plugin stays `yt`, displayed as Yellowtail Club.

## 0.7.4 (19 Oct 2026)

- **Yellowtail Club** is the core skills plugin for the Yellowtail MCP; its install ID stays `yt@yellowtail`. Descriptions now name the core toolkit, and managed-service command links use `/raise:`.

## 0.7.3 (17 Oct 2026)

- **Only the answer is drawn, for real.** 0.7.2 asked Claude to pass `display: false` on its
  lookups, and it didn't, so every lookup still drew a card. Now Yellowtail's reads never draw at
  all: a skill's working-out is just the collapsed tool line, and the user sees its output, or one
  `show_results` card when a single read is the answer (`/help`'s one-step questions).
  `display.md` § Only the answer is drawn, house rules §13.

## 0.7.2 (17 Oct 2026)

- **Only the answer is drawn.** Every skill passes `display: false` on the Yellowtail reads it makes
  on the way, so the chat shows its result, not a card per lookup; `/help` keeps the card when one
  read is the answer. `display.md` § Only the answer is drawn and house rules §1 and §13.
- **Every Yellowtail read draws its own card.** Projects, workspaces, pipelines, lists, the round's
  timeline, history, notes, tags and the desk's records now come back as a Yellowtail table or card,
  not only firms, people, deals and signals. `display.md` §2 says so: never repeat a drawn card's
  rows as a table.

## 0.7.1 (17 Oct 2026)

- **`/help` answers one-step questions itself.** "What stage is Lantern on?" is answered, and "move
  Lantern to Term sheet" is shown as a card and made on approval, rather than sending you to a
  command; commands stay the answer for jobs with several steps. `/help` on its own now ends with
  **Or just ask**: things you can say to Yellowtail without a command, for the tool groups your
  connection has (from `about_yellowtail`, so it never lists one you can't use).
- **`/find-investors` adds the new firms it finds to Yellowtail.** Each investor from the web that
  Yellowtail didn't have and the shortlist keeps is filed to the shared catalog during the run, so
  the next search (yours or anyone's) finds it: the firm only, with existing tags, no people, and
  marked unreviewed for the Yellowtail team to check (`/crm-hygiene --catalog` lists them). Strategic
  buyers, ruled-out firms, conflicts and possible duplicates are not filed; they stay `create`.
  `--no-file` turns it off for a run.
- `/find-contacts`' description is back under claude.ai's 1024-character limit, past which the
  whole plugin failed to load.

## 0.7.0 (17 Oct 2026)

- **Results look like Yellowtail, and most stay in the chat.** Finding, searching, adding and
  editing now answer with one branded widget in the conversation (shortlists as tables, firms and
  people as cards with their contact statuses, a stage strip, the timeline), drawn with Claude's
  Visualizer; buttons on a card post the same `approve 1 3` reply you would type, and typing still
  works. Numbers you come back to are a **live dashboard** made once per project and reused:
  `/pipeline-review`, `/weekly-wrap`, `/round-forecast`, `/portfolio-review`, `/pass-analysis` and
  `/close-plan` read Yellowtail through the connector each time the page opens, held to each
  reader's own access. Briefs and reviews (`/call-prep`, `/investor-brief`, `/investor-qa-prep`,
  `/round-strategy`, `/pitch-review`, `/term-sheet-review`, `/investor-update`) are a branded
  document. One reference for all of it: `references/display.md`; house rules §13.
- **Yellowtail draws its own cards.** In claude.ai (and any client with MCP Apps) searching firms,
  listing investors or prospects, opening a firm, listing people, deals or signals comes back as a
  Yellowtail card from the server itself, the same whether you asked in your own words or ran a
  command. Shortlists (`/find-investors`, `/investor-prospecting`, `/club:prospect`,
  `/club:tier-prospects`) go through the new `show_shortlist` card: grouped by fit, numbered,
  with evidence and sources, ticks, and buttons that send `create 1 3` for you.
- **`/help [question]`.** Ask in your own words ("how do I get investors onto my round", "I've got a
  call at 3", "my project shows no investors") and get the one command that fits, written out with
  your firm, list or number in it, why it fits, what it needs first (it checks your project,
  interests, rounds and lists where that matters), and what comes after. It reads every skill in the
  session (yt-raise's too), the plugin's guides and the help centre, has a troubleshooting table for
  the usual connection and setup problems, and links the help-centre article for questions about the
  app. With no question, a menu and the recommended next step. Read-only; it runs the command only
  when you say so.
- **`/find-contacts` on a firm Yellowtail doesn't have looks at it before giving up.** It reads the
  firm's site first: one that invests is offered as "add it to Yellowtail, then find contacts?",
  and a `yes` puts the new firm and its people on one card. One that doesn't (an agency, a startup)
  gets a plain "it doesn't look like an investor — did you mean `/find-investors`?", with the user
  able to say it does invest after all. It never files a non-investor to make the command work.

## 0.6.4 (16 Oct 2026)

- **Firms are filed fully tagged, and new tags are rare.** Recent filings went in under-tagged: a
  biotech fund tagged only Food & agriculture, life-sciences funds with no Health tag, semiconductor
  funds tagged Industry, university funds with no geography. Tagging is now its own check
  (`research-an-investor.md` § Firm — tags): the whole tree is read, each sector maps to the most
  specific existing tag plus its parent (synonyms map, they don't create), the main sector comes
  first, and every applicable kind is tagged or listed on the card as **Tags not stated** with where
  it was searched. `file_investor`'s dry run now returns `tagGaps` to check against, and refuses a
  new firm with no tags.
- **A new tag has to pass a gate** (house rules §12): a market term other funds use, not the firm's
  own phrase; two other firms it would apply to; led with by the thesis; one level under an
  existing group, named like its siblings; at most one per card. A term that fails gets the nearest
  existing tag and goes in the description. `/find-investors` lists in-Yellowtail firms whose tags
  miss the evidence under **Tags to fix**, for `/create-investor` to retag.

## 0.6.3 (16 Oct 2026)

- **`/add-to-list` creates the list when it does not exist.** Adding people to a list the project
  doesn't have (or to a project with no lists) used to stop at "run `/create-list` first", and could
  end in an `add_to_list` call with no real list id. Now the card proposes the new list and its
  people together, and one approval runs `create_list` then the adds. A near-match name still asks
  first. `/create-list` hands a "make a list with these people" request to `/add-to-list`.

## 0.6.2 (16 Oct 2026)

- **`/create-project` asks first and finishes.** A new Step 0 asks, in one question call before any
  research, only what would block creation: which workspace (the active project's recommended, each
  with its project count against its limit), what to do about a full one (raise the limit, free a
  slot, pick another), and whether to refresh a same-name project; plus, while asking anyway, the
  round being raised. An explicit "create" request is then the approval: the project is made and the
  card is the report, with an undo line. It stops only for a new tag or a strong duplicate, and a
  question it waits on now opens the reply instead of trailing the card (house rules §2).

## 0.6.1 (16 Oct 2026)

- **The plugin loads again.** The descriptions of `/call-prep`, `/create-investor`, `/crm-hygiene`, `/data-room`, `/find-contacts`, `/find-investors` and `/handle-objection` were over the 1024-character
  limit, which stopped the whole plugin loading. They are shortened; the skills themselves are unchanged.

## 0.6.0 (15 Oct 2026)

From Jack's end-to-end test of YT Raise (`docs/plans/2026-10-15-yt-raise-test-feedback.md`).

**Moved**

- **`/client-update` moved to yt-raise** as `/yt-raise:client-update`. It is the advisor's weekly
  report to the client founder; a founder running their own raise uses `/weekly-wrap` or
  `/investor-update`. Every mention here now points there or to those.

**Added**

- **The round's timeline.** A round's dates are milestones (`list_milestones`, `create_milestone`,
  `update_milestone`): launch, lead term sheet, lead committed and legal close are separate, so
  "close by" no longer means two things. `/create-pipeline` puts the dates the user gives on its
  card; `/round-strategy` dates the close backwards as a feasibility check up front (legal lead
  times by jurisdiction and instrument: a Dutch B.V. priced round needs a notarial deed, a German
  GmbH round is notarised) and ends with a card of milestone creates and updates; an open decision
  about a date is the milestone's note and a project note.
- **Timeline at risk** in `/daily-briefing`, `/end-of-day`, `/weekly-wrap` and
  `/portfolio-review`: overdue and due-soon milestones, an overdue one always action 1.
  `/end-of-day` proposes a dated note on a milestone the day moved.
- **Run it on a schedule** notes in `/end-of-day` and `/weekly-wrap`, as `/daily-briefing` has: a
  scheduled task or routine in Claude. Yellowtail itself runs nothing on its own.
- **`/crm-hygiene` plan vs record**: overdue milestones not done, deals expected to close after
  legal close, a raising round with no timeline, a linked plan doc whose dates disagree.
- **Pre-launch mode** in `/pipeline-review` and `/round-forecast`: before anything is contacted,
  what is ready, what is missing and the next dates, with no empty forecast.
- **`/call-summary` coaching**: a What landed / What to improve section, each line citing the
  transcript.
- **`/term-sheet-review`**: a Suggested action column (accept, ask for, push back on, ask your
  lawyer about). Still not legal advice.
- **`/call-prep` batch mode** ("prep my calls today"): every section kept, shorter.
- **`/create-investor` tag backfill** for platform admins: after a new tag, a card-first pass over
  likely firms, checked against each firm's description and portfolio, in batches of about 15.

**Changed**

- **Round currency.** Rounds carry `target` and `currency`; `targetUsd` is deprecated and no skill
  keeps a target, currency or close date in `attributes`. Currencies are never converted.
  `/add-to-pipeline` and `/update-deal` leave a deal in the round's currency and pass `currency`
  only for a figure given in another one.
- **`/find-investors`** scores cheque fit and lead appetite without a pipeline (the round's
  target, else the project description, else one question), sets aside firms already on any of
  the project's pipelines **or lists** (so `/find-investors 5` tops up a running round), and when
  Yellowtail runs short leans on the web and pushes `create` so new firms are saved to Yellowtail.
  Tag filters now include a tag's children ("Europe" finds a European country).
- **Contact emails: Hunter first.** `/find-contacts`, `/create-investor`, `/crm-hygiene` and
  `references/research-an-investor.md` try Hunter through Yellowtail first (`find_email` for the
  named partner, then `verify_email`), then an address the person publishes, then an evidenced
  pattern, `unverified`. Without a key they say once that they searched by hand and how to connect
  Hunter. The right partner for this round, never a role inbox: `info@`, `hello@`, `team@` and the
  like are never a person's email (house rules §5).
- **`/create-investor`** says filing needs no round or pipeline and offers `/add-to-pipeline`
  after. A new specific tag goes on **alongside** its broad parent, never instead. A tag kind the
  user can't create is listed under **Tags to ask the Yellowtail team for** (house rules §12),
  replacing "Not created".
- **`/close-plan`** plans against `legal_close` and `lead_committed` from `list_milestones` and
  proposes moving a milestone, with the reason as its note, when its plan says a date is
  unrealistic. `--close-by` is now `--legal-close`.
- **`/call-prep`** always has Recent deals and signals ("None in Yellowtail;" and what the web
  found) and likely questions with suggested answers (`references/meeting-prep-brief.md`).
- **`/inbox-sweep`** says the Gmail connector cannot read Spam, and ends by asking the user to check
  Spam for investor replies and mark them Not spam.
- **`/handle-objection`** with no objection given finds the latest reply or pass first (email, the
  deal's comments, call notes) and quotes it.
- **`/draft-outreach`** warm mode: one ask to a connector can cover several investors, picked from
  Yellowtail (the round, the connector's co-investors), drafted with what is known and `[to
  confirm]` only for what isn't. Drafting from the founder's mailbox reads their "Founder voice"
  project note, and the first batch goes to the founder for approval.
- **`/add-user`**: Yellowtail now emails the invite when email delivery is on (`emailedAt`); the
  link and a Gmail draft are offered only when it wasn't emailed or for a personal note.
- **`/setup`** mentions the round's timeline in What you can do now.

## 0.5.0 (10 Oct 2026)

- **Connection.** `.mcp.json` uses the bare URL `https://www.yellowtail.club/api/mcp/admin`, which
  lists every tool group the plugin needs, Hunter's `enrichment` included. `/setup` reads
  `whoami.toolsets` and shows the URL to reconnect with when a group is missing.
- **`/setup`** prints which account Gmail, Google Drive and Calendly are signed into and warns when
  it isn't the work domain; recognises Hunter (inside Yellowtail, never tested at the cost of
  quota), Calendly and e-sign connectors.
- **Investors file to the shared catalog.** `/create-investor`, `/find-contacts`,
  `/investor-prospecting` and `/crm-hygiene` file investor cards to Yellowtail's global catalog for
  the project's workspace, enrich firms already there (fill, change, retag, people), skip firms in
  another workspace's private catalog, and show name-only look-alikes without blocking the card.
- **New tags are created on approve.** Cards carry **New tags (created on approve)**: industry (with
  its group) and focus tags are created by anyone with `create_tag` before filing; an existing
  look-alike is reused and said; other kinds are reported as the Yellowtail team's. Replaces
  "Tags to add".
- **Contact sources.** Every contact value is filed with a `sources` line. Work emails on personal
  mailboxes are refused.
- **Hunter.** `/find-contacts`, `/create-investor`, `/investor-prospecting` and `/crm-hygiene` use
  `domain_search`, `find_email` and `verify_email` when the workspace has a key, filing exactly the
  status and source Hunter returns. Without a key they work as before.
- **New `/export-list`**: a list as a CSV (to Drive as a file or Sheet, or in the chat) or pushed
  to a Hunter Leads list, card first, people with no email offered to `/find-contacts` first.
- **The Meeting Prep Brief.** `/call-prep` writes it (who, the fund, activity check with "may be
  inactive" after 18 months, recent deals and signals, portfolio conflicts, tensions, likely
  questions with answers from the data room, the angle, the goal, what to ask), can save it as a
  Google Doc and draft an email to the founder. `/investor-brief` uses the same layout before a
  meeting exists. Shared in `references/meeting-prep-brief.md`.
- **The data room index.** `/data-room` saves a project note starting `# Data room index`
  (documents with links, dates and summaries; key metrics with their source; known gaps);
  `--refresh` updates it in place. `/investor-qa-prep`, `/call-prep`, `/inbox-sweep`,
  `/handle-objection` and `/follow-ups` read it first and answer only from it, citing the document
  (`references/data-room-index.md`).
- **`/client-update` is the Friday report**: the plan line (phase, week n of m, plan end, extension
  decision date), the week's numbers, a call analysis, highlights, next week, asks of the founder,
  and the renewal heads-up within four weeks of plan end.
- **`/find-investors`** ranks on evidence (description, notable investments, dated deals and
  signals), flags "may be inactive" and portfolio conflicts, treats name-only matches below 0.8 as
  new, and has `--snapshot` (exactly 20, existing backers excluded). `references/fit-scoring.md`
  matches.
- **`/schedule-meeting`** offers a single-use Calendly link for a named event type when Calendly is
  connected.
- **`/crm-hygiene --catalog`** (platform admins): catalog-wide duplicate firms and people, with
  `merge_companies` cards, dry run first.

## 0.4.0 (9 Oct 2026)

- First published set of Yellowtail skills: project and round setup, investor research and filing,
  the pipeline loop with email, calendar and meeting notes, and the round's reviews.
