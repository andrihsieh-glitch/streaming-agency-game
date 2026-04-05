# Streaming Talent Agency — Game Design Reference
> Implementation reference for Claude build sessions. Fetch this file at the start of every build chat.
> Live URL: https://raw.githubusercontent.com/andrihsieh-glitch/streaming-agency-game/refs/heads/main/DESIGN.md

---

## PROJECT OVERVIEW

**Genre:** Turn-based management game (weekly turns)
**Stack:** React + JS, browser-based PWA
**Tone:** Dark studio. Late-night creative operation. Characters have voice.
**Core loop:** Manage talent weekly → earn money → grow agency → navigate drama → reach an ending
**Emotional spine:** The Ren ↔ Hana relationship. Affected by almost every major decision.

---

## PLAYER CHARACTER — REN

- Name: player-defined (free text input, no default)
- City: player-chosen — Taipei / Tokyo / Seoul
- No other creation options — personality emerges through decisions
- Has one special action per stage: **Org Review** (see Org Structure)

---

## GAME STRUCTURE

### Stage Progression
```
Garage → Boutique → Mid-size → Corporate (ending only)
```

**Garage → Boutique triggers (all required):**
- 2+ talents with 10K+ subscribers
- Agency reputation ≥ 40
- Cash runway ≥ 8 weeks
- Hana relationship ≥ 60

**Boutique → Mid-size triggers (all required):**
- 4+ talents, at least 2 with 100K+ subscribers
- One successful group formed
- Agency reputation ≥ 65
- Hana relationship ≥ 40

**Mid-size → Corporate triggers:**
- 6+ talents, at least one with 500K+ subscribers
- Staff team of 4+
- Sustained agency revenue
- Investor pressure OR Hana departure (Corporate is an ending, not a phase)

### What Changes Per Stage
- Garage: 2 talent slots, no staff, no brand pipeline (only small sponsorships)
- Boutique: 3+ talent slots, first staff hire unlocks, brand pipeline opens
- Mid-size: full brand pipeline, investor events, org structure matters, Hana pivot fires
- Corporate: ending state only

---

## WEEKLY TURN STRUCTURE

Each week the player:
1. Assigns one **primary slot** per talent
2. Assigns one optional **secondary slot** per talent
3. Spends up to 2 **agency actions** (scouting, brand deals, staff, check-ins)
4. Reviews **event feed** from last week
5. Advances week → outcomes resolve → feed updates

---

## TALENT SYSTEM

### Talent Stats (per talent card)
```js
{
  id: string,
  name: string,
  handle: string,
  niche: string,
  homeCity: string,
  currentCity: string,
  subscribers: number,
  energy: number,        // 0-100, recovers weekly
  morale: number,        // 0-100, slow change
  skill: number,         // 0-100, grows via training/streams
  popularity: number,    // 0-100, grows via events
  audienceTrust: number, // 0-100, degrades with bad-fit deals
  burnout: number,       // 0-100, HIDDEN from UI
  bond: number,          // 0-100, relationship with Ren
  traits: string[],      // 2-3 trait keys
  memoryTags: string[],  // permanent consequence markers
  proximityState: 'local' | 'regional' | 'distant' | 'remote',
  chemistryWith: { [talentId]: number } // 0-100 per pair
}
```

### Traits

**Energy Type**
- `night_owl` — morale penalty for morning events; bonus for late streams
- `sprinter` — Push weeks give bonus but next week energy recovery reduced; cannot Push twice in a row without burnout accumulation
- `marathon_runner` — consecutive regular streams compound into slow growth bonus; Push weeks break streak

**Social Style**
- `extrovert` — solo-heavy schedules add passive morale drain; collab weeks give morale bonus
- `introvert` — heavy collabs add burnout; solo streams get small skill/morale bonus
- `performer` — events give morale bonus; training weeks give morale penalty

**Work Ethic**
- `perfectionist` — training gives 2x skill; Push without Extra Prep = morale hit
- `instinctive` — Extra Prep reduces performance; spontaneous schedule changes occasionally give bonus output
- `people_pleaser` — morale swings amplified in both directions

**Ambition**
- `hungry` — no subscriber growth for 2+ weeks = morale hit regardless of other factors
- `craftsperson` — bad-fit brand deals cause morale hit even when executed adequately; training has high morale payoff
- `coasting` — Push weeks give reduced output and higher burnout; regular streams very consistent

### Primary Slots
```
stream_regular    energy: -15  growth slow, morale depends on viewership vs expectation
stream_push       energy: -30  growth fast, morale risk if disappoints inflated expectation
rest              energy: +25  no growth, morale stabilizes
event             energy: -20  popularity+, morale depends on fit
collab            energy: -20  chemistry+ with partner, risky if chemistry < 40
training          energy: -10  skill+, no subscriber movement
brand_execution   energy: -15  fulfills contracted deal, cash in, morale depends on fit
```

### Secondary Slots
```
extra_prep        reduces morale risk on push/event; wasted on instinctive trait
social_push       small subscriber bump, -5 energy
personal_time     +5 morale, small burnout reduction
hana_1on1         Hana relationship+, may surface hidden talent state
offline_hangout   requires local proximity + bond >= 35; relationship+8, morale+5
```

### Viewer Expectation Mechanic
- Each talent has `viewerExpectation` (rolling average of recent performance)
- Stream below it: morale hit
- Stream above it: morale bonus + subscriber spike
- Push streams raise the expectation ceiling over time (the treadmill)

### Burnout Track (hidden from UI)
```
0-30    clear      normal
31-55   straining  feed mentions tiredness; performance variance increases
56-75   fraying    1-on-1 option appears with marker; morale swings amplify
76-90   on_edge    Hana raises concern directly; stream quality drops
91-100  breaking   crisis event fires (character-specific)
```

**Burnout accumulates when:**
- Energy < 30 for 2+ consecutive weeks
- Morale < 40 for 2+ consecutive weeks
- Schedule consistently mismatches traits
- Bad-fit brand deal executed
- Active conflict with another talent
- No check-in for 4+ consecutive weeks

**Burnout decreases when:**
- Multiple consecutive rest-heavy weeks
- Morale sustained high
- Hana 1-on-1 resolves underlying issue
- Player decision prioritizes talent wellbeing over agency interests

### Chemistry System
- Every talent pair has `chemistry` score (0-100)
- Starting values defined per character pair (see Character Roster)
- Chemistry changes per collab based on outcome, shared crises, player decisions
- Effects:
  - >= 70: collabs overperform, morale boost from proximity, cover each other in bad weeks
  - 40-69: neutral baseline
  - < 40: collabs underperform, passive tension, drama seed risk
  - Forced collab at < 40: immediate drama seed

---

## SCHEDULE SYSTEM — WEEKLY RESOLUTION ORDER

```
1. Player assigns slots
2. Trait modifiers applied to energy costs
3. Outputs calculated (subscribers, skill, popularity)
4. Viewership vs expectation → morale modifier
5. Brand deal fit check → morale modifier
6. Chemistry check if collab → chemistry update
7. Burnout accumulation check
8. Event feed entries generated
9. Drama seed checks run
10. Week advances
```

---

## BRAND DEAL PIPELINE

### Deal Object
```js
{
  id: string,
  brandName: string,
  brandTier: 'local' | 'regional' | 'national' | 'premium',
  payout: number,
  fitScore: number,       // 0-100, shown as rough signal not exact number
  executionWeeks: number,
  intrusion: 'low' | 'medium' | 'high',
  riskFlag: boolean,
  status: 'available' | 'in_negotiation' | 'active' | 'complete' | 'failed'
}
```

### Pipeline Stages
```
discovery → pitch → negotiation → execution → aftermath
```

