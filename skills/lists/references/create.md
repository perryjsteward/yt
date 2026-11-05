# Create a list

The `create` action of `/lists`. Arguments: `<list name> [description] [people or firms to add]`.

A list is a plain, named set of **people** a project is tracking: the contacts at the firms, plus angels
who have no firm. It has no stages and no values; that is what a pipeline is for. Lists track
contacts, deals are about firms, and a list can later be copied onto a pipeline as deals with
`/pipeline add`. This skill makes the empty list. Filling it is `/lists add`, which also creates the list
itself when it does not exist yet, on the same card as the people.

Read `${CLAUDE_PLUGIN_ROOT}/references/house-rules.md` first.

## Tools

| Tool | For | Required? |
| --- | --- | --- |
| `whoami` | The user, the active project, its workspace | yes |
| `list_lists` | The project's lists now, to catch a duplicate | yes |
| `create_list` | Create it: `{ projectId, list: { name, description? } }` | yes, on approval |
| `get_project` | The project's name and interests, to suggest a description | no |

All Yellowtail, `pipelines` toolset (plus `whoami`). If `create_list` is not listed, the connection
is read-only or the toolset is missing: say which (`whoami` → `key.canWrite`) and stop.

## Steps

1. **Orient.** `whoami`. Take the project from `activeProjectId` unless the user named another one.
2. **Name it.** Use the user's name verbatim, trimmed. `name` is 1 to 120 characters. If they gave
   only a purpose ("a list for the angels from last night"), propose a short name ("Founders Forum
   angels, Oct 2026") and say it is a suggestion. Never pad a name with the project name; the list
   already lives in the project.
3. **Check for a duplicate.** `list_lists { projectId }`. If a list with the same or a very similar
   name exists, show it with its `entryCount` under **Needs your input**: use that one, or make a
   new one anyway?
4. **Description.** Optional, plain text. Use what the user said about the list's purpose. If they
   said nothing, leave it empty; don't invent a purpose.
5. **Propose** the card below and wait for an approve.
6. **Create.** `create_list` with `{ projectId, list: { name, description } }`. Omit
   `description` when it is empty.
7. **Report** with the link `/lists/{listId}`.

## Card

```
**New list in Acme Robotics** (Northstar Advisory workspace)

### 1. Tier 1 climate leads
- Description: Lead-capable seed funds with a climate thesis, to work first
- People to add next: Sarah Lee (Northwind), Anna Holm (Brightwater): via /lists add

**Needs your input**
- **A.** "Tier 1 leads" already exists (12 people). Use it, or create this one as well?

Reply with any of: `approve all` · `1: <change>` · `A: <answer>`
```

Leave out the "People to add next" line when the user named nobody.

## Output

Display: inline — one Yellowtail widget in the chat, cards under it (`${CLAUDE_PLUGIN_ROOT}/references/display.md` §2). Yellowtail's reads draw nothing on the way; this output is the one thing shown (§ Only the answer is drawn).

One line: `Created **Tier 1 climate leads** in Acme Robotics: /lists/{listId}`.

If the user named people or firms in the same request ("make a Tier 1 list with Northwind and
Brightwater"), this is `/lists add`'s job, not this one: it proposes the new list and its people
on one card and creates both on one approval. Hand over before proposing anything here.

## What comes next

- `/lists add`: put people on it (a firm becomes its relevant people).
- `/investors find`: build the shortlist that goes on it.
- `/pipeline add`: once the list is ready, copy it onto a round as deals.
