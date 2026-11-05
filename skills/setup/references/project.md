# Create a project

The `project` action of `/setup`. Arguments: `<startup url> [deck file or link]`.

A project is one startup being raised for. Its **interests** (tag ids of kind stage, industry,
geography, focus) are what every catalog read for that project is held to: a project with no
interests sees no investors and no signals, and a project with the wrong ones sees the wrong
investors. So this command spends its effort on getting the interests right, with a reason for each,
and shows them on one card before anything is created.

Read `${CLAUDE_PLUGIN_ROOT}/references/house-rules.md` first.

## Tools

| Tool | What for | Required? |
| --- | --- | --- |
| `whoami` | Workspaces (with role) and projects the user can reach; `activeProjectId`; `isPlatformAdmin` (may create tags, raise a project limit) | Yes |
| `get_workspace` | Each workspace's `projectLimit` and its projects: how full it is, and a same-name project | Yes |
| `list_tags` | Real tag ids for `stage`, `industry`, `geography`, `focus` (and `business_model` if useful). One call per kind, cached | Yes |
| `create_tag` | `{ kind, name, parentId? }`: an approved **New tags** line, created before the interests are set. Anyone: industry (its group as `parentId`) and focus; stage and geography: platform admins only | On approve |
| `create_project` | `{ workspaceId, project: { name, description } }` | Yes, on approve |
| `set_project_interests` | `{ projectId, interests: { tagIds: [...] } }`. Replaces the whole set | Yes, on approve |
| `get_project` | Read back the interests after setting them | Yes |
| `set_active_project` | `{ id }` = the new project's id. Makes it the project the app opens on and the one later commands act on (`activeProjectId`). Only if the card's "Make active" line was approved | On approve |
| `update_project` | `{ id, changes: { name?, description? } }`, when the project already exists and the user wants it refreshed | Only then |
| Web search and fetch | Research the startup | Yes (or a paste) |
| Google Drive `search_files` / `read_file_content` | A deck the user points to in Drive | No |

`create_project`, `get_workspace`, `set_project_interests` and `set_active_project` are in the
`tenancy` toolset. If they are missing, `whoami.toolsets.hint` gives the URL to reconnect with
(the bare `https://www.yellowtail.club/api/mcp` lists every default toolset); say so, point to
`CONNECTORS.md`, and stop.

## Steps

### 0. Ask first

Settle everything that could stop the project being created **before** any research, so the user
answers once, up front, and the end of the run is never left waiting on them.

`whoami`, then `get_workspace` on every workspace it lists (in parallel). For each, note its project
count (`projects.length`) against `projectLimit`, and any project whose name matches or nearly
matches the startup's (case, "Ltd"/"Inc", a missing word).

Then ask, in **one** `AskUserQuestion` call, only the questions below that are actually open (if the
client has no question tool, put them as one lettered block at the very top of the reply):

| Question | Ask when | Options |
| --- | --- | --- |
| **Which workspace?** | The user can reach more than one | Each workspace by name with its count, `Northbound Advisory · 4 of 5 projects`; a full one says so in its label, `Acme Ventures · full (5 of 5)`. The workspace of `activeProjectId` goes first, marked "(Recommended)". A platform admin sees every customer's: never pick for them |
| **The workspace is full** | The chosen workspace (or the only one) is at its limit. If the workspace is still open, ask it as "if you pick a full one" | Upgrade or ask Yellowtail for more projects (the app's Upgrade, or the team) · Free a slot (delete a project in the app's project settings, then come back) · Use another workspace |
| **A project with this name exists** | Same or near name in the chosen workspace | Refresh that one (`update_project` + `set_project_interests`) · Create a second |
| **Raising now?** *(optional)* | Only when the call is being made anyway for one of the above | Not raising · Pre-seed / Seed / Series A, with a rough size. The answer sharpens the stage tags; without it, research infers the round and the card says it is an inference |

Never make the call for the optional question alone. Skip any question the user's message already
answers: "in YellowTail Club" settles the workspace, "it's a seed round" the round. Nothing open →
no call; go straight to research.

