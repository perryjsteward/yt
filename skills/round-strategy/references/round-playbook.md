# Round playbook

How good seed and Series A advisors run a round. Every number here is a **rule of thumb to adjust
with the founder**, never a market statistic. Where the plan needs a market figure (valuations,
round sizes, dilution), it comes from the user or from a source named with its date — never from
memory.

## 1. Sizing

- **Start from the milestones, not the market.** Ask: what must be true for the next round to be
  easy (revenue, retention, a product shipped, a pilot converted)? How many months does that take,
  realistically, with the team the plan hires?
- **Arithmetic.** Raise ≈ (months to milestones + buffer) × average monthly net burn over that
  period, minus cash already in the bank that isn't earmarked. Burn rises as the team grows, so
  use the hiring plan's month-by-month burn when the model gives it, not today's burn.
- **Buffer.** Rounds take longer than planned, and the next round should start with months left
  in the bank, not weeks. Common advice is to plan for runway well past the milestone date
  (founders often hear 18–24 months in total); state the buffer you used and let the founder move it.
- **Range, not a number.** A minimum that still reaches the core milestone, and a target. The
  minimum tells you when you can close; the target is what you pitch.
- **Too much is a risk too**: more dilution, a higher bar for the next round, and a lead who
  thinks the round is "too big for the stage". Say what the extra money buys.

## 2. Instrument and valuation

- **Priced equity round**: sets a valuation and a share price, comes with a board and investor
  rights, more legal cost and time. Usual for a lead-led seed in many markets and for Series A.
- **SAFE / ASA / convertible note**: fast, cheap, converts later, often with a valuation cap and
  sometimes a discount. Good for pre-seed and for rolling closes; stacked caps can surprise a
  founder at conversion — model it.
- **Local norms differ** (e.g. UK founders often need SEIS/EIS-compatible structures, which affects
  which instruments work; the US uses SAFEs widely). Ask where the company is incorporated and say
  which local rules you have not checked.
- **Valuation context**: only from the user's own data (prior round, offers in hand), or a dataset
  named with its date (e.g. a public report from a cap-table or data provider). Without one, say
  the valuation is set with the lead and list what the founder should benchmark against.

## 3. Sequencing and the tiered list

- **Lead first.** Followers ask "who's leading?". Until there is a lead (or a clear price), Tier 2
  and 3 conversations stall. Spend the first weeks almost entirely on lead candidates.
- **Lead candidate** = leads rounds at this stage (`leads_rounds` true, or evidence of led rounds),
  cheque size fits a meaningful share of the round, sector and geography fit, active recently (a
  new fund, a led deal in the last months), and no portfolio conflict.
- **Tier 1 (8–15)**: the best lead candidates, approached first and close together, so meetings
  and partner meetings line up in the same weeks.
- **Tier 2 (15–30)**: strong fit, likely to follow or co-lead; approached a week or two after Tier 1
  so their process overlaps with Tier 1's later stages.
- **Tier 3 (fill)**: angels, operators, strategics, smaller funds — once the lead and terms exist.
  Angels with relevant reach can come earlier if they help with intros.
- **Warm beats cold.** For each Tier 1 firm, the plan names the intro path: a mutual (portfolio
  founder, co-investor, operator), and the partner to target. A forwardable blurb (`/draft-outreach`)
  makes the mutual's job easy.
- **Fit evidence**: tag or attribute in Yellowtail, a signal with its date, or a sourced fact. "Seems
  like a fit" is not evidence.

## 4. Timeline and momentum

- **Prep (2–4 weeks)**: deck, model, data room, references, target list, intro paths.
- **Pre-marketing (optional)**: low-key catch-ups with friendly investors for feedback, no ask.
- **Outreach window (concentrated, ~2 weeks)**: all Tier 1 intros go out together, so first meetings
  cluster. Spread-out outreach is how a round loses momentum.
- **Meetings → partner meetings (3–6 weeks)**: follow-ups within 24 hours, data room on request,
  updates to engaged investors when something good happens.
- **Term sheet → close (4–8 weeks)**: diligence, legal docs, filling the round, closing.
- **Momentum**: a deadline is credible only when there is demand behind it. Never invent interest;
  do share real progress (a term sheet, a strong metric) with engaged investors.
- If the window ends with no lead, stop and diagnose (story, metrics, targeting) rather than spraying
  Tier 3.

### Dating it backwards: is the close feasible?

Work from the date that matters (usually legal close: money in) back to today before promising
anything. Each step takes what it takes, and legal close is the step most often underestimated.
Typical lead times from signed term sheet to money in, to be confirmed with the company's lawyer:

| Jurisdiction and instrument | Typical | Why |
| --- | --- | --- |
| UK, SAFE / ASA (or BVCA-style short form) | 1–3 weeks | no notary; SEIS/EIS advance assurance can add weeks if needed first |
| UK, priced equity | 4–8 weeks | subscription and shareholders' agreements, new articles, Companies House filings |
| US, SAFE | days to 2 weeks | standard form, e-signed |
| US, priced (NVCA) | 4–8 weeks | full document set, board and stockholder consents |
| Netherlands B.V., priced | 6–10 weeks | share issue needs a **notarial deed**; notary scheduling, KYC on every investor |
| Germany GmbH, priced | 6–10 weeks | **notarised** investment and shareholders' agreements; notary reads the documents aloud |
| Germany / Netherlands, convertible loan | 2–4 weeks | usually no notary until conversion |

Add holidays (late December, August in much of Europe), investor KYC and any board or shareholder
approvals. If the founder's date is earlier than the backwards dating allows, say so up front and
offer the realistic date, or what would have to be true to hit theirs. A date that depends on a
decision the founder hasn't made (which instrument, whether "close" means lead committed or money
in) is an **open decision**, written into the milestone's note.

## 5. Readiness checklist

| Item | Ready means |
| --- | --- |
| Deck | 10–15 slides for sending, tells the story without a presenter; `/pitch-review` passed |
| Financial model | Monthly, 18–36 months, burn and hires by month, assumptions stated |
| Data room | Incorporation docs, cap table, previous round docs, key contracts, IP assignment, team, metrics definitions, financials |
| Metrics pack | The traction metric with definitions and history the investor can check |
| References | Customers and former colleagues willing to take a call, briefed |
| Cap table | Clean and current, option pool stated, pro-forma for this round |
| Intro paths | A warm route named for each Tier 1 firm |
| Q&A prep | Answers to the hard questions (`/investor-qa-prep`) |
| Round terms | Size range, instrument, use of funds |
