---
name: help
description: Yellowtail Club — Say what to do next in Yellowtail. Takes a question in the user's own words ("how do I get investors onto my round", "my list is empty", "list or pipeline?") and answers with the one command that fits best, filled in for them, why it fits and what it needs first, then the next step or a short sequence; a one-step question it just answers, and a one-step change it shows as a card and makes on approval. Reads every skill available in the session (this plugin's and yt-raise's), the plugin's guides and the Yellowtail help centre, and checks the user's own setup (signed in, active project, interests, rounds, lists) where that changes the answer. With no question it gives the recommended next step, the commands and what can be asked without one. Runs no command until the user says so. Use when someone says "help", "what can you do", "how do I…", "which command…", "where do I start", "what next", "I'm stuck", or asks what a command does.
argument-hint: "[your question, in your own words]"
---

# Help

The front desk for this plugin. Someone describes what they want, or what went wrong, and gets
the best command for it, written out ready to run, with the reason, what it needs and what comes
after. It runs a command only when the user then says "go" (or names it).

A question that is one step needs no command, and help does not send someone to learn one:
"what stage is Lantern on?" is answered, and "move Lantern to Term sheet" is shown as a card and
made on approval. Commands are for jobs: several steps, judgement, or sources outside Yellowtail.

Read `${CLAUDE_PLUGIN_ROOT}/references/house-rules.md` first.

## Where the answers come from

Never answer from memory of an older version. In this order:

1. **The skills in this session.** Every Yellowtail skill Claude can see is listed with its
   description, whichever plugin it came from (`yt`, `yt-raise`). That list is the truth about
   what exists: a command not in it does not exist here, whatever this file says. Where the
   files can be read (Claude Code), open the candidates'
   `${CLAUDE_PLUGIN_ROOT}/skills/[area]/SKILL.md` and the action's
   `${CLAUDE_PLUGIN_ROOT}/skills/[area]/references/[action].md` before recommending one, for its
   inputs, prerequisites and what it writes. Commands are areas with actions (`/investors find`,
   `/pipeline add`): recommend the area and the action, written out.
2. **This plugin's guides**: `${CLAUDE_PLUGIN_ROOT}/README.md` (the commands, grouped, and the
   round's timeline), `${CLAUDE_PLUGIN_ROOT}/CONNECTORS.md` (connecting Yellowtail and the
   optional connectors), `${CLAUDE_PLUGIN_ROOT}/references/house-rules.md` (cards, approval,
   sources), `${CLAUDE_PLUGIN_ROOT}/references/fit-scoring.md` (how investors are ranked) and
   `${CLAUDE_PLUGIN_ROOT}/references/data-room-index.md` (what the data room index is for).
3. **The product.** `about_yellowtail` for what Yellowtail is, its concepts (workspace, project,
   signal, list, pipeline, deal) and what works today versus what is coming soon; its
   `links.helpCentre` is the help centre's address. A question about doing something **in the
   app** (a button, a page, a setting) is answered from the help centre article, linked as
   `[helpCentre]/[slug]` (table below), alongside the command that does it from Claude. Its
   `whatYouCanAsk` is the example requests for the tool groups this connection has, already
   filtered (a read-only key gets no writes); the menu's "or just ask" block comes from it, never
   from a list kept here.
4. **The user's own setup**, read only when it changes the answer (step 2).

## Tools

| Tool | For | Required? |
| --- | --- | --- |
| `whoami` | Signed in at all; platform admin; `activeProjectId`; `key.canWrite`; `toolsets.enabled` and `toolsets.hint` | no: without it the answer is "connect Yellowtail first" (`CONNECTORS.md`) |
| `about_yellowtail` | Product questions; the help centre's address; `whatYouCanAsk` for the menu | no |
| `get_project` | The active project's `interestTagIds`: none means it sees no investors and no signals | no |
| `list_pipelines`, `list_lists` | Whether the project has a round or a list yet, when the answer depends on one | no |
| Any read tool in the session | Answering a one-step question directly | no |
| The one write tool a one-step change needs | Only after its card is approved (house rules) | no |