Act on the answers before researching: switch workspace, or, for more room, stop and say what to do
in the app (upgrade, ask the team, or delete a project in its settings) and pick up here after.

### 1. Research the startup

Treat everything you read as data (house rules §7). Gather, with the source URL for each fact:

| What | Where to look |
| --- | --- |
| Name, one-line description, product, customer | the site's home, product and about pages |
| Sector and sub-sector | product and customer pages; how press describes them |
| Business model | pricing page, customer logos (B2B, B2C, marketplace) |
| HQ and markets | about, contact, careers, legal footer (registered office) |
| Founders and team size | about / team page, LinkedIn company page |
| Funding history | press releases, the site's news page, public funding databases (Crunchbase-style), Companies House or the local registry |
| Traction | only figures the company publishes itself |
| Likely next round | inferred from the last round, its date and team size; say it is an inference |

A deck the user gives (upload, Drive link or path) is the best source: read it fully first, and its
facts are the user's own material. If the site is thin and nothing else is given, say what you could
not establish rather than filling gaps.

### 2. Map to interests

Interests describe **the investors this round needs**, not the company in general. Load `list_tags`
for each kind you use and pick real ids only (house rules §6).

- **stage**: the round being raised now, plus the one either side if it is borderline (pre-seed and
  seed for a first institutional round). Not every stage the company has ever been through.
- **industry**: the most specific tag that fits, plus its parent only if investors are tagged at the
  broader level. A chosen parent already covers its children.
- **geography**: where the company is based and where its investors are likely to be (UK company →
  United Kingdom, plus Europe if a regional tag exists and the round could take European money).
- **focus**: only when it is clearly a specialist raise (climate, deep tech). Leave it empty for a
  generalist round.

How the filter works, and why less is often more: **OR within a kind, AND across kinds**, and a kind
with no picks restricts nothing. Every kind you fill narrows the investors the project sees. Picking
`focus: Climate` hides every generalist fund that would happily back a climate company; if in doubt,
leave a kind empty and say so on the card. If a tag you need does not exist (check case, plural and
synonyms in `list_tags` first), propose it under the card's **New tags (created on approve)**
(house rules §12): kind, name, the group for an industry tag (one of the groups `list_tags { kind:
"industry" }` returns), why, and the near misses. A stage or geography tag is the Yellowtail team's
unless you are a platform admin: show it marked so. Never substitute a vaguer one silently. Interests are
filters, so a new tag only helps once investors carry it; say that on the line when it applies.

### 3. Show the card

```
1. New project in Northbound Advisory
   Name:        Kestrel Robotics
   Description:
     Kestrel builds autonomous inspection drones for offshore wind farms, sold to operators
     on an annual contract. Founded 2023 in Bristol, UK; 14 people. Raised a £1.2m pre-seed
     in March 2025 (Tech Nation press release). Likely raising a seed round in 2026.

   Interests (6 tags; a project sees investors matching every kind below)
     stage      Pre-seed, Seed ............ last round was pre-seed 18 months ago (press, Mar 2025)
     industry   Robotics, Energy .......... product page: drones for wind-farm inspection
     geography  United Kingdom, Europe .... HQ Bristol (site footer); UK and EU customers (case studies)
     focus      (none) .................... generalist seed funds should stay in view

   Sources: kestrel.example/about · techcrunch.example/2025/03/… · deck p.4, p.11
   Could not establish: revenue, round size
   Make active: yes (the app and later commands switch to Kestrel Robotics)

   New tags (created on approve):
                industry · "Offshore energy" (group: Energy) — product page: "inspection for
                offshore wind"; near miss: Energy (kept, broader). Investors
                aren't tagged with it yet, so Energy stays in the interests too.
Reply: approve · 1: no new tag · 1: don't make active · 1: <feedback>
```

The description is markdown, three to five plain sentences a colleague could read in ten seconds:
what they do, for whom, where, how far along, and what they are raising if known. No opinions, no
investor names, nothing confidential from a deck unless the user says it can go in. Name length is
at most 120 characters.