**Discovery sources:**
- Organic inbound (based on talent niche + popularity)
- Agency reputation (higher rep = better quality inbounds)
- Active scouting (player spends agency action, specifies niche)

**Negotiation options:**
- Accept as-is
- Counter payout (risk: brand walks; success depends on agency leverage)
- Counter terms (reduce weeks/intrusion; lower payout)
- Involve Hana (better outcome, costs her bandwidth)

**Execution quality tiers:**
- Excellent: morale high + fit >= 70 → brand relationship+, possible renewal, agency rep+
- Adequate: morale medium or fit 40-69 → deal fulfilled, no relationship growth
- Poor: morale low or fit < 40 → no renewal, talent morale-, brand rep damage
- Failed: morale critical → public incident possible

**Aftermath flags:**
- Misalignment penalty: if Hana flagged bad fit and player overrode → Hana relationship hit
- Audience trust drift: multiple bad-fit deals → audienceTrust stat degrades

---

## DRAMA SYSTEM

### Drama Lifecycle
```
seed → tension → warning → crisis → response → aftermath
```

**Seed conditions (examples):**
- Low chemistry talents collabed repeatedly
- Talent suspects favoritism
- Hana disagrees with decision silently
- Talent passed over for wanted brand deal
- Good performance went uncelebrated

**Warning:** Named feed event with distinct visual treatment. 1-on-1 option appears. 2-3 week intervention window.

**Crisis:** Fires if warning unaddressed. Character-consistent event. 3-4 response options with values-based framing (no good/bad labels).

**Memory tags:** Permanent markers on character state post-crisis.
Examples: `sided_with_brand`, `asked_hana_to_handle`, `let_it_go_public`, `showed_up_for_me`

Memory tags influence future dialogue options and relationship calculations.

---

## HANA SYSTEM

### Hana Stats
```js
{
  relationship: number,     // 0-100
  bandwidth: number,        // 0-100, depleted by agency actions involving her
  internalCompass: {
    mission: number,        // how much she perceives Ren values the mission
    people: number,         // how much she perceives Ren values talent welfare
    friendship: number      // how much she perceives Ren values their bond
  },
  arcState: 'honeymoon' | 'tension' | 'pivot' | 'aligned' | 'drifting' | 'departed',
  memoryLog: string[]       // tracks key decisions she has been watching
}
```

### Hana Relationship Inputs (high weight)
- Consulted vs. informed on major decisions
- Protected talent vs. protected revenue when they conflict
- Shared credit vs. took credit
- Followed up after hard conversations
- How first talent burnout crisis was handled
- Whether she was involved in first major brand negotiation

### Arc State Transitions
- `honeymoon` → `tension`: accumulated tensions (bad crisis handling, overridden disagreements, missed check-ins)
- `tension` → `pivot`: fires at Boutique → Mid-size transition (scripted conversation, content assembled from memoryLog)
- `pivot` → `aligned`: high relationship + right answers in pivot conversation
- `pivot` → `drifting`: low-medium relationship or misaligned answers
- `pivot` → `departed`: very low relationship or repeated ignored warnings

### Hana Bandwidth
- Depleted by: leading negotiations, talent evaluations, mediating staff conflicts, handling assigned crises
- Recovers: 20/week at rest, 10/week normally
- At 0 bandwidth: her reads stop appearing in feed; stops flagging issues proactively

---

## BURNOUT CRISIS EVENTS (per character)

- **Mirae:** Goes silent on socials between streams. Hana reveals panic attacks before sessions.
- **Yuki:** Keeps streaming but something is gone. Hana: "They've decided something."
- **Cato:** Raw on-stream moment about feeling like a content machine. Goes viral wrong.
- **Sable:** Requests a calm meeting. Lays out reasoning. Asks what you think.
- **Kai:** 8-hour stream, increasingly bad decisions, logs off mid-sentence. 3 days silence.

---

## CHARACTER ROSTER