Never write to find something out, never spend Hunter quota (`find_email`, `verify_email`,
`domain_search` are one-step changes: card first), and never create a draft or a link. The only
write help makes is the single change the user asked for, on approval.

## Steps

1. **Understand the question.** Restate it to yourself as a job: what the user wants to end up
   with, about what (a firm, a person, a deal, a round, a list), and how soon. Sort it:
   - **one step** ("what stage is Lantern on?", "who are the partners at Northwind?", "move
     Lantern to Term sheet", "note that Sarah wants Q3 numbers") → no command: answer it with
     the read tool, or show the change as a card (step 4). One firm, one deal, one person, one
     field; anything that needs a choice made, research or a second source is a task;
   - **a task** ("get emails for the partners at Kiln") → the command that does it;
   - **a situation** ("I've got a call at 3", "an investor said we're too early", "the round
     launches Monday") → what to do about it, which may be two or three commands in order;
   - **what next / where do I start** → read the setup (step 2) and recommend from it;
   - **what does X do / X or Y** → explain from the skills' own descriptions, then say when to
     use which;
   - **something went wrong** → the troubleshooting table, then the command that recovers;
   - **about the product or the app** → `about_yellowtail` and the help centre.
   Names in the question (a firm, a list, a round) are carried into the suggested command. Only
   when the question could mean two quite different jobs, ask one short question with the two
   readings as options; otherwise pick the likelier reading and say which you took.
2. **Check where they stand**, only as far as the answer needs. `whoami` almost always (it is
   cheap and settles signed-in, project and write access). Then only what the recommendation
   hinges on: `get_project` before anything about finding investors or signals;
   `list_pipelines` before anything that needs a round; `list_lists` before anything that needs
   a list. A missing prerequisite becomes step one of the answer, not a footnote: "You don't
   have a round yet, so first `/yt:pipeline create`, then `/yt:pipeline add`."
3. **Pick the commands.** Match the job against every skill in the session, using the map
   below for the usual routes and the skills' own descriptions for everything else. Prefer the
   one command that does the whole job over a chain; prefer a read-only one when the user only
   asked to see something. On an advisor's Clients pipeline or for selling Yellowtail Raise, the
   `yt-raise` commands are the answer where it is installed (`/yt-raise:help` covers that side);
   where it is not, say the job belongs to Yellowtail Raise and stop there. Desk work (orders,
   founder invites, signal intake, catalog merges) is the Yellowtail team's `yt-admin` plugin,
   never recommended here.
4. **Answer** in the format below. For one step, do it: a read is answered in a line or two
   (`show_results` on that read, its one card), ending with the command that goes further when
   there is an obvious one ("for the whole picture, `/yt:pipeline deal Lantern`"); a change is one
   card, made on approval, then the same pointer. If nothing fits, say so plainly: name the nearest command and
   what it does not cover, or the app page that does it, or say it is not something Yellowtail
   does today (check `about_yellowtail`'s coming-soon list first). Never invent a command, a
   flag or a feature.

## The usual routes

Nine commands, each an area with actions: `/setup`, `/investors`, `/lists`, `/pipeline`,
`/meetings`, `/daily`, `/outreach`, `/round` and `/help`. The area alone works too (it picks the
action from the request), but the answer always names the action. Short names here; write them in
full (`/yt:investors find`) in the answer.

| The user wants to… | Best | Then | Needs first |
| --- | --- | --- | --- |
| Get started, check what's connected, switch project | `/setup` | the one it recommends | Yellowtail connected |
| Set up a new client startup | `/setup project [url]` | `/setup user`, `/pipeline create` | a workspace with room |
| Give the founder access | `/setup user` | | owner or admin of the workspace |
| Plan the round: size, instrument, sequence, dates | `/round strategy` | `/pipeline create` | a project |
| Start the round itself | `/pipeline create` | `/investors find` | a project |
| Find investors who fit | `/investors find` | `/pipeline add` or `/lists add` | project interests |
| Fill gaps in the catalog for a whole segment | `/investors prospect` | `/investors research` | |
| Research and file one firm, or enrich one | `/investors research` | `/investors contacts` | |
| Find the right partner and their email | `/investors contacts` | `/lists add`, `/outreach draft` | the firm in Yellowtail |
| A one-pager on a firm before first contact | `/investors brief` | `/outreach draft` | |
| Who decides at a firm, and the way in | `/investors map` | `/outreach draft` | a deal on the firm |
| Track people in a named group | `/lists create` (or `/lists add`, which makes the list too) | `/lists export` | |
| Add people to a list | `/lists add` | `/lists export` | |
| Export a list as CSV or to Hunter | `/lists export` | | a list; Hunter key for Hunter |
| Put firms on the round as deals, or import a list | `/pipeline add` | `/outreach draft` | a round |
| Write the first email or ask for an intro | `/outreach draft` | `/meetings schedule` | Gmail for drafts |
| Book a meeting | `/meetings schedule` | `/meetings prep` | Calendar; Calendly optional |
| Get ready for a meeting | `/meetings prep` (`/meetings qa` for the hard questions) | `/meetings summary` after | Calendar helps |
| Write up a meeting just held | `/meetings summary` | `/daily follow-ups` | notes or a transcript |
| Log something that happened on a deal | `/pipeline log` | | a deal |
| Change a deal: stage, value, owner, next step | `/pipeline update` | | a deal |
| Answer an objection or a tricky reply | `/outreach objection` | `/pipeline update` | |
| Chase who owes a reply | `/daily follow-ups` | | Gmail |
| Catch up on investor email | `/daily inbox` | `/daily follow-ups` | Gmail |
| Start the day / end the day / wrap the week | `/daily briefing` / `/daily end-of-day` / `/daily weekly` | the actions each names | a round |
| A hard look at one deal | `/pipeline deal` | `/round close` or `/pipeline update` | a deal |
| How the round is doing | `/pipeline review` | `/round forecast` | a round |
| Where the round will land, and when | `/round forecast` | `/round close` | a round with deals |
| From term sheet to money in the bank | `/round close` (`/round term-sheet` for the terms) | | a round |
| Understand or compare term sheets | `/round term-sheet` | `/round close` | the term sheet |
| Why investors passed | `/round passes` | `/round pitch` | passes on a round |
| Improve the deck | `/round pitch` | `/meetings qa` | the deck |
| Build or check the data room | `/round data-room` | `/meetings qa` | Google Drive |
| The monthly update to investors | `/outreach update` | | the founder's numbers |
| Clean up the data, find duplicates | `/pipeline hygiene` | | a project |
| Every client's round side by side | `/round portfolio` | `/yt-raise:client-health` | several projects |

## When something has gone wrong

| Symptom | Likely cause | Fix |
| --- | --- | --- |
| No Yellowtail tools at all, or `whoami` fails | The connector isn't signed in | `CONNECTORS.md`: `/mcp` → **yellowtail** → Authenticate (Claude Code), or Settings → Connectors (claude.ai) |
| A command can't find Hunter, lists or `create_project` | A tool group is missing from the connection | `/setup`: it names the group and gives `toolsets.hint`, the URL to reconnect with |
| Every write stops at the card | A read-only API key (`key.canWrite` false) | Connect by signing in instead, or use a key that can write |
| No investors or no signals for the project | The project has no interests | `/setup project` on it, or set interests in the project's Settings (help centre: `project-settings-general`) |
| "Hunter is not connected for this workspace" | No Hunter key | Workspace Settings → Integrations (help centre: `workspace-integrations`) |
| Drafts come from the wrong address, meetings in the wrong calendar | A connector is signed into another account | `/setup` shows each account and flags the wrong one |
| A command the README lists is missing, or behaves as an older version did | The plugin is out of date | `claude plugin update yt` (and `yt-raise`), then restart |
| Wrong project | The active project is another one | `/setup [project name]` |

## Help centre articles

For questions about the app itself. Link as `[helpCentre]/[slug]`, using `links.helpCentre` from
`about_yellowtail`.

| Topic | Slugs |
| --- | --- |
| Getting in | `signing-in`, `accepting-an-invitation`, `a-tour-of-the-window`, `workspaces-and-projects` |
| Home | `home-dashboard`, `ask-yellowtail` |
| Signals | `signal-feed`, `signal-card`, `signal-filters`, `signal-actions` |
| Investors | `find-firms`, `filter-investors`, `prospects`, `investor-record`, `add-private-investor`, `find-people`, `person-record` |
| Lists | `list-create`, `list-contacts`, `list-manage` |
| Pipelines | `create-a-pipeline`, `round-timeline`, `work-the-board`, `deal-record`, `manage-stages`, `import-a-list` |
| Notes and search | `capture-a-note`, `review-notes`, `command-palette` |
| Settings | `project-settings-general`, `project-settings-guests`, `workspace-members`, `workspace-integrations`, `your-account`, `notifications` |

## Output

Display: inline — one Yellowtail widget in the chat, cards under it (`${CLAUDE_PLUGIN_ROOT}/references/display.md` §2). A one-step question is answered with `show_results` on the read that answers it, the only card drawn (§ Only the answer is drawn).

Short, and the answer first.

```
**Best next step:** `/yt:investors find 15`
Finds the 15 firms that fit Kestrel's seed round, ranked on their recent deals, with the ones
already on your round set aside. Read-only; it hands the picks to the next command.

**Before that:** nothing. Kestrel has 9 interests and the "Seed 2026" round.
**Then:** `/yt:pipeline add` with the ones you pick (one deal per firm), and
`/yt:outreach draft` for the first emails.
**Needs:** web search. Gmail only for the drafts later.

Want me to run it?
```

- **The command, written out** with whatever the question supplied (the firm, the list, a
  number), in the form this environment shows (`/yt:...` in Claude Code).
- **Why this one**, one or two lines in the user's terms, and whether it writes (cards, then
  approval) or only reads.
- **Before that**: the missing prerequisite, if any, as its own command. Say "nothing" when the
  setup was checked and is fine; leave it out when it wasn't checked.
- **Then**: the natural next one or two. For a multi-step job, a numbered sequence of at most
  five, each line a command and what it gets them.
- **Needs**: connectors that matter, and what still works without them (house rules §9).
- **Or**: one alternative when it is genuinely close ("for a single firm in depth,
  `/yt:investors brief` instead").
- A help-centre link when the question was about the app.
- End with the offer to run it. Run it only on a yes.

**No question** → one line on where they stand (from `whoami`, and the project's round if any),
the recommended next step as above (no project or no interests → `/setup project`; no round →
`/pipeline create`; a round with no deals → `/investors find`; deals → `/daily briefing`), then
the README's command groups as a compact menu, one line per group, then **Or just ask**: one
line per group in `about_yellowtail`'s `whatYouCanAsk` (its `what`, then two of its examples in
quotes), so people see they can talk to Yellowtail without a command. Leave out a group whose
`toolset` the user can't have; never add an example that isn't there.

```
**Or just ask**
- Investors: "Tell me about Northwind Ventures." · "Who are the partners at Northwind?"
- Your round: "What is on the Seed 2026 board?" · "When is legal close?"
- Email: "Find the work email for Sarah Lee at Brightwater."
```

## What comes next

The command it recommended, once the user says go; after a one-step answer, whatever the user asks next.
