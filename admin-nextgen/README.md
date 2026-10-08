# Admin next gen: concept proposal

> STATUS: EXPLORATION, NOT RATIFIED. This folder does not replace
> `docs/design/admin-standard/admin-prototype.html`, which stays the
> founder-ratified pixel truth. Nothing in `app/admin` or `packages/ds` was
> changed. Open `prototype.html` in a browser (hash routes `#today`, `#lens`,
> `#risk`, `#studio`; the "Night shift" button switches theme).

Inputs: live snapshot of admin.swoobz.com (2026-10-07), the admin data
inventory and datamap (staging `4119f20b5a`), the creator brand kit, a
screenshot of the current home, and a read of `app/admin`, `packages/ds`,
`packages/bonus-engine`, `packages/freebets`, `packages/risk-scoring` and
`packages/engagement-segmentation`.

## 1. Is the data enough?

**For "what exists and where each number comes from": yes, very good.**
The inventory maps every route, datapoint, source table and gap. That is
rarer than good screenshots and it is the base for honest design.

**For the design itself, these inputs are still missing:**

| Missing input | Why it matters |
|---|---|
| Screenshots of the key pages (player dossier, cash bonus, bonus codes, affiliates, performance, player intelligence) at 1440 and 390 | To judge density and composition per page, not only the home |
| Who works in the admin, per role, and their top 5 jobs per day | IA must follow real jobs: support, compliance, finance, growth, community |
| Product decision on wagering (see section 4) | The bonus studio looks very different with or without wagering |
| USDC launch scope: which bonus types at launch, which countries | Decides which reward types are live, locked or hidden |
| Approval thresholds and budgets (per role, per reward type) | Needed for the dual-control rules in the studio |
| May the new direction move the ratified standard (IA, hubs, dark theme)? | Decides if this is an extension or a new version of the standard |
| One agreed definition per KPI (example: "new players 7d" is 35 on /players and 62 on /performance) | A data center is only useful if each number has one meaning |

## 2. What the snapshot tells us (design problems, not only bugs)

1. **The console does not say what it can trust.** The home pulse never
   loads, the anomaly sentinel stopped updating around 2026-07-30, the
   Originals monitor fails, and "0 open anomalies" looks like "all fine".
   Today trust lives in the operator's head.
2. **The same player has two truths.** A player is "Dormant, risk 82" on
   /players/at-risk and "active 7 of 7 days, churn low" in the dossier.
   Segments are keyed on email, the event lake on player id.
3. **51 nav paths, one flat list.** Jobs are spread over pages that do not
   link: risk lives in /antifraud, /antifraud/queue, /ops/sportsbook-risk,
   /players/at-risk and /rg.
4. **Bonus tools are four separate forms.** Cash bonus, bonus codes,
   freebets and prize bands each have their own form, units (lamports, BPS,
   centi-odds, raw unix seconds) and no shared audience, budget or result.
5. **Scores are not explained.** Risk 82, engagement 9, fraud 75 are
   heuristics or constants, but the UI shows them like model output.
6. **Real signals get lost.** The scripted-looking stake burst on fixture
   14447773 and the 3 quests that can never complete were only found by
   reading tables by hand.

## 3. The concept: four ideas

### A. Today becomes a shift briefing (`#today`)
- **What changed since you last looked**: 3 to 5 cards, written as one
  sentence each, every card links to the case or page.
- **Waiting on you**: one ranked list across every queue the role can see,
  with SLA timers. Gated queues say "switched off, not checked", never 0.
- **Data health**: every source that feeds a number, with its last good
  refresh. A stale source greys out every KPI it feeds, console-wide.
- **Hand over shift**: a short note for the next operator, stored with the
  briefing.

### B. Player lens: one identity, one timeline (`#lens`)
- One identity spine (player id, email, wallet, guild, affiliate code) so
  segments, events and money join on the same key.
- Four scores, each with "why": engagement, churn, **care** (harm risk) and
  **threat** (risk to the house). Tap a score to see the named signals and
  their points.
- One timeline for every lane: play, rewards, care events, operator actions,
  support. Sources that disagree show side by side.
