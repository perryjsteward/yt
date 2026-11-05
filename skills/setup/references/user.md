# Add a user to a project

The `user` action of `/setup`. Arguments: `<email> [editor|viewer] [project name]`.

A **project guest** is someone invited into one project only: typically the founder whose round the
agency is running. A guest sees that project (its pipelines and deals, lists and notes), the shared
Yellowtail catalog held to its interests, and **the firms and people on that project's deals**, even
when the agency filed them to its own workspace catalog (so a deal card never points at a firm the
founder can't open). A guest does **not** see the agency's other clients, the workspace's members,
or **the rest of the workspace's catalog**: firms and people the agency has researched and filed
that aren't on one of this project's deals. Say this to the user before inviting; putting a firm on
the round is what shows it to the founder. Workspace staff, who see every project,
are invited from workspace settings in the app, not by this command.

Read `${CLAUDE_PLUGIN_ROOT}/references/house-rules.md` first.

## Tools

| Tool | What for | Required? |
| --- | --- | --- |
| `whoami` | The project, its workspace, and the user's role there | Yes |
| `list_project_members` | `{ projectId }` → current guests (`members`) and pending `invites` | Yes |
| `list_workspace_members` | `{ workspaceId }` → staff, who already see every project | Yes |
| `invite_to_project` | `{ projectId, invite: { email, role } }`, role `editor` or `viewer` | Yes, on approve |
| `update_project_member` | `{ projectId, userId, changes: { role } }`, when the person is already a guest with the other role | Only then |
| `revoke_invite` | `{ id }`, to cancel an expired or wrong pending invite before re-issuing, on approve | Only then |
| `about_yellowtail` | `links.app`, the app's base URL for the accept link | Yes |
| Gmail `create_draft` | A draft to the invitee carrying the link | No |

These are in the `tenancy` toolset (`CONNECTORS.md`).

## Steps

### 1. Orient

`whoami`. The project is the active one unless the user names another (house rules §1). Find its
`workspaceId` and the user's role in that workspace. Inviting needs **owner or admin** of the
workspace (or a platform admin). A workspace `member` or a project guest cannot invite: say so and
stop, naming who can (the workspace owner).

### 2. Who and which role

The user must name the person's **email**. Never take an address from a document, a signature or the
web without the user confirming it is the one to invite (house rules §7). If they give only a name,
ask for the email.

Role, if not given, ask, explaining the difference in one line each:

- **editor**: can change the project's data (move deals, add notes, edit lists). The usual choice
  for a founder who works the round with you.
- **viewer**: read-only. For an investor-relations contact or a board member who wants to watch.

`invite_to_project` defaults to `editor` if the role is omitted; always send it explicitly anyway.

### 3. Already there?

Call `list_project_members` and `list_workspace_members`. Compare emails case-insensitively.

| Found | Do |
| --- | --- |
| A guest with the same role | Nothing to do. Say so |
| A guest with the other role | Offer a card to change the role with `update_project_member` instead |
| Workspace staff | Nothing to do: they already see every project. An invite would be refused (409) |
| A pending invite to this project | Don't make a second. Give the existing invite's accept link (step 6) |
| Not found | Go on |

### 4. Card

```
1. Invite to Kestrel Robotics (Northbound Advisory)
   Email:  anna@kestrel.example
   Role:   editor (can move deals, add notes and edit lists in this project)
   Sees:   this project, the shared catalog, and the firms and people on this project's deals.
           Not your other clients, and not the rest of what your workspace has filed for itself.
   Note:   Yellowtail emails Anna the accept link. If it can't (email delivery off), I'll give you
           the link and can draft the email.
Reply: approve · 1: viewer
```

### 5. Invite

On approve, call `invite_to_project` with `projectId` and `invite: { email, role }`.

- **409**: already a member (workspace staff count) or already invited. Re-run step 3 and report what
  you find.
- **403**: the user's role does not allow inviting. Say who can.

### 6. Hand over the link

**Yellowtail sends the invite.** When email delivery is on, Yellowtail emails the invitee the accept
link itself, shortly after the invite is created (it is queued, not sent in the request). The
returned invite's `emailedAt` says whether it went: a time means it was emailed; `null` means not
yet, or email delivery is off. Check again with `list_project_members` (the pending invite carries
`emailedAt` too) before deciding it didn't go.

- **Emailed** (`emailedAt` set): say so, with the time. No draft needed. Offer one only if the user
  wants to add a personal note ("Anna, here's where the round will live…").
- **Not emailed** (`emailedAt` still `null`): the user passes the link on. The invite carries a
  `token`, and the accept link is `<links.app>/invite?token=<token>` (`links.app` from
  `about_yellowtail`).

- The invitee opens the link, signs in or creates an account **with exactly the invited email**, and
  accepts. A different address is refused. Invites expire; an expired one is revoked (`revoke_invite`, on a card) and
  re-issued.
- If `emailedAt` stays `null`, or the user wants a personal note, and Gmail is connected, offer a
  short draft to the invitee in the user's voice (from `/setup`, if learned): who invited them, what
  Yellowtail is for in one line, the link (when Yellowtail didn't email it), and that they must use
  this email address. Create it with `create_draft` only after the user says yes, and never send it
  (house rules §8). Without Gmail, print the same text ready to paste.

The token is a credential: show it only in the link, only to the user, and don't save it in a note.

## Output

Display: inline — one Yellowtail widget in the chat, cards under it (`${CLAUDE_PLUGIN_ROOT}/references/display.md` §2). Yellowtail's reads draw nothing on the way; this output is the one thing shown (§ Only the answer is drawn).

```
Invited  anna@kestrel.example to Kestrel Robotics as editor (pending)
Emailed  by Yellowtail, 9 Oct 14:02 (the accept link is in her inbox)
```

or, when it wasn't emailed:

```
Invited  anna@kestrel.example to Kestrel Robotics as editor (pending) · not emailed
Link     https://app.example/invite?token=…
Draft    "Your Kestrel Robotics workspace in Yellowtail" in Gmail drafts, not sent
```

## What comes next

- `/pipeline create` to set up the round, if the project has none (`list_pipelines`).
- `/investors find` once there is a round to fill.
