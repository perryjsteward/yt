---
name: draft-outreach
description: Yellowtail Club — Draft investor outreach for a raise — a forwardable intro blurb a mutual contact can pass on (for a warm intro), or a short cold email anchored on the investor's own thesis or a recent Yellowtail signal. Personalised from Yellowtail (the person, their firm, its signals, the deal and its history) and the project, in the user's own voice, or in the founder's when drafting from the founder's mailbox (their Founder voice note). One warm ask to a connector can cover several target investors, picked from Yellowtail and drafted with what is known. Creates Gmail drafts only, never sends, and logs the outreach as a comment on the deal after approval. Batch mode drafts several investors at once, each one genuinely personalised. Use when someone says "draft outreach to [firm]", "write to [partner]", "intro blurb for [firm]", "forwardable email", "ask [mutual] for an intro to [firm]", "cold email these Tier 1 investors", or "email the first ten on the round".
argument-hint: "<firm or person>[, <firm or person>…] [--warm via <mutual>] [--cold] [--as-founder]"
---

# /draft-outreach

The email an investor decides on in ten seconds. A warm intro converts far better than a cold
email, and the best warm intro is a **forwardable blurb**: a short, standalone note the founder
sends to a mutual, which the mutual forwards to the investor unchanged. A cold email earns a read
only when its first line is about *the investor* (their thesis, a deal they led, something they
said) and its second line gives a concrete reason to care. This skill writes both from what
Yellowtail knows, in the sender's voice, and never sends.

Read `${CLAUDE_PLUGIN_ROOT}/references/house-rules.md` first.

## Tools

| Tool | For | Required? |
| --- | --- | --- |
| `whoami`, `get_project` | Active project; the startup's description | yes |
| `search_companies`, `search_people` | Resolve each name the user gave | yes |
| `get_company` | Firm: description, tags, attributes, people, five recent signals | yes |
| `get_person` | Title, email and `emailStatus`, LinkedIn | yes |
| `search_signals` | The firm's or person's recent signals (`companyId` / `personId`, `from` 180 days back) — the best hook | yes |
| `list_pipelines`, `list_deals` (`companyId`) | The deal for this firm on the round | yes |
| `get_deal` | Contacts, notes and comment thread: has anyone written already? | when a deal exists |
| `comment_on_deal` | Log the outreach — card first | on approve |
| `update_deal` | `{ id, changes: { nextStep, nextStepOn } }`: the follow-up date, on the same card | on approve |
| `list_project_notes`, `search_notes`, `get_note` | The saved writing-voice note from `/setup`; the client's **Founder voice** note when drafting as the founder | optional |
| `get_list`, `list_companies` | Likely targets for a warm ask: firms on the round, firms in the connector's portfolio's co-investors | warm mode |
| Email (Gmail `search_threads`, `get_thread`, `create_draft`) | Prior threads, the sender's voice, the draft | optional; paste-ready text otherwise |
| Web | A thesis quote, a recent post or deal, when Yellowtail has no hook — sourced | optional |

## Steps

1. **Orient.** `whoami`, `get_project`. Find the round with `list_pipelines` (ask if several).
2. **Resolve each recipient.** A person → `search_people`, then their firm. A firm → `get_company`
   and pick the partner to target: the person whose focus matches the startup's sector, from the
   firm's people, the deal's primary contact if there is one, or ask. Ambiguous match → ask; never
   write to a guess.
3. **Read the relationship.** `list_deals` with `companyId`; `get_deal` for its comments, notes and
   `nextStep` (or, on an old deal with none, the newest `**Next:**` line in its comments).
   Search email for prior threads with the person (open the full thread, never judge from a
   preview). Already in conversation → this is a follow-up, not outreach; say so and draft that.
