---
name: setup
description: Yellowtail Club — Get Yellowtail ready and set up a client, in one command with three actions - check (who you are signed in as, the workspace and project, what is connected, how you write), project (a new client startup set up from its website and deck) and user (invite the founder or a co-founder into a project as an editor or viewer). Use at the start of a session, after installing, or when someone says "set me up", "what's connected", "which project am I in", "new client", "set up a project for [startup]", "onboard [company]", "add the founder", "invite [name]", "give [email] access".
argument-hint: "[check | project | user] [the action's own arguments]"
---

# /setup

Get Yellowtail ready and set up a client: check the connection, create a project from a startup's website, invite a founder in. One command, with an action for each job; each action's procedure is a reference in this
folder, read only when that action runs.

| Action | What it does | Arguments | Reference |
| --- | --- | --- | --- |
| **`check`** | Get Yellowtail ready to work in this conversation | `[project name]` | `references/check.md` |
| **`project`** | Set up a new client startup in Yellowtail from its website | `<startup url> [deck file or link]` | `references/project.md` |
| **`user`** | Invite someone, usually the client founder or a co-founder, into one Yellowtail project as a guest (editor or viewer) | `<email> [editor\|viewer] [project name]` | `references/user.md` |

## Pick the action

1. The first word of the arguments names it (`/setup check …`); the rest are that action's own
   arguments. With no arguments at all, `/setup` is **`check`**.
2. No action named: match the request to the phrases below and take the closest.
3. Still unclear between two: ask one short question naming both. Never guess an action that writes.

- **`check`** — Use it at the start of a session, after installing the plugin, or when someone says "set me up", "what's connected", "which project am I in", "get started with Yellowtail" or "learn how I write".
- **`project`** — Use when someone says "new client", "set up a project for [startup]", "create a project from this URL", "onboard [company]", or pastes a startup's website or deck and wants it in Yellowtail.
- **`user`** — Use when someone says "add the founder", "invite [name] to the project", "give [email] access", "share the project with the client", or after /setup project.

Then read `references/<action>.md` and follow it exactly: its tools, steps, rules, output and next
steps are the action's own. Read `${CLAUDE_PLUGIN_ROOT}/references/house-rules.md` first, as every
action says.

## Output

Display: inline — unless the action's reference says otherwise (some actions are a dashboard or a document): its own `## Output` section is the display for that action (`${CLAUDE_PLUGIN_ROOT}/references/display.md` §2).