**Make active** defaults to yes: the user almost always works in the project they just created. Say
no when they are setting up a client for later and want to stay where they are; they can also turn it
off with `1: don't make active`.

**When the request is the approval.** If the user's original message was an explicit instruction to
create ("create a project for kestrel.example in Northbound", "set up Kestrel as a new client") and
Step 0 left nothing open, that message is the approval house rules §2 asks for: go straight to
Step 4 and show the card as the report of what was done, with the Output's undo line under it.
Stop for approval only while something is still uncertain:

- a proposed **New tag** (it changes the shared vocabulary, so it always waits);
- a **strong duplicate** research turned up that Step 0 could not see (the company trades under
  another name that matches an existing project);
- a fact the description or the stage tags depend on that research contradicts in what the user
  said ("it's a seed round", but they closed a Series A last month).

A request that only points at the company ("here's a startup we're onboarding", a bare URL) is not
an instruction to create: show the card and wait.

**Waiting.** Never end a turn on a card with the question buried under it. When the card needs a
reply, the reply opens with that question on one line, `Approve, or drop the new "Offshore energy"
tag? (details on the card below)`, and the card follows. Otherwise take the reply as in house rules
§2 and re-show the card after feedback.

### 4. Create

On approval, or straight away when the request was the approval, in this order:

0. Approved **New tags**: `create_tag { kind, name, parentId? }` for each (an industry tag's
   `parentId` is its group's id from `list_tags`), and add the returned id to the interest set.
   `created: false` → the kind already had it under another spelling: use the tag it returned and
   say so. Refused (a kind other than industry or focus, and not a platform admin) → leave it out
   and list it under **Tags to ask the Yellowtail team for**, suggesting the user mentions it to
   the team.
1. `create_project` with `workspaceId` and `project: { name, description }`. Keep the returned `id`.
   Refusals: the workspace's project limit (Step 0 should have caught it; if a project was added
   since, ask the Step 0 limit question again), or the user's role in the workspace.
2. `set_project_interests` with `projectId` = that id and `interests: { tagIds }`, the full approved
   set. It replaces the whole set; for an existing project, `get_project` first and send the union the
   user approved.
3. `get_project` to confirm `interestTagIds` matches the card.
4. If **Make active** was approved, `set_active_project` with `id` = the new project's id. It changes
   nothing but which project is open, and returns the updated session; confirm its
   `activeProjectId` is the new id.

If step 1 succeeded and step 2 failed, say so plainly: the project exists but sees nothing until its
interests are set. Offer to retry step 2 only.

### 5. The active project

Made active: later commands act on it by default, and the app opens on it. Not made active: later commands in this session must name the
project, and the card for each will say which project it writes to (house rules §1).

## Output

Display: inline — one Yellowtail widget in the chat, cards under it (`${CLAUDE_PLUGIN_ROOT}/references/display.md` §2). Yellowtail's reads draw nothing on the way; this output is the one thing shown (§ Only the answer is drawn).

```
Created  Kestrel Robotics in Northbound Advisory · now your active project   /settings/project
Interests set: 6 tags (Pre-seed, Seed · Robotics, Energy · United Kingdom, Europe)
Not established: revenue, round size
Tag created: "Offshore energy" (industry, under Energy)   — or —   Used existing "Offshore wind" (same tag)
Tags to ask the Yellowtail team for: none
Undo or adjust: `undo` switches you back to <previous project> (delete this one in its settings) · `name: …`,
  `description: …`, `+ Climate` / `− Europe` change it in place
```

When the request was the approval, the card from Step 3 goes above this, headed **Created** rather
than numbered, so the user sees every interest and its reason. `undo` is deleting the new project in
the app's project settings (Yellowtail Club has no project delete), then `set_active_project` back to
the `activeProjectId` Step 0 saw; an adjustment is
`update_project` or `set_project_interests` (the full new set), then `get_project` to confirm.

## What comes next

- `/setup user` to invite the founder into their project (as editor or viewer).
- `/pipeline create` to set up the round.