4. **Warm or cold?** The user says, or: a mutual named (`--warm via`), or a deal comment / note naming
   an intro path → warm. Otherwise ask once: "Anyone who knows <partner> well — a portfolio founder,
   co-investor, former colleague?" No path → cold. (Yellowtail's Network room, which will find
   mutuals, is not built yet.)
   **Warm mode drafts with what it has.** One ask to a connector can cover **several** target
   investors: don't stop to ask which firms if Yellowtail can suggest them. Pick likely targets from
   Yellowtail: firms already on the round (`list_deals`), and firms that co-invested with the
   connector or their portfolio companies (the connector's firm via `get_company`, its
   `notable_investments` and deal signals; firms named in those signals). Rank them by fit with the
   round, put two to four in the ask, and say why each. Draft the whole email with what is known;
   placeholders only for what is truly unknown, each marked `[to confirm]` ("we met at
   [to confirm: where]"). Never stop at an outline because one detail is missing.
5. **Find the hook** — one specific, verifiable reason this investor, now, in priority order:
   a Yellowtail signal (summary + date), a deal or thesis statement on the firm record, a portfolio
   company adjacent to (not competing with) the startup, a recent post or talk found on the web
   (link it). No hook you can cite → say so; a generic opener is worse than a short honest one.
   **Check conflicts**: a portfolio company competing with the startup is a reason to flag, not
   to write.
6. **Learn the voice.** Whose mailbox is it going from?
   - **The founder's** (`--as-founder`, or an advisor drafting for the client to send): read the
     client's **Founder voice** project note, if one exists (`list_project_notes { projectId,
     query: { q: "Founder voice" } }` or `search_notes`, then `get_note`; its body starts
     `Founder voice:`). Match it. None → say so, write plainly, and suggest saving two or three of
     the founder's own emails as that note. **The first batch goes to the founder for approval**
     before anything is sent: say so on the cards, and offer one email to the founder with the
     drafts for them to approve or edit.
   - **The user's own**: the voice from `/setup` if this conversation has it, or the saved "Writing
     voice for <user>" note (`list_project_notes`); otherwise read 3–5 of the sender's recent sent
     emails (Gmail `search_threads` with `in:sent`), or ask for a pasted example.
   Match length, greeting, sign-off, formality. Never sound like a template.
7. **Draft** (formats below). Facts about the startup come from the project, the deck or the user,
   never invented: no traction number, customer name, or "oversubscribed" that you were not given.
8. **Show the cards** (one per recipient), take the reply, then: create the Gmail draft(s) and post
   the `comment_on_deal`, and set the deal's next step with `update_deal { id, changes: { nextStep:
   "Follow up with <name> if no reply", nextStepOn: <7 days on> } }`. Warm intro: the draft goes **to the mutual**, with the blurb below a
   two-line personal ask. Cold: the draft goes to the investor's email from Yellowtail; an
   `unverified` email is shown with its status and source, and the user decides.

### Forwardable blurb (warm)

- Ask the mutual first (double opt-in): "Would you be up for introducing me to <partner>? Blurb below
  to forward if so — no worries if not." With several targets: "Would you be up for introducing me
  to one or two of these? <Partner, Firm: why, one line> × 2–4. Blurb below to forward to whichever
  you know best." One blurb serves them all; a per-firm line can follow it.
- The blurb stands alone, written **about** the company in the founder's voice, 80–120 words:
  subject line that says what it is ("Intro: Fernway — <one-line what> — raising seed"); one line on
  what the company does; one or two lines of proof (traction, team, notable customer — given facts
  only); why this investor (the hook); the round in one line (size, stage, lead wanted or not);
  the deck link if the user provides one.

### Cold email

- Under 120 words. Subject 4–7 words, about them or the hook, not "Investment opportunity".
- Line 1: the hook, specific to them. Line 2–3: what the company does and the proof. Line 4: why it fits
  their thesis. Ask: one low-friction CTA ("Open to a 20-minute call next week?"). Signature.
- No "hope this finds you well", no "I wanted to reach out", no attachments by default.

### Batch mode

Several names → resolve all first, then propose at most **10 cards per round**. Each card has its own
hook; if two drafts would be interchangeable after swapping the firm name, one of them lacks a hook —
say so on that card rather than padding it. Warm and cold mix freely in one batch.

## Card format

```
1. Warm ask — Tom Reid (Ashgrove Angels, an existing backer) → tom@ashgrove.co · from the founder's mailbox
   Targets: Sarah Lee, Northwind (on the round; co-invested with Ashgrove in Gridly, signal 3 Jun) ·
            Raj Patel, Kiln Climate (co-led Ampwise with Ashgrove, Aug 2026)
   Voice:   Priya's Founder voice note (12 Oct) · first batch: to Priya for approval before sending
   Body:    <the ask, then the blurb> · "[to confirm: the Q3 number you want to quote]"

2. Cold — Maya Okafor, Partner, Northwind Capital → maya@northwind.vc (unverified · firm team page)
   Hook:    led Coldline's pre-seed (Yellowtail signal, 14 Sep 2026) — same buyer, not a competitor
   Subject: Coldline's buyer, one step upstream
   Body:    <the full email>
   Then:    Gmail draft (not sent) · comment on deal "Northwind Capital" (Seed 2026, stage Researching):
            "**Outreach drafted** (cold, 9 Oct 2026) to Maya Okafor — subject: Coldline's buyer…
             Hook: Coldline pre-seed signal. **Next:** send, then follow up 16 Oct if no reply."
            · next step: none → "Follow up with Maya if no reply" · due 2026-10-16

Reply: approve all · approve 1 3 · 2: <feedback> · reject 2
```

No deal for the firm → the card says "not on the round; nothing logged" and offers `/add-to-pipeline`.
Never create a deal from this skill.

## Output

Display: inline — one Yellowtail widget in the chat, cards under it (`${CLAUDE_PLUGIN_ROOT}/references/display.md` §2). Yellowtail's reads draw nothing on the way; this output is the one thing shown (§ Only the answer is drawn).

After writing: one line per card — draft link (or paste-ready text if no email connector), the
deal comment written, anything refused. Remind once: nothing has been sent. When the user later says
a draft was sent, offer to add a short "Sent" comment on the deal (card first).

## What comes next

- `/investor-qa-prep <firm>` and `/call-prep` once a meeting is booked.
- `/update-deal` to move the deal to the outreach stage once sent.