### Hana — Co-Founder
- Home: Taipei | Age: 24 | Not signable talent — partner
- See Hana System above
- Backstory mechanic: if runway < 4 weeks, feed entries shift tone (she watched her mother's agency close at age 12)
- Proximity note: if Ren not in Taipei, starts Regional — most important relationship mediated by distance from day one

### Mirae — @miraegg
- Home: Taipei | Age: 22 | Niche: Tactical gaming
- Subscribers at signing: 8,000
- Traits: `marathon_runner`, `hungry`, `introvert`
- Starting chemistry: Yuki 65, Cato 30, Sable 45, Kai 35
- Backstory mechanic: `self_sufficiency_resistance` active weeks 1-6; schedule changes without explanation = morale penalty; lifts permanently once trust established
- Pivots: First Bad Week (W6-8) → Collab Question (W15-20) → The Offer (late game)
- Loseability: HIGH — leaves cleanly if trust never built; poaching risk late game
- Three versions: GROUNDED / INDEPENDENT / GONE

### Yuki — @yukidraws
- Home: Taipei | Age: 26 | Niche: Art/creative streams
- Subscribers at signing: 3,500
- Traits: `craftsperson`, `night_owl`, `perfectionist`
- Starting chemistry: Mirae 65, Cato 20, Sable 70, Kai 40
- Backstory mechanic: `commercialization_threshold` hidden counter; increments per bad-fit deal; triggers quiet devastating conversation at threshold
- Pivots: Brand Question (early) → Collab That Shouldn't Work (mid) → The Conversation (threshold-triggered)
- Loseability: HIGH — quietest departure, hardest to detect coming
- Visa event: Late Boutique story-driven relocation possibility
- Three versions: FLOURISHING / COMPROMISED / DEPARTED

### Cato — @catorized
- Home: Taipei | Age: 23 | Niche: Variety/entertainment
- Subscribers at signing: 25,000
- Traits: `performer`, `sprinter`, `people_pleaser`
- Starting chemistry: Mirae 30, Yuki 20, Sable 35, Kai 75
- Backstory mechanic: `validation_dependency` tied to subscriber growth; stagnation 3+ weeks = accelerated morale collapse
- Pivots: The Slip (early) → The Serious Idea (mid) → Breaking Point Response (burnout-triggered)
- Loseability: MEDIUM — loud departure if mishandled; sticky if Pivot 01 goes well
- Rootlessness event: low morale + no deep connections → Manila drift
- Three versions: REALIZED / PERFORMING / GONE LOUD

### Sable — @sable.wav
- Home: Taipei | Age: 28 | Niche: ASMR/ambient/lifestyle
- Subscribers at signing: 62,000
- Traits: `coasting`, `night_owl`, `craftsperson`
- Starting chemistry: Mirae 45, Yuki 70, Cato 35, Kai 25
- Backstory mechanic: `community_expectation_event` fires early; early brand/schedule decision tests whether Ren protects her audience
- Pivots: Growth Conversation (early) → Staff Question (Miso-triggered) → Offer to Shape (late)
- Loseability: LOW-MEDIUM — never dramatic; leaves so cleanly you almost miss it
- Never relocates (identity locked to Taipei)
- Three versions: ANCHOR / INDEPENDENT / GONE CLEAN

### Kai — @kaiclutch
- Home: Seoul | Age: 20 | Niche: Competitive gaming
- Subscribers at signing: 15,000 (declining)
- Availability: mid-game unlock
- Traits: `sprinter`, `hungry`, `instinctive`
- Starting chemistry: Mirae 35, Yuki 40, Cato 75, Sable 25
- Backstory mechanic: `the_incident` variable content — investigate before signing (costs agency action) or discover after; incident can resurface at any time
- Pivots: Incident Resurfaces (signing-triggered) → The Slowdown (burnout-triggered) → The Real Conversation (post-breaking-point)
- Loseability: MEDIUM-HIGH — leaves fast without warning if trust never built
- Relocation event: mid-game Taipei move offer if arc positive + trust high
- Three versions: REBUILT / RUNNING / GONE BEFORE THE END

### Talent 06 — Late Unlock
- Undefined by design — emerges through play
- Possible sources: collab partner audience falls for, Hana scouts independently (if relationship high), completely different niche that forces agency identity expansion

---

## STARTING CHEMISTRY MAP
```
          MIRAE  YUKI  CATO  SABLE  KAI
MIRAE      --    65    30    45     35
YUKI       65    --    20    70     40
CATO       30    20    --    35     75
SABLE      45    70    35    --     25
KAI        35    40    75    25     --
```

---

## STAFF ROSTER

### Availability by Stage
```
Early Boutique:   Tomas (Editor) OR Joon (Talent Coach) — one slot, player chooses
Mid Boutique:     Priya (Social), Dana (Finance) unlock
Late Boutique:    Felix (Operations) unlocks
Mid-size:         Miso (Brand Partnerships) unlocks
```

### Staff Object
```js
{
  id: string,
  name: string,
  role: 'editor' | 'social_manager' | 'talent_coach' | 'brand_manager' | 'operations' | 'finance',
  homeCity: string,
  salary: number,           // weekly cost
  cultureFit: number,       // 0-100
  trait: string,            // dominant trait key
  relationship: number,     // 0-100 with Ren
  arcState: string,         // role-specific arc state
  romanticFlags: string[],  // IDs of characters with romantic tension
  loyaltyScore: number,     // 0-100, affects betrayal risk threshold
  proximityState: string
}
```

### Staff Profiles

**Tomas Villanueva** — Editor
- Home: Manila (Remote) | Trait: `collaborative`
- Needs direct talent access; output degrades through intermediaries
- Relationship risk: Tomas x Mirae — quiet, real; causes editing quality gap toward Mirae
- Betrayal risk: LOW

**Priya Subramaniam** — Social Manager
- Home: Singapore (Remote) | Trait: `values_driven`
- Needs clear brand direction from Ren/Hana directly; conflicts with revenue-first Miso
- Relationship risk: Priya x Cato — over-invested community management; personal reactions to audience criticism
- Betrayal arc: LOYALTY TRANSFER — tells talent what they ask to know when values diverge far enough from agency direction

**Park Joon** — Talent Coach
- Home: Seoul | Trait: `empathic`
- Needs protected uninterrupted talent time; impact compounds slowly over weeks
- Back-channel: surfaces talent issues through Hana when trust is high on all sides
- Relationship risk: Joon x Sable — healthy relationship but Joon loses coach role for her
- Betrayal risk: NONE

**Yamamoto Miso** — Brand Partnerships Manager
- Home: Tokyo | Trait: `dealmaker`
- Needs explicit mandate on where identity protection overrides revenue
- Without mandate: optimizes toward revenue, blind to identity costs
- Relationship risk: Miso x Kai — pipeline imbalance, tailors deals around Kai at roster expense
- BETRAYAL ARC: routes info to competing agency if autonomous + undervalued + approached
- Detection: brand partner mentions something they should only know from an insider

**Felix Huang** — Operations Manager
- Home: Taipei | Trait: `process_oriented`
- Excellent in support role; becomes rigid and harmful with authority over creative decisions
- Alliance risk: Felix x Dana — when aligned, excellent; dark version creates unified positions hard to push back on
- QUIET SABOTAGE ARC: passive-obstructive if passed over for expected structural role
- Detection: Dana flags unexplained slowdowns; Joon cannot get schedule answers

**Dana Cho** — Finance Manager
- Home: Taipei | Trait: `pragmatic`
- Most structurally flexible staff member; value depends entirely on Ren actually listening
- Confidence erosion if consistently overridden → eventually considers other clients
- Strongest trust alignment with Hana in the entire cast
- Betrayal risk: NONE — will leave if ignored long enough, not betray

---

## STAFF RELATIONSHIP WEB

### Formation Triggers
- Two characters work closely 4+ consecutive weeks
- Crisis weathered together
- One advocates for another publicly
- Shared late nights (ambient feed events)

### Romantic Pairings and Complications
```
Tomas x Mirae    quiet and real; editing quality gap develops toward Mirae
Joon x Sable     healthy but Joon loses coach role for her
Priya x Cato     destabilizing; over-personal community management
Miso x Kai       pipeline imbalance toward Kai at roster expense
Felix x Dana     excellent operations but unified-front risk
Tomas x Priya    friendly then competitive as agency scales and resources become zero-sum
```

### Detection System
Player does NOT automatically know when relationships form. Requires:
- Feed attention (warm entries between characters = earliest signal)
- Hana relationship >= 50 (she flags things she notices)
- Joon back-channel (surfaces things if trust high on all sides)
- Active 1-on-1 with staff member
- Dana anomaly detection (financial irregularities = last but clearest signal)

---

## ORG STRUCTURE SYSTEM

### Structure Types
```
flat            everyone to Ren/Hana; breaks above 5 staff
functional      organized by function with leads; risk: leads make unauthorized calls
talent_centric  staff assigned to talent pods; risk: factions form, takes sides in drama
hybrid          functional for business + talent-centric for creative; requires Hana relationship >= 50
```

### Org Review — Ren Special Action (once per stage)
- Shows: current structure type, active friction points, information flow gaps, Hana's read if relationship >= 50
- Does NOT show: the right answer
- UI feel: diagrammatic, whiteboard-like — distinct from the rest of the game

### Dysfunction Arc Pattern
```
quiet signal (week 1-2) → louder signal (week 3-4) → crisis event (week 5+)
```

### Example Dysfunction: Miso x Priya
- Condition: Miso has autonomous deal authority; Priya has no formal brand-identity input channel
- Week 1-2: Priya asks about upcoming deal (feed, quiet)
- Week 3-4: Tense conversation after staff meeting (feed, louder)
- Week 5+: Deal executes with conflicting brief → community incident → structural failure exposed

---

## PROXIMITY SYSTEM

### Proximity States
```
local     same city as Ren; full offline menu available when threshold met
regional  2-4 hour flight; visit event required before offline interactions
distant   long-haul; rare expensive visits
remote    no visit context until story event creates meeting
```

### Starting Cities
```
Hana    Taipei      Mirae   Taipei      Yuki    Taipei
Cato    Taipei      Sable   Taipei      Kai     Seoul
Joon    Seoul       Miso    Tokyo
Tomas   Manila (remote)     Priya   Singapore (remote)
Felix   Taipei      Dana    Taipei
```

### Proximity by Ren Starting City
```
Ren in Taipei:
  Local    — Hana, Mirae, Yuki, Cato, Sable, Felix, Dana
  Regional — Miso (Tokyo), Joon (Seoul), Kai (Seoul)
  Distant  — Tomas (Manila), Priya (Singapore)

Ren in Tokyo:
  Local    — Miso
  Regional — all Taipei characters, Joon, Kai
  Distant  — Tomas, Priya

Ren in Seoul:
  Local    — Joon, Kai
  Regional — all Taipei characters, Miso
  Distant  — Tomas, Priya
```

### Story-Driven Relocations
- **Kai → Taipei:** mid-game offer fires if arc positive + trust high; encourage = relocates in 2 weeks
- **Yuki → Tokyo:** Late Boutique visa event; agency sponsors = stays + trust+; self-solves = stays but noted; no action = relocates to Tokyo
- **Cato → Manila:** rootlessness accumulates (low morale + no connections) → drift event fires
- **Hana → elsewhere:** if not in Taipei + relationship degrading → residency offer event fires late game

### Player-Initiated Relocations
- Invite any character: costs 1 agency action + optional relocation support + min bond 30
- Character response depends on arc state and relationship
- Sable never relocates (identity locked to Taipei — hard coded)
- Ren can relocate once per game: 3 weeks reduced efficiency, full proximity map resets

### Offline Interaction Requirements
```
casual_hangout        bond >= 35   local only
work_session          bond >= 25   local or regional (after visit)
offline_collab        bond >= 40   local or regional (after visit)
group_meetup          bond >= 30 (all members); 2+ local, rest can visit
secret_date           bond >= 60 + romantic_flag active;  local only
late_night_debrief    bond >= 45   local only
intervention          bond >= 50   local preferred; flying to regional = extra weight
```

### Visit System
- Cost: 1 agency action + travel budget (scales with distance)
- Minimum relationship: 30
- Duration: 1 week (reduced agency efficiency that week)
- Occasion types: `agency_business` / `check_in` / `event_support` / `emergency`
- Emergency visit: no relationship requirement; the act of showing up is the message
- Visit memory: +5 lasting relationship bonus if visit goes well; feed entry references it weeks later

---

## GROUP SYSTEM

### Formation Requirements
- Chemistry >= 50 between ALL members
- At least 2 collabs completed between all members
- No active drama between any members
- Player proposes group and chooses theme

### Theme Options
```
late_night_variety    casual, parasocial; loyal audience; brand deals: lifestyle, food
competitive_gaming    skill-focused, hype; brand deals: peripherals, energy drinks
life_vlog             personal, slow-burn; brand deals: personal care, travel
creative              art/music; niche passionate audience; brand deals: premium, software
```

### Group Stats
```js
{
  id: string,
  memberIds: string[],
  theme: string,
  groupMorale: number,     // 0-100
  groupChemistry: number,  // 0-100
  audienceSize: number,
  brandValue: number
}
```

### Group Benefits (groupMorale >= 60, active drama seeds <= 1)
- Group streams outperform individual streams for all members
- Audience loyalty higher; individual slip-ups forgiven more easily
- Brand deal value increases (combined reach)
- Members cover each other's off-weeks

### Group Risks
- Internal conflict is public — drama moves faster in groups
- Identity override: `craftsperson` and `hungry` traits resent being "the one from [group]"
- Dissolution: graceful (high morale, no drama, player-managed messaging) vs. catastrophic (forced by crisis, audience picks sides)

---

## WIN / LOSE CONDITIONS

### Hard Lose Conditions
- Cash runway hits 0
- Reputation collapse (cascading unresolved crises, mass talent departure)
- Hana leaves AND reputation < 30

### Endings
```
"The Real Thing"      Hana relationship high at end; 3+ loyal talents; avoided Corporate trigger
"Empire"              Corporate stage reached; epilogue tone depends on Hana status
"Split Custody"       Hana departed and built her own agency; tone depends on how she left
"It Meant Something"  Cash fail BUT Hana relationship >= 60 AND talent loyalty high
"The Cost"            Grew fast, treated people as resources; ex-talents speak publicly
"Still Here"          Never past Boutique; high relationships; 8 years old and still running
```

---

## EVENT FEED

Primary narrative surface. Every system outputs to it.

### Entry Types
```
info        neutral update, no action required
opportunity actionable, time-limited
warning     drama or burnout signal, intervention window open
crisis      immediate response required
memory      post-resolution consequence note
ambient     texture and flavor, no mechanics
```

### Feed Design Rules
- Ambient entries should outnumber mechanical entries
- Burnout signals never use the word "burnout"
- Staff relationship formation: never stated directly, only implied through warmth of entries
- Hana's entries change tone as arc state changes (warmer in honeymoon, more formal in drifting)
- Offline interaction entries are longer and more specific than remote interaction entries
- Group meetup entries are the longest in the game

### Feed Entry Generation Priority
```
1. Crisis events (always surface)
2. Warning events (always surface)
3. Opportunity events (always surface)
4. Hana observations (surface if bandwidth > 0)
5. Staff relationship signals (surface if Hana relationship >= 50 OR player used 1-on-1 that week)
6. Ambient texture (always surface, 1-2 per week)
```

---

## CASH & RUNWAY

```js
{
  cash: number,
  weeklyExpenses: number,    // salaries + operations
  weeklyRevenue: number,     // brand deals + sponsorships
  runway: number             // weeks of cash remaining at current burn rate
}
```

- Runway < 4 weeks: Hana backstory mechanic activates
- Runway = 0: hard lose condition

---

## IMPLEMENTATION NOTES

### State Architecture (suggested)
```
agencyState      — stage, cash, reputation, week number
talentState[]    — all stats, traits, arc progress, memory tags, proximity
staffState[]     — relationship, loyalty, arc state, romantic flags, proximity
hanaState        — relationship, bandwidth, compass scores, arc state, memory log
dramaSeeds[]     — active seeds with lifecycle stage and source
groupState[]     — active groups with stats
dealPipeline[]   — all deals at every pipeline stage
```

### What to Hide From Player UI
- Burnout exact number (show symptoms via feed, not stat bar)
- Chemistry exact scores (show approximate: cold / neutral / warm / strong)
- Hana internal compass scores
- Staff loyalty scores
- Drama seed existence until Warning stage
- Staff romantic flags until feed implies it
- Miso loyalty score specifically (betrayal should be a surprise)

### What to Show in Player UI
- Energy, morale, skill, popularity, subscribers, audienceTrust per talent
- Cash, runway, reputation for agency
- Hana relationship (approximate bands, not exact number)
- Proximity state per character
- Bond per character (approximate bands)
- Active brand deals and pipeline stage
- Current stage and week number

---

## WEEK ADVANCEMENT LOGIC

The full sequence that runs when "End Week" is pressed. Order is strict — later phases cannot affect earlier ones within the same tick.

### Phase 1 — Validate & Lock Schedules
- Confirm every active talent has a primary slot assigned
- If no primary slot → auto-assign `rest`, generate feed note, small rep cost if pattern
- Lock all assignments — no changes after this point
- Flag trait/slot conflicts (e.g. `instinctive` + `extra_prep`)

### Phase 2 — Resolve Energy
```
newEnergy = currentEnergy + slotEnergyDelta + traitModifier + staffModifier
```
Slot deltas: stream_regular −15, stream_push −30, rest +25, event −20, collab −20, training −10, brand_deal −15
Trait modifiers: sprinter on push −5, marathon_runner on push −5, night_owl on morning event −5
Staff modifier: Tomás active + local access → stream/brand_deal slots cost −5 less energy
Cap: floor 0, ceiling 100

### Phase 3 — Resolve Slot Outcomes
**Streams:**
```
basePerformance = skill * 0.4 + popularity * 0.3 + RNG(−10, +10)
performanceDelta = basePerformance − viewerExpectation
subscriberGain = basePerformance * stageMultiplier * slotMultiplier
```
stream_push multiplier: 1.8, RNG widens to (−20, +20)
Stage multipliers: Garage 1.0 / Boutique 1.3 / Mid-size 1.6

**Collabs:**
```
collabPerformance = (talentA.skill + talentB.skill) / 2 + chemistry * 0.3
chemistry < 40: × 0.7 | chemistry > 70: × 1.2
```

**Brand deal execution:**
```
executionScore = morale * 0.5 + fitScore * 0.3 + prepared * 0.2
excellent > 75 | adequate 50–75 | poor 25–50 | failed < 25
```

**Training:**
```
skillGain = 3 + (perfectionist ? +3 : 0) + (joonActive ? +2 : 0)
```

**Events:**
```
popularityGain = 5 + fitScore * 0.1
moraleDelta = fitScore > 60 ? +5 : fitScore < 30 ? −8 : 0
```

### Phase 4 — Resolve Morale
```
newMorale = currentMorale
  + performanceDelta * 0.3
  + scheduleTraitFit         // perfect +5, neutral 0, mismatch −5, hard −10
  + checkinBonus             // hana_1on1 +5, direct +3, no checkin 3+ weeks −3
  + conflictPenalty          // active drama seed −4/week
  + proximityBonus           // offline interaction this week +3
```
people_pleaser modifier: all morale deltas × 1.5
Cap: floor 0, ceiling 100

### Phase 5 — Update Burnout Track (hidden)
```
burnoutDelta = 0
if energy < 30: +8 | if morale < 40: +6
if scheduleTraitMismatch: +4 | if badFitBrandDeal: +5
if activeDramaConflict: +3 | if weeksWithoutCheckin >= 4: +2
if restHeavyWeek: −5 | if morale > 70: −3
if hana1on1ResolvedIssue: −10 | if playerShowedGenuineCare: −8
if personalTimeSlot: −2
newBurnout = clamp(currentBurnout + burnoutDelta, 0, 100)
```
Burnout stages: Clear 0–30, Straining 31–55, Fraying 56–75, On the Edge 76–90, Breaking Point 91–100
Breaking Point → queue crisis event for Phase 6

### Phase 6 — Resolve Events & Drama
Drama seed advancement:
```
seed.weeksSinceLastEscalation++
if >= escalationThreshold: advanceSeedStage()
```
Escalation thresholds: seed→tension 3wk, tension→warning 3wk, warning→crisis 2–3wk

Crisis priority queue (one per week): breaking_point > drama_crisis > staff_conflict > brand_crisis

New seed conditions this week:
- Forced collab chemistry < 40 → seed: `friction`
- Subscriber stagnation 3+ weeks on `hungry` talent → seed: `validation_crisis`
- Bad-fit deal executed → seed: `resentment`
- Major decision without consulting Hana → seed: `silent_divergence`
- Staff relationship threshold met → seed: `entanglement`

### Phase 7 — Update Viewer Expectation
```
newExpectation = (currentExpectation * 0.7) + (weekPerformance * 0.3)
```
Rest weeks: expectation frozen (audience doesn't see rest weeks)

### Phase 8 — Resolve Agency & Finances
```
weeklyRevenue = brandDealPayouts + subscriberRevenueShare + groupBonuses
weeklyExpenses = staffSalaries + operatingCosts + travelCosts
netCash = weeklyRevenue − weeklyExpenses
runway = currentCash / weeklyExpenses
```
Runway < 4 weeks → Hana backstory mechanic fires
Runway = 0 → hard lose condition

Reputation deltas: excellent deal +3, poor deal −5, public breaking point −8, crisis handled well +2, mishandled −4

Hana relationship deltas:
```
consulted on major decision: +3 | informed without consulting: −2
protected talent over revenue: +4 | protected revenue over talent: −3
followed up after hard conversation: +5 | ignored flagged concern: −6
shared credit: +3 | took credit: −2
```

### Phase 9 — Generate Feed & Advance Week
- Generate feed entries (see Feed Generation Rules)
- Increment week counter
- Grant new agency actions
- Reset schedule slots
- Generate new brand deals in pipeline

---

## AGENCY ACTIONS ECONOMY

**Actions per week by stage:**
- Garage: 2 | Boutique: 3 | Mid-size: 4 | Corporate path: 5

**Costs 1 agency action:**
- Targeted talent scouting
- Brand deal pitch (medium/large)
- Deep prospect evaluation
- Staff evaluation before hiring
- Initiating a visit (travel week)
- Investor meeting (mid-size+)
- Direct player check-in with talent

**Free (no action cost):**
- Scheduling
- Browsing deal pipeline
- Reading feed entries
- Assigning secondary slots

**Hana bandwidth (2/week, separate from actions):**
- Hana-led talent 1-on-1
- Hana co-negotiation on brand deal
- Hana running staff evaluation
- Hana mediating staff conflict
- Depleted bandwidth → Hana stops surfacing early warnings that week

---

## STARTING VALUES & ECONOMY NUMBERS

### Full playthrough target: ~120 weeks
Stage durations: Garage 20–30wk / Boutique 30–40wk / Mid-size 30–40wk / Endings from Week 80+

### Agency — Week 1
```
cash: 8,000
weeklyExpenses: 1,200 (base, no staff)
runway: ~6.5 weeks
reputation: 15
culture: 60
agencyActions: 2
stage: "garage"
```

### Ren — Week 1
```
hanaRelationship: 65
reputationAsManager: 10
agencyActionsPerWeek: 2
```

### Hana — Week 1
```
bond: 65
bandwidthPerWeek: 2
arcStage: 1
compassPerceivedValues: { mission: 33, people: 33, friendship: 34 }
```

### Talent Starting Stats

**MIRAE**
```
subscribers: 8,200 | energy: 45 | morale: 58 | skill: 62
popularity: 28 | audienceTrust: 71 | burnoutMeter: 38
viewerExpectation: 420
```

**YUKI**
```
subscribers: 3,500 | energy: 72 | morale: 74 | skill: 71
popularity: 18 | audienceTrust: 84 | burnoutMeter: 12
commercializationCounter: 0 | viewerExpectation: 180
```

**CATO**
```
subscribers: 25,400 | energy: 58 | morale: 66 | skill: 48
popularity: 61 | audienceTrust: 52 | burnoutMeter: 24
validationDependency: 0 | viewerExpectation: 1,800
```

**SABLE**
```
subscribers: 62,000 | energy: 81 | morale: 70 | skill: 76
popularity: 44 | audienceTrust: 91 | burnoutMeter: 8
viewerExpectation: 3,200
```

**KAI** (available ~Week 25–30)
```
subscribers: 14,800 | energy: 71 | morale: 44 | skill: 69
popularity: 38 | audienceTrust: 43 | burnoutMeter: 31
viewerExpectation: 980
```

### Subscriber Growth Rates (weekly, before multipliers)

| Slot | Garage | Boutique | Mid-size |
|---|---|---|---|
| stream_regular | 80–140 | 200–380 | 500–900 |
| stream_push | 180–320 | 450–780 | 1,100–2,000 |
| collab (per talent) | 60–180 | 150–420 | 400–1,000 |
| event | 40–120 | 100–280 | 250–600 |
| brand_deal | 20–60 | 50–140 | 120–300 |
| training / rest | 0 | 0 | 0 |

**Performance multipliers:**
```
performanceDelta > 20: ×1.4 | 5–20: ×1.1 | −5 to 5: ×1.0
−5 to −20: ×0.7 | < −20: ×0.4
```

**Trait multipliers:**
- hungry on good week: ×1.1
- marathon_runner on 4th consecutive regular: ×1.15
- coasting on push: ×0.8
- sprinter on push after rest: ×1.2

### Revenue Model

**Subscriber revenue per week:**
```
talentWeeklyRevenue = subscribers * revenuePerSubscriber * platformRate
agencyRevenueCut = talentWeeklyRevenue * agencyShareRate
```
Revenue per subscriber: Garage 0.008 / Boutique 0.010 / Mid-size 0.012
Agency share: standard 20% / talent-favorable 15% / agency-favorable 25%

**Week 1 combined subscriber revenue (all 4 talents): ~$160/week**
vs. expenses of $1,200/week → brand deals are required to survive

### Brand Deal Payouts

| Tier | Payout | Execution Weeks | Available From |
|---|---|---|---|
| Local | $400–$800 | 1 | Garage |
| Regional | $1,200–$2,400 | 1–2 | Garage |
| National | $3,500–$7,000 | 2–3 | Boutique |
| Premium | $8,000–$20,000 | 2–4 | Mid-size |

### Operating Costs by Stage
```
Garage: $1,200/week | Boutique: $2,400/week
Mid-size: $4,800/week | Corporate: $9,600/week
```

### Staff Salaries (weekly)
```
Tomás (Editor): $600 | Priya (Social): $700 | Joon (Coach): $750
Miso (Brand): $900 | Felix (Operations): $650 | Dana (Finance): $600
```

### Subscriber Milestones
```
5K: feed celebration | 10K: counts toward Boutique unlock
25K: mid-tier brand deals | 50K: National deals available
100K: counts toward Mid-size unlock | 250K: Premium deals
500K: counts toward Corporate unlock | 1M: Empire ending territory
```

### Stat Floors & Ceilings
All stats: floor 0, ceiling 100
Energy recovers weekly | Morale moves slowly | Skill only increases
Audience Trust hard to recover once lost | Burnout hidden from UI

### Starting City Difficulty
- Taipei: local access to 4 talents + 3 staff. Warmest start.
- Tokyo: most talent at Regional. Hana is a call away. Harder.
- Seoul: local to Joon/Kai (unavailable at start). Quietest opening. Hardest.

---

## UI/UX — CORE SCREENS

### Design Language
- Background: `#0d0d12` | Text: `#e8e4d9`
- Character accent colors: Hana #f472b6, Mirae #60a5fa, Yuki #a78bfa, Cato #fb923c, Sable #34d399, Kai #facc15
- Monospace for data | Serif for narrative | Sans for UI labels
- Target: 1280px+ desktop first

### Shell Architecture
```
┌─────────────────────────────────────────────────────────┐
│  TOPBAR — Week, cash, runway, stage, End Week button    │
├──────────┬──────────────────────────────┬───────────────┤
│ SIDEBAR  │      MAIN PANEL             │  EVENT FEED   │
│ 200px    │      ~680px flexible        │  320px fixed  │
└──────────┴──────────────────────────────┴───────────────┘
```
Modals expand over main panel only. Feed always visible. Topbar always visible.

### Topbar
- Left: agency name + stage badge
- Center: WEEK [N], season flavor line, ambient tone line
- Right: cash (amber tint if runway < 4wk), runway weeks, agency actions (◆◆◇), Hana bandwidth (●●), END WEEK button

### Sidebar
- Navigation: ROSTER / SCHEDULE / DEALS / AGENCY / SCOUTING
- Bottom pulse: total subscribers + delta, reputation bar, Hana status word

### View A — Roster
Talent cards in 2-column grid.
Card shows: portrait, name, handle, niche, city, energy/morale/skill bars (hover for numbers), subscriber count + delta, assigned slots, check-in button.
Hidden on card: burnout meter, audience trust, traits, chemistry.
Burnout bleeds into card texture — worn appearance at Straining, indicator at Fraying, portrait expression shifts, pulse at Breaking Point.

### View B — Schedule
Table: talent rows × primary/secondary/projected-energy columns.
Energy column shows current → projected, amber if < 30, red if 0.
Trait conflict indicator: subtle ⚠ with tooltip.
Bottom bar: projected subscriber movement, cash change, Hana bandwidth used.

### View C — Deals
Kanban: AVAILABLE / IN PITCH / NEGOTIATION / EXECUTING columns.
Deal card: brand name, tier, payout, fit bar (label not number), risk flag, suggested talent, pursue/pass actions.
Executing deals show progress + talent morale indicator.

### View D — Agency (3 sub-tabs)
D1 Staff: hired staff cards + empty role slots as dashed outlines.
D2 Org Structure: diagrammatic whiteboard view. Friction = red lines, gaps = dotted lines. Hana annotation if bond ≥ 50. Cooler color temperature, graph paper texture.
D3 Finances: income vs. expenses breakdown, 8-week runway projection, Dana notes if hired.

### View E — Scouting
Active prospect cards (known vs. hidden fields). Scouting action buttons: Browse (free), Targeted Search (1 action), Referral check.

### Event Feed
Vertical scroll, newest first, infinite history.
Entry types: neutral (dot), signal (pulse + warm bg), warning (amber border + follow-up button), crisis (modal interruption), memory (italic, dim), hana (distinct card shape).
Filter row: by character or entry type.

### Modal Types
- Character Profile: full stats, traits, chemistry web, memory tags, arc timeline
- Decision Modal: portrait + situation + 3–4 option cards + Hana note if bond ≥ 50
- 1-on-1 Modal: conversation interface, two columns
- Hiring Modal: candidate profile, evaluate vs. offer actions
- Negotiation Modal: accept / counter payout / counter terms + outcome word (Likely/Uncertain/Risky)
- Org Review Modal: full-screen whiteboard diagram

### Transitions
No hard cuts. End Week has deliberate pause before feed entries appear. Crisis modals arrive slowly — fade then settle. Feed updates feel like messages arriving.

---

## EVENT FEED GENERATION RULES

### Entry Priority Stack (runs each week after all phases complete)
```
1. CRISIS entries        — always, trigger modal
2. WARNING entries       — always, follow-up button
3. HANA entry            — always, one per week
4. SIGNAL entries        — threshold crossings
5. TALENT entries        — 1 per active talent
6. STAFF entries         — 0–2 per week
7. AGENCY entries        — 0–1 per week
8. AMBIENT entries       — 0–2 per week, texture only
```
Max entries/week: 8 | Min: 3 | Crisis weeks may exceed 8

### Hana Entry Rules
- Always present, one per week
- Selected from state pool matching her current arc stage
- Never purely informational — always has texture, a detail, her presence
- Gets more words than other entries (3–5 sentences allowed)
- In low-relationship states: entries get sadder, not emptier

### Talent Entry Rules
Template selection: slot type + outcome quality + burnout stage + relationship state
Key template pools:
- stream_regular overperformed: warmth, specific detail, subscriber number
- stream_push underperformed: quiet aftermath, flat number acknowledgment
- rest + check_in: small human moment, warmth
- rest + no_check_in: absence noted, no drama
- collab high chemistry: physical-space detail, audience feeling it
- collab low chemistry: professional, polite, forgettable
- brand_deal poor + bad fit: "got through it" language

### Burnout Signal Entries (blend into talent entries, not separate)
- Straining: "seemed off", "different kind of tired"
- Fraying: sleep mentions, 1-on-1 follow-up button appears
- On the Edge: Hana raises it directly, distinct visual
- Breaking Point: held-breath quality, then modal fires

### Signal Entries (threshold crossings)
- Subscriber milestone: specific, slightly delayed (she noticed the next morning)
- Chemistry shift: "something changed", named without being explained
- Hana bandwidth depleted: matter-of-fact, she hasn't said anything
- Runway warning: you ran the numbers twice, Hana's tone shifts this week and next
- Staff relationship forming: observed proximity, you've noticed

### Staff Entries (0–2/week, only when notable)
- Good performance: specific output, talent response
- Structural friction: thing that happened, one person's muted reaction
- Relationship forming: a shared space, a moment in the agency chat
- Betrayal signal: a fact, probably nothing, more calls than usual

### Agency Entries (0–1/week)
- Reputation shift: someone mentioned the agency, nothing loud but it bleeds
- Stage approaching: the agency feels different, more real
- Investor contact: someone reached out, you haven't replied

### Ambient Entries (0–2/week, no gameplay significance)
Weather, group chat quiet, working late, someone posted something personal, Hana fixed a bug without mentioning it, the coffee place is closed.

### Tone Shift Rules

**Morale modifier:**
```
80–100: warm, specific, present
60–79: neutral, observational
40–59: muted, factual
20–39: sparse, careful
0–19: absent quality, something missing
```

**Relationship modifier:**
- High bond (70+): named details, small observations, intimacy
- Medium (40–69): accurate, less texture
- Low (0–39): reading at distance, managing someone you don't know

**Burnout bleeds into every entry** — not explicit, but through what the entry notices.

### Feed Voice Rules
- Never clinical. Stats become people.
- Specific over generic. Details carry emotion.
- Earned emotion — never tell the player how to feel.
- Short — 2–4 sentences. One-sentence entries can be the most powerful.
- Present tense for events, past tense for observations.
- The player is "you." Not Ren. Just you.
- Hana gets more words. She earns them.
- Ambient entries can be beautiful. Let them.

### Feed History
All entries stored permanently. Scrolling back to Week 3 from Week 60 should feel like reading old messages. Late-game ambient entry can quietly reference this.

---

---

## OPENING SCENE & ONBOARDING

### Opening Sequence (pre-gameplay, unskippable first time)

SCREEN 1 — Black. Two lines fade in slowly:
> *Three years ago, Hana called you at 2am and said: let's just do it.*
> *You said yes before she finished the sentence.*
Both fade out.

SCREEN 2 — Character Setup. Minimal, one lamp warmth, no music.
- Name field: free text, cursor blinks, no placeholder
- City: Taipei / Tokyo / Seoul with one flavor line each
  - Taipei: "Where most of them already are."
  - Tokyo: "A city that keeps its distance. So do you, sometimes."
  - Seoul: "Colder than you expected. You're still figuring that out."
- Agency name: free text, skippable (defaults to placeholder, changeable later)

SCREEN 3 — Establishing line using player's name + city:
> *[Name]. Taipei. Week 1. The agency has two clients, one shared calendar, and exactly 6.5 weeks of runway. Hana already has opinions about all three.*

Then: full UI loads. No fanfare. No tutorial popup. Just the game.

### Week 1 Feed Entries (pre-populated)

Entry 1 — HANA (type: hana):
*"Hana's first message came in at 7am. 'I've been thinking about the schedule. Also I found a good ramen place near the office that I want you to try before you have opinions about anything else today. Also good morning.' You've known her for eight years. This is exactly what eight years sounds like."*

Entry 2 — MIRAE (type: signal):
*"Mirae sent her streaming schedule for the week at midnight. No message. Just the schedule, formatted correctly, with a note at the bottom: 'let me know if you want changes.' She's been doing everything herself for two years. It's going to take her a while to believe you're actually here."*

Entry 3 — YUKI (type: neutral):
*"Yuki posted a new piece at 11pm. A sketch — something with light in it. 180 people watched them draw it live. They didn't announce it in the agency chat. They just made it."*

Entry 4 — AGENCY (type: signal):
*"The runway is 6.5 weeks. That's the number that matters this week. Hana knows. She hasn't said anything yet, which means she's decided to trust you to handle it."*

### Talent Stagger by City
- Taipei: Mirae + Yuki Week 1, Cato Week 2, Sable Week 3
- Tokyo/Seoul: One talent Week 1, others trickle over Weeks 2–3

### Gentle Failure States
- No schedule assigned → auto-rest + mild feed note
- Brand deal ignored → expires Week 2, feed note, no rep damage
- Runway < 4 weeks → Hana tone shifts, no alarm
- Mirae pushed in Week 1 → burnout ticks invisibly, symptom feed entry Week 3–4

### Onboarding Philosophy
No tutorials. Three teaching channels:
1. Feed tells you what happened (outcomes as narrative)
2. UI shows what's possible (live projected energy, trait conflict ⚠)
3. Hana tells you what she thinks (as a person, not a guide)

### First Five Weeks Arc
- Week 1: Meet Mirae + Yuki, first schedule, cash visible
- Week 2: Cato joins (Taipei) — louder energy, contrast immediate
- Week 3: Sable joins (Taipei) — quietest entry yet
- Week 4–5: First real cash pressure, first drama seed possible
- Week 6–8: Mirae Pivot 01 fires. Tutorial period over. Game begins.

---

## DIALOGUE SYSTEM

### Core Philosophy
Dialogue opens when something requires direct human contact — pivot moments, crisis responses, 1-on-1s that surface something real. Rarity makes it weight-bearing. Most communication happens through the feed.

Three rules:
1. Not every message needs a response option
2. Choices are labeled by action, never by moral weight
3. Conversations are remembered — Week 6 can surface in Week 40

### Visual Design
Chat interface modal. Opens over main panel. Feed visible behind, dimmed. Topbar remains.

- Character messages: left-aligned, soft bubble, accent color tint
- Ren has no bubble — responses trigger next beat, not a sent message
- Timestamps: small, right-aligned, feel real
- Response options: card group at bottom, visually distinct from conversation
- Continue button: single centered button, lowercase "continue", no options needed

### Four Conversation Types

TYPE 1 — CHECK-IN
Casual, low-stakes, relationship maintenance. 3–5 messages, 0–1 response moments.
Triggered by: hana_1on1 secondary slot, direct check-in agency action.
Ends with: relationship tick, sometimes hidden state reveal.

TYPE 2 — 1-ON-1 (SURFACING)
Deeper. 6–10 messages, 2–3 response moments.
Triggered by: feed warning follow-up, Hana flagging something.
Ends with: hidden stat revealed, something said or unsaid that persists.

TYPE 3 — PIVOT CONVERSATION
Named, weighted, unrepeatable. 10–20 messages, 3–4 response moments.
Fully written, fully branching. Some choices close options permanently.
Ends with: arc state change, memory tags written.

TYPE 4 — CRISIS RESPONSE
High stakes. 2–4 setup messages, one major decision.
Visual difference: darker tint, no timestamps, larger response cards.
Each option has one line of subtext showing what it IS, not outcome prediction.
Ends immediately after choice — consequence plays out in feed.

### Response Option Rules
- Framed as actions/intents, never as Ren's scripted words
  ✅ "Ask her what she thinks we should do."
  ✅ "Admit you should have talked to him first."
  ❌ "'Hey, I hear you. Let's figure this out together.'"
- No good/bad labels — options reflect different values
- Continue button for moments that don't need a choice

### Conversation Memory
- Full conversation log stored per character
- Future conversations can reference specific past exchanges by content
- Character profile timeline shows key conversations — readable history
- Hana can cite specific past promises if you broke them

### Ren Has No Voice
Ren speaks through action framing only. The character voice belongs entirely to the people Ren is talking to. This keeps the player projected into Ren rather than watching them perform.

### Proximity Effect on Dialogue
Remote: timestamps visible, sense of asynchrony
Local (after offline interaction): first message has no timestamp, feels more like talking than texting, physical space occasionally referenced

### Character Message Voices

HANA: Multiple short messages in sequence. Thinks out loud. Lowercase casual. Periods are intentional — serious messages have them, casual ones don't.

MIRAE: Complete sentences, always. Minimal. Long gaps between messages. Questions are genuine, never small talk.

YUKI: Short, precise. Starts with context before the point. Uses "I think" as accuracy, not hedge.

CATO: Fast. Fragments. Multiple quick messages when excited/nervous. Single message, no punctuation, when processing something hard.

SABLE: One message, exactly what she means. Never multiple. No typos. Precise but not cold.

KAI: Short. "hey" as a full message. Long gap. Then something real. Communicates in the gaps.

STAFF: More professional. Joon warmest. Miso businesslike. Felix like a PM. Dana makes hard numbers sound neutral. Priya careful with words.

### Hana's Arc in Dialogue Texture
Arc 1 (warm): She initiates, asks follow-ups, leaves space, uses your name
Arc 2 (forming): Still initiates, but messages contain unasked questions
Drifted: Responds when you initiate. Complete messages. No loose ends. No follow-up bait.
Post-departure: One optional late-game conversation. No gameplay consequence. Just what's left.

### Unresponded Messages
Some conversations only have a continue button. Sable's early messages. Quiet moments.
This is also how Hana's drift manifests — she stops leaving space for response. The player has fewer opportunities to say anything because she's stopped inviting it.

---

## OPENING SCENE & ONBOARDING

### Opening Sequence (pre-gameplay, unskippable first run)

SCREEN 1 — Black. Two lines fade in slowly:
"Three years ago, Hana called you at 2am and said: let's just do it."
"You said yes before she finished the sentence."
Both fade out. Black again.

SCREEN 2 — Character Setup. Minimal, one lamp warmth, ambient room tone.
- Name field: free text, cursor blinks until player types
- City: Taipei / Tokyo / Seoul with one flavor line each
  - Taipei: "Where most of them already are."
  - Tokyo: "A city that keeps its distance. So do you, sometimes."
  - Seoul: "Colder than you expected. You're still figuring that out."
- Agency name: free text, skippable (defaults to placeholder, changeable later)

SCREEN 3 — Establishing line. Player name + city + specific flavor:
Taipei: "[Name]. Taipei. Week 1. The agency has two clients, one shared calendar, and exactly 6.5 weeks of runway. Hana already has opinions about all three."
Tokyo: "[Name]. Tokyo. Week 1. The agency has two clients, a shared calendar, and a Hana who is three time zones closer than you."
Seoul: "[Name]. Seoul. Week 1. The agency has two clients, a shared calendar, and a runway that won't last. Hana is in Taipei. The distance is mostly fine. Mostly."

Then: full UI loads. No fanfare. No tutorial popup. Just the game.

### Week 1 Feed Entries (pre-populated)

ENTRY 1 — Hana (hana type):
"Hana's first message came in at 7am. 'I've been thinking about the schedule. Also I found a good ramen place near the office. Also good morning.' You've known her for eight years. This is exactly what eight years sounds like."

ENTRY 2 — Mirae (signal type):
"Mirae sent her streaming schedule at midnight. No message. Just the schedule, formatted correctly, with a note: 'let me know if you want changes.' She's been doing everything herself for two years. It's going to take her a while to believe you're actually here."

ENTRY 3 — Yuki (neutral type):
"Yuki posted a new piece at 11pm. A sketch — something with light in it. 180 people watched them draw it live. They didn't announce it in the agency chat. They just made it."

ENTRY 4 — Agency (signal type):
"The runway is 6.5 weeks. That's the number that matters this week. Hana knows. She hasn't said anything yet, which means she's decided to trust you to handle it."

### Roster Stagger by City
Taipei: Mirae + Yuki visible Week 1. Cato joins Week 2. Sable joins Week 3.
Tokyo/Seoul: One talent visible Week 1. Others trickle in over first 3 weeks.

### Onboarding Philosophy — No Tutorials
Three teaching channels, none are popups:
1. The feed tells you what happened (outcomes described as events, not stat changes)
2. The UI shows what's possible (live energy preview, slot labels, trait conflict ⚠)
3. Hana tells you what she thinks (not as a guide — as a person with opinions)

### Gentle Failure States (early game)
- No schedule assigned: auto-rest fires, feed entry explains gently, no hard penalty
- Brand deal expires: feed notes the miss, no reputation damage, lesson learned
- Runway drops < 4 weeks: Hana's tone shifts (not an alarm — a texture change)
- Push stream Week 1: works fine, burnout ticks invisibly, pays off later as context

### First Five Weeks Shape
Week 1: Meet Mirae + Yuki. First schedule. Hana warm. Cash visible.
Week 2: Cato joins. Louder energy. Contrast is immediate.
Week 3: Sable joins. Quietest entry yet. Three distinct energies now in feed.
Week 4–5: First real cash pressure. First drama seed may be planted.
Week 6–8: Mirae Pivot 01 fires. Tutorial period over. Game begins.

### What Player Knows by Week 5 (without being told)
- Hana has opinions and a memory
- Mirae works alone and needs trust
- Yuki creates on their own terms
- Cato is performing and it takes energy
- Cash is real and time-limited
- The schedule matters more than it looks
- The feed is worth reading

---

## DIALOGUE SYSTEM

### Core Philosophy
Dialogue opens when something requires direct human contact — a pivot, a crisis, a 1-on-1 that surfaces something real. Rarity makes it weight-bearing.
Rules: Not every message needs a response. Choices are never labeled good/bad — labeled by what you do. Conversations remember.

### Visual Design — Chat Interface
Modal over main panel. Feed visible behind, dimmed. Topbar remains.
- Character messages: left-aligned, soft bubble, accent color tint
- No Ren bubble — responses trigger next beat, Ren has no scripted voice
- Timestamps: small, right-aligned, feel real
- Response options: card group at bottom, distinct from conversation
- Continue button: lowercase "continue", centered, understated
- Character header: small portrait (32×32), name, arc-state color temperature

### Four Conversation Types

TYPE 1 — CHECK-IN
Casual, relationship maintenance. Triggered by hana_1on1 slot or agency action.
Structure: 3–5 messages, 0–1 response moments, ends with relationship tick + possible hidden state reveal.

TYPE 2 — 1-ON-1 (SURFACING)
Deeper. Triggered by feed warning Follow Up or Hana flag.
Structure: 6–10 messages, 2–3 response moments, surfaces hidden stat, ends with permanent state change.

TYPE 3 — PIVOT CONVERSATION
Named, weighted, unrepeatable. 2–3 per character.
Structure: 10–20 messages, 3–4 response moments, ends with arc version state change. Some choices close options permanently.

TYPE 4 — CRISIS RESPONSE
High stakes. Fires at breaking point or major drama crisis.
Structure: 2–4 setup messages, one major decision (3–4 options with cost subtext), immediate consequence, aftermath queued for feed.
Visual: darker tint, no timestamps, response options larger with subtext framing (not outcome prediction).

### Response Option Rules
- Labeled by action/intent, never by scripted dialogue
- ✅ "Ask her what she thinks we should do."
- ✅ "Admit you should have talked to him first."
- ❌ "'Hey, I hear you. Let's figure this out together.'"
- Crisis options include one subtext line showing what the choice costs/signals — not outcome prediction, just framing

### Conversation Memory
Every dialogue writes to conversation history log. Future conversations can reference past ones explicitly. Character Profile timeline shows key conversations abbreviated. Player can read back to Week 6 from Week 50.

The gap mechanic: if Ren stated an intent and acted against it, both are logged. Characters notice the gap. Hana notices most.

### Character Message Voices (fixed, don't change with arc state)
HANA: Multiple short messages in sequence. Thinks out loud. Lowercase casual. Period = serious. No period = casual.
MIRAE: Complete sentences always. Minimal. Long gaps. Questions are genuine.
YUKI: Short and precise. Context before point. "I think" used for accuracy not hedging.
CATO: Fast fragments in sequence when excited. Single message no punctuation when processing.
SABLE: One message, complete, no typos. Precise but not cold.
KAI: "hey" as full message. Long gap. Then something real.
STAFF: Joon warmest, Miso most businesslike, Felix like a PM, Dana makes hard numbers neutral, Priya thoughtful and careful.

### Unresponded Messages
Continue button only — no choice. Sable's early messages. Quiet moments.
Hana's drift manifest: she stops leaving space for response. Player has fewer opportunities to say anything because she's stopped inviting it.

### Offline vs. Online Texture
Remote: timestamps visible, sense of asynchrony
Local/after offline: no timestamps on first message, feels more like talking, may reference physical space ("I'm still thinking about what you said at the ramen place")

### Hana Arc in Dialogue
Arc 1 (warm): She initiates, asks follow-ups, leaves space, uses your name
Arc 2 (forming): Still initiates, messages contain unasked questions
Drifted: Responds when you initiate. Complete. No loose ends. No follow-up bait.
Post-departure: One optional late-game conversation. No gameplay consequence. Just what's left.

---

*Last updated: Week Advancement, Agency Actions, Starting Values, UI/UX, Event Feed, Opening Scene, Dialogue System.*
*Next: Investor System (#8), Ren romance arc, Talent 06, staff hiring UX, save system.*
*Fetch this document at the start of any build session.*