- Behaviour fingerprint: play mix, when they play (weekday x hour), median
  session, average stake, reward share of play, value to the house.
- **Audiences** are saved queries from the lens and the ask bar. The same
  audience object feeds the bonus studio and campaigns.

### C. Risk and care: two axes, one case board (`#risk`)
- Care and threat are separate questions and are never added up. A 2x2 board
  places every open case: care first, threat first, both (escalate), monitor.
- One case = all signals from antifraud, sportsbook risk, RG, KYC/AML and
  affiliate quality, with evidence (stake timeline, linked accounts graph)
  and actions (limit stake, freeze with 2 signers, mark reviewed).
- Every score is a sum of named signals with weights. No black box. This
  fits what exists today (`risk-scoring` factors, the fraud queue constants)
  and makes it honest.
- Hard rule: **care signals can only reduce marketing, never target it.**

### D. Bonus studio: write the offer as one sentence (`#studio`)
"Give [audience] [amount] [reward] when they [trigger], max [n] per player,
until [end]. Budget cap [x]."
- **One reward catalogue**: OP drop, free bet, raid booster, quest booster,
  cashback, cash bonus (locked until USDC launch). Same flow for all.
- **Live player preview**: the exact card and the one-sentence terms the
  player sees (rewards journey rule: terms in one sentence on the offer and
  in the wallet).
- **Pre-flight**: audience size, eligible after guardrails, expected cost
  with a P10 to P90 range, share of budget, abuse exposure in plain words.
- **Guardrails, some locked**: care signals and restricted countries are
  always excluded; open threat cases excluded; second signer above a
  threshold; budget cap pauses the offer by itself.
- **Holdout by default (10%)**: every offer is measured against a random
  group that did not get it. The results table shows D7 uplift and cost per
  extra active player, so the team learns which offers really work.
- Lifecycle: write, pre-flight, second signer, live, results.

### Across all hubs
- **Ask bar (Cmd K)**: ask in plain words, get the answer plus the query,
  sources and freshness. Any answer can be saved as an audience.
- **Eight hubs instead of 51 links**: Today, Players, Risk and care,
  Rewards, Money, Community, Growth, Platform.
- **Real vs sample** is labelled in the prototype. In the product, the
  same idea becomes the freshness and coverage stamp on every panel.
- **Two themes**: day = the ratified light standard; night shift = the
  brand kit (ink, slate, one cyan accent). Admin today uses Geist; the
  prototype uses the brand fonts to test the feel, this is open.

## 4. Decisions needed (product forks)

1. **Wagering.** The brand kit promises "No wagering requirements. No locked
   bonus money." The code models wagering: `cash_bonuses` requires a
   wagering multiplier (CHECK > 0), bonus-code campaigns allow one, and
   `packages/bonus-engine` forfeits bonus on withdrawal. Choose one: keep
   wagering for USDC bonuses (and change the brand line), or build the
   studio without wagering (OP and cash rewards with no rollover).
2. **Care vs marketing.** Confirm the hard rule that care signals only ever
   remove players from offers.
3. **Holdout by default.** Confirm a 10% holdout on every offer, even when
   that means fewer players get it.
4. **IA move.** Confirm eight hubs may replace the current group list, or
   keep the groups and add the new hubs inside them.

## 5. Suggested order (smallest slices)

1. Data health + tile truth on Today (no new data, only honest states).
2. Player identity spine (fixes the email vs id split; needs data work).
3. Case board read-only (merge existing risk reads into one list).
4. Bonus studio for OP drop + free bet (both exist today), with holdout.
5. Ask bar on top of saved, reviewed queries.

Every step goes through the normal admin gate (`admin-design-qa-maker` and
verifier). Steps that touch grants, limits or freezes are high risk under
CLAUDE.md rule 5 and need the money-path and security verifiers.

## Files

- `prototype.html`: static, self-contained, no build. Numbers marked
  **Real** come from the 2026-10-07 snapshot; numbers marked **Sample** are
  made up to show the design. The pre-flight math is a toy model with stated
  assumptions, not a forecast.
