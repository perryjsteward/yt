---
name: add-to-list
description: Yellowtail Club — Add investors to a Yellowtail list. A list holds people, not firms, so "add Northwind" means adding Northwind's relevant contacts, picked by the project's sector and seniority, with the choice explained. Works on one name or a batch (a pasted shortlist, the output of /find-investors, a CSV of firms or people), creates the list on the same card when it does not exist yet, skips anyone already on the list, and offers /find-contacts for a firm Yellowtail has no people at. Proposes cards and writes only on approval. Use when the user says "add X to my list", "put these funds on Tier 1", "track these investors", "add the angels from last night to a list", or picks names from a shortlist to track.
argument-hint: "<list name> <firms or people…>"
---

# Add to a list

Puts people on a list in the active project. The rule that shapes everything here: **a list is a
set of people** (`list_entries.person_id`). The firm comes through the person. When the user names
a firm, the job is to choose who at that firm they would actually talk to, say why, and let them
change the choice.

Read `${CLAUDE_PLUGIN_ROOT}/references/house-rules.md` first.

## Tools

| Tool | For | Required? |
| --- | --- | --- |
| `whoami` | The user, active project, workspace | yes |
| `list_lists`, `get_list` | Find the list; read its entries so nobody is added twice | yes |
| `search_people` | A named person (`q`, `linkedinUrl`); everyone at a firm (`companyId` alone) | yes |
| `search_companies`, `get_company` | Resolve a firm (`q`, `domain`, `linkedinUrl`); its people and tags | yes |
| `get_project`, `list_tags` | The project's interests (tag ids) and their names, for picking people by sector | yes, for firms |
| `create_list` | A list that does not exist yet: `{ projectId, list: { name, description? } }` | on approval, when the list is new |
| `add_to_list` | One entry: `{ listId, entry: { personId, notes? } }`. `listId` must be a real id from `list_lists` or `create_list`, never a name | yes, on approval |

There is no remove tool on a customer connection, so a wrong entry stays until someone takes it off
in the app. That is why every entry is on a card first.

## Steps

1. **Orient.** `whoami`; the project is `activeProjectId` unless the user named another.
2. **Find the list.** `list_lists { projectId }` and match the user's name.
   - **One match:** `get_list { id }` and keep the `personId`s already on it.
   - **No match, or the project has no lists at all:** the list is **new**. Don't stop, don't send
     the user to `/create-list` and never call `add_to_list` with a name: the card proposes the list
     and its people together, and one approval creates both (step 9). Name it as `/create-list`
     does: the user's name verbatim, trimmed, 1 to 120 characters; if they gave only a purpose
     ("track the angels from last night"), suggest a short name and say it is a suggestion.
     Description only from what the user said about it.
   - **A near match** ("Tier 1" when "Tier 1 leads" exists): ask under **Needs your input** whether
     they meant that one or want a new list, before proposing people.
   - **Several plausible matches:** ask.
   - **No list named** ("track these investors"): pick the one obvious list if there is one, ask if
     there are several, and propose a new one with a suggested name if there are none.
3. **Read the input.** One item per firm or person. For a pasted table or CSV, say in one line how
   you read the columns. Over about 15 items, resolve them all first (Yellowtail only, no web) and
   report the split before proposing: "20 firms: 16 found with people, 3 found with no people,
   1 not in Yellowtail."
4. **Resolve each item.**
   - **A person:** `search_people` by name (within the firm when given) and by LinkedIn URL. A
     same-named person at a different firm is someone else or a job move; ask, never assume.
   - **A firm:** `search_companies` by name, domain and LinkedIn URL. Ambiguous → a question.
     Not in Yellowtail → under **Not added**, offering `/create-investor`.
   - **A firm's people:** `search_people { companyId }` lists everyone there.
5. **Pick people at each firm** (`${CLAUDE_PLUGIN_ROOT}/references/house-rules.md` §6 applies: never invent a fit).
   - Read the project's interests once: `get_project`, then `list_tags` per kind to turn the ids
     into names (industry, focus, stage, geography).
   - Prefer people whose title makes them a decision-maker for this round (Partner, General Partner,
     Managing Partner, Principal; Investment Director at a CVC or family office), and among them
     the ones whose sector matches the project's: a title, a profile note or a signal that names
     the sector. `get_company` shows the firm's recent signals and who they name.
   - Default to **one or two people per firm**, and say why on the card ("Partner; leads their
     climate deals, per her signal of 6 Oct").
   - If nothing tells the people apart (three partners, no sector evidence), list them all on the
     card as a question; don't pick at random.
   - Analysts and associates only when they are the only people there, or the user asks.
   - **No people at the firm:** don't add anything for it. Put it under **Needs contacts**, offering
     `/find-contacts <firm>`, and pick it up once that skill has filed people.
6. **Skip what is there.** Anyone already on the list goes under **Already on the list**.
7. **Notes (optional).** If the user gave a reason or context ("met at Founders Forum", "intro via
   Sarah"), put it in the entry's `notes`. Otherwise leave it empty.
8. **Propose** the cards, at most 15 firms per round. Say how many remain.
9. **Write** on approval. A new list first: `create_list`, and take its `id` from the result. If
   that fails, stop and say why; nothing has been added. Then one `add_to_list` per approved person
   with that `listId`. If one fails, say why and carry on with the rest.

## Card

One card per firm (or per person with no firm). People are lines on it, so `2: not Erik` drops one.
When the list is new, the header says so and **Already on the list** is left out:

```
**New list "Founders Forum angels"** in Acme Robotics, with 4 people at 3 firms
- Description: angels met at the Founders Forum dinner, 15 Oct
```

Otherwise:

```
**Add to "Tier 1 climate leads"** in Acme Robotics: 5 people at 3 firms, 1 needs contacts

### 1. Northwind Capital
- **Sarah Lee**, Partner: leads their climate deals (signal, 6 Oct)
- Notes: met at Founders Forum

### 2. Brightwater Ventures
- **Anna Holm**, Founding Partner: the firm is two people and she leads seed
- **Erik Lund**, Partner: energy software (title)

### 3. Halcyon Ventures
- **Raj Patel**, Partner · **Mia Chen**, Partner: both generalists. Pick one or both?

**Already on the list** · Tom Reid (Northwind)
**Needs contacts** · Lantern Capital: no people in Yellowtail. Run `/find-contacts Lantern Capital`?
**Not added** · "Greenfield Partners": not in Yellowtail. `/create-investor` to research it.

Reply with any of: `approve all` · `approve 1 2` · `2: not Erik` · `3: Raj only` · `reject 3`
```

## Output

Display: inline — one Yellowtail widget in the chat, cards under it (`${CLAUDE_PLUGIN_ROOT}/references/display.md` §2). Yellowtail's reads draw nothing on the way; this output is the one thing shown (§ Only the answer is drawn).

After writing, `Created **<list>**` first when the list is new, then one line per firm: who was added, with the list link `/lists/{listId}` once at
the top. Then anything that failed, with the reason, and the firms still waiting on contacts.

## What comes next

- `/find-contacts <firm>` for every firm under **Needs contacts**, then this skill again.
- `/add-to-pipeline` to put the list's firms on a round as deals (a whole list imports at once).
- `/investor-brief <firm>` before reaching out to anyone on it.
