# Streaming Talent Agency — Game Design Reference
> Browser-based React game. Turn-based (weekly). Pan-Asian streamer culture setting.
> This document is the single source of truth for mechanics, characters, and systems.

---

## QUICK INDEX
- [Core Principles](#core-principles)
- [Player Setup](#player-setup)
- [Characters — Talent](#characters--talent)
- [Characters — Staff](#characters--staff)
- [S01 Schedule System](#s01-schedule-system)
- [S02 Brand Deal Pipeline](#s02-brand-deal-pipeline)
- [S03 Personality & Chemistry](#s03-personality--chemistry)
- [S04 Burnout & Breaking Points](#s04-burnout--breaking-points)
- [S05 Drama System](#s05-drama-system)
- [S06 Group System](#s06-group-system)
- [S07 Stage Progression](#s07-stage-progression)
- [S08 Hana's Arc](#s08-hanas-arc)
- [S09 Scouting System](#s09-scouting-system)
- [S10 Staff & Org Structure](#s10-staff--org-structure)
- [S11 Endings](#s11-endings)
- [S12 Proximity & Offline](#s12-proximity--offline)

---

## CORE PRINCIPLES

1. **Drama = earned consequences, not random events.** Every crisis traces back to a player decision.
2. **Systems create emergent stories.** Not scripted events — interactions between mechanics.
3. **Org structure IS gameplay.** Bad structure creates dysfunction even with great people.
4. **Hana relationship is the emotional spine.** Almost every major decision should affect it.
5. **Lore and mechanics are inseparable.** Backstory creates hidden mechanical variables.
6. **Distance is emotional texture.** Remote-first world; physical presence is earned and memorable.

---

## PLAYER SETUP

Fires before Week 1. Minimal screen — dark, late-night aesthetic.

```
Fields:
  name: string (free text, no default, cursor blinks)
  city: enum ['Taipei', 'Tokyo', 'Seoul']
    Taipei → "Where most of them already are. Familiar ground."
    Tokyo  → "A city that keeps its distance. So do you, sometimes."
    Seoul  → "Colder than you expected. You're still figuring out if that's the city or you."
```

No other customization. Personality emerges through decisions.

---

## CHARACTERS — TALENT

### Hana — Co-Founder & Partner
```
role: co_founder
age: 24
home_city: Taipei
traits: [reads_people, negotiator, culture_keeper]

internal_compass:
  - mission: building something real
  - people: protecting talent, keeping culture human
  - you: the friendship, what you started this as
  (she will sacrifice one for others based on what she sees Ren prioritize)

hidden_mechanics:
  agency_mother_lost:
    trigger: cash_runway < 4 weeks
    effect: feed entries shift tone (urgent, less collaborative)
    note: never stated explicitly — felt in texture

relationship_inputs (high weight):
  - consult vs inform on major decisions
  - protect talent vs revenue when they conflict
  - take credit vs share it
  - follow up after hard conversations
  - how first burnout crisis was handled

arc_paths:
  A: stays_and_aligns   → senior partner, own agency actions, emotional anchor
  B: stays_but_drifts   → present, disconnected, feed entries get shorter/formal
  C: leaves_agency      → external character, resurfaces once in context of how she left

arc_beats:
  beat_1: "We're Doing This" (early game) — warm, backs every call, honeymoon
  beat_2: "I've Been Meaning to Talk to You" (mid game) — triggered by accumulated tensions
  beat_3: "Who Are We Now?" (Boutique→Mid-size) — scripted conversation, content from history
```

---

### Mirae — The Grinder
```
handle: @miraegg
age: 22
home_city: Taipei
niche: gaming (tactical shooters, ranked climb)
subs_at_signing: 8000

traits: [Marathon Runner, Hungry, Introvert]

hidden_mechanics:
  solo_operator_resistance:
    active: first 4-6 weeks after signing
    trigger: schedule changes without check-in explanation
    effect: small morale penalty per unexplained change
    resolves: permanently lifts once trust threshold met
    if_never_addressed: feeds burnout track

pivots:
  P1 "The First Bad Week" (week 6-8):
    A: check_in_directly → trust+, opens up, lets-people-in version begins
    B: adjust_schedule_silently → self-sufficiency resistance hardens
    C: assign_push_week → learns Ren responds to struggle with pressure

  P2 "The Collab Question" (week 15-20):
    A: let_her_decide → autonomy stat forms, easier to manage long-term
    B: gently_push → survives discomfort, but if repeated, boundaries feel negotiable
    C: schedule_without_asking → self-sufficiency wall rebuilds

  P3 "The Offer" (late game):
    high_trust: tells Ren before deciding, asks what you think
    medium_trust: sits on it 2 weeks, Hana finds out first
    low_trust: already decided when Ren finds out, cold and professional

versions:
  GROUNDED: stayed, learned to trust, agency anchor, says hard things to your face
  INDEPENDENT: on roster, excellent work, arm's length, professional and slightly sad
  GONE: left — texture of departure depends on how it happened
```

---

### Yuki — The Craftsperson
```
handle: @yukidraws
age: 26
home_city: Taipei
niche: creative (live illustration, game soundtrack covers, game dev diaries)
subs_at_signing: 3500
note: highest audience_trust stat on roster at signing

traits: [Craftsperson, Night Owl, Perfectionist]

hidden_mechanics:
  commercialization_threshold:
    counter: increments with each bad-fit brand deal
    at_threshold: Yuki requests quiet meeting
    meeting_content: "What do you think I'm for?"
    if_identity_protected: moment of connection
    if_counter_maxed: they've already decided — meeting is them being honest

  visa_event (late Boutique):
    options:
      agency_sponsors: formalizes relationship, Yuki moved
      yuki_figures_it_out: stays, lack of support noted
      yuki_relocates_tokyo: proximity changes

pivots:
  P1 "The Brand Question":
    A: pass_explain → creative relationship opens, they share more about the work
    B: take_present_positively → counter ticks, warning question fired
    C: take_without_discussing → counter ticks faster, feels like a product

  P2 "The Collab That Shouldn't Work" (with Mirae):
    frame_as_growth: goes through motions, fine
    frame_as_experiment: chemistry +15, friendship forms offscreen
    no_framing: awkward, notes being treated as pieces to move

  P3 "The Conversation" (commercialization threshold):
    protected_history: conversation about what to build next — best relationship moment
    inconsistent_history: gap surfaces, repairable but slow
    counter_maxed: already decided, meeting is farewell with respect

versions:
  FLOURISHING: artistic home, modest numbers, deeply engaged audience, credits agency quietly
  COMPROMISED: executing, subtly less alive, feed entries about streams get shorter
  DEPARTED: left clean, sent Hana a handwritten note, Hana keeps it
```

---

### Cato — The Performer
```
handle: @catorized
age: 23
home_city: Taipei
niche: variety/entertainment (reaction, challenges, parasocial late-night talk)
subs_at_signing: 25000

traits: [Performer, Sprinter, People Pleaser]

hidden_mechanics:
  validation_dependency:
    trigger: subscriber growth stalled 3+ consecutive weeks
    effect: morale drops faster than numbers suggest
    reason: numbers = proof leaving performing arts was right
    never_stated: in feed texture and check-in tone

  performing_arts_deflect:
    trigger: direct question about the program
    response: deflects with a joke
    hana: notices he deflects
    player_can: push or drop it

pivots:
  P1 "The Slip" (early-mid game):
    defend_then_investigate: loyalty anchors, trusts Ren sees him as person
    investigate_then_respond: understands but warmth cools
    manage_optics_first: sees the distance, People Pleaser activates, burnout spiral begins

  P2 "The Serious Idea" (documentary pitch):
    back_it: stunned, makes it, underperforms metrics/overperforms everything else
    suggest_current_format: drops it, brings up once more tentatively, then gone
    back_with_caveats: hears caveats more than backing, doesn't pitch personal again

  P3 "The Breaking Point Response":
    call_him_first: picks up immediately, most human conversation in game
    team_meeting_first: finds out through Hana, not surprised, tired
    statement_first: watches on phone, professional after 4 days, something gone

versions:
  REALIZED: performer and person in same body, would follow Ren anywhere
  PERFORMING: hitting numbers, visibly emptier, the drift is in the eyes
  GONE_LOUD: left honestly, became a story, Hana still thinks about it
```

---

### Sable — The Enigma
```
handle: @sable.wav
age: 28
home_city: Taipei
niche: ASMR / ambient / lo-fi lifestyle
subs_at_signing: 62000
note: reputation in community — signing her is a statement
note: NEVER relocates — Taipei is load-bearing to her identity

traits: [Coasting, Night Owl, Craftsperson]

hidden_mechanics:
  community_expectation_event:
    fires: early after signing
    trigger: something scheduled/considered puts audience at risk of feeling she's changed
    protect: trust accelerates dramatically
    dont_protect: subscriber churn begins, quiet reassessment

  late_game_advisor_offer:
    condition: pivot 3 accepted
    effect: becomes unofficial advisor, her read joins Hana's as early warning system

pivots:
  P1 "The Growth Conversation":
    ask_first: fully formed opinion, explains why — informs management for rest of game
    push_as_opportunity: asks for a week, comes back with conditions
    done_deal: "I'd like to have been part of this decision." significant trust hit

  P2 "The Staff Question" (Miso's deal push):
    correct_miso_before_sable: never knows, Hana notes you caught it
    let_miso_approach: declines all, messages Ren "do we want the same things?"
    back_miso: executes 2, declines rest, requests meeting anyway

  P3 "The Offer to Shape":
    accept: agency's institutional memory, late-game prescience
    decline_politely: stays as talent, offer closed
    decline_dismissively: independent meeting triggers regardless

versions:
  ANCHOR: stayed, shaped culture, agency's conscience
  INDEPENDENT: left quietly, still posts, occasionally mentions agency positively
  GONE_CLEAN: door closed so softly you almost didn't hear it
```

---

### Kai — The Wildcard
```
handle: @kaiclutch
age: 20
home_city: Seoul
niche: competitive gaming / esports adjacent
subs_at_signing: 15000 (declining)
availability: mid-game unlock

traits: [Sprinter, Hungry, Instinctive]

hidden_mechanics:
  the_incident:
    content: variable — not what headlines said, but not nothing
    investigate_before_signing: costs 1 agency action, reveals full story
    not_investigated: handle blind when resurfaces — higher stakes
    resurface_triggers: rival mention, brand due diligence, journalist inquiry

  relocation_event:
    trigger: high trust + mid-game arc progress
    feed: mentions thinking about moving to Taipei
    encourage: relocates in 2 weeks
    ignore: moment passes

pivots:
  P1 "The Incident Resurfaces":
    investigated_get_ahead: trust forms fast, used to being handled not partnered
    not_investigated_defend: respect for instinct, then full story, real-time decision
    not_investigated_cautious: watches Ren hedge, files it

  P2 "The Slowdown":
    mandate_proactively: pushes back "I'm fine" — pushback is the tell
    suggest_let_decide: declines, two weeks later mandatory, feels the difference
    dont_act: breaking point fires (8hr stream, mid-sentence logout, 3 days silence)

  P3 "The Real Conversation":
    stay_in_it: builds something slower, audience shrinks/deepens
    redirect_to_strategy: door shuts immediately
    involve_hana: works if Kai fragile + Hana relationship strong

versions:
  REBUILT: slower, realer, talks about incident on own terms
  RUNNING: still fast, fine right up until not
  GONE_BEFORE_END: left at a sprint, note was short, Hana read it twice
```

---

### Chemistry Map (Starting Values)
```
         MIRAE  YUKI  CATO  SABLE  KAI
MIRAE     —     65    30    45     35
YUKI      65    —     20    70     40
CATO      30    20    —     35     75
SABLE     45    70    35    —      25
KAI       35    40    75    25     —

All scores are starting values. Everything moves.
```

---

## CHARACTERS — STAFF

### Availability by Stage
```
Early Boutique:  Tomás OR Joon (one slot, player chooses)
Mid Boutique:    Priya, Dana unlock
Late Boutique:   Felix unlocks
Mid-size:        Miso unlocks (last — changes agency's center of gravity)
```

---

### Tomás — Editor
```
age: 25 | home: Manila (Remote)
trait: Collaborative
voice: "Just tell me what you're actually going for. I need the real answer, not the brief."

structural_dependency:
  needs: direct access to talent
  breaks_in: tiered or siloed structure
  symptom: talent says edits don't feel right, Tomás could explain why, nobody asked

relationship_risk:
  TOMÁS × MIRAE: quiet, real, mutual
  complication: unconsciously prioritizes her editing, quality gap becomes visible
```

### Priya — Social & Community Manager
```
age: 27 | home: Singapore (Remote)
trait: Values-Driven
voice: "I can write this post. But I think we're telling the audience something we'll walk back in six weeks."

structural_dependency:
  needs: clear brand direction from Ren/Hana directly
  breaks_when: receiving direction from Miso instead
  escalation: contradictory instructions to talent during crisis

loyalty_risk:
  condition: values diverge, talent asks her to be honest
  action: tells them — believes in them more than the institution at that point

relationship_risk:
  PRIYA × CATO: high energy, high chemistry, reads his community management personally
```

### Joon — Talent Coach
```
age: 31 | home: Seoul
trait: Empathic
voice: "Does this person actually want what we're building toward? I can push anyone. Question is whether they'll still be here in a year."
background: former streamer, burned out at 29

structural_dependency:
  needs: protected uninterrupted time with talent
  breaks_when: Operations treats sessions as moveable slots
  symptom: talent hits burnout Joon could have caught

back_channel:
  condition: Joon + talent + Hana relationships all high
  effect: surfaces things talent aren't ready to tell Ren

relationship_risk:
  JOON × SABLE: if forms, can no longer coach her — lose best warning system for most self-contained talent
```

### Miso — Brand Partnerships Manager
```
age: 29 | home: Tokyo
trait: Dealmaker
voice: "The numbers are good. The brand is clean. I don't see the problem."
blind_spot: can tell you what a deal pays, not what it costs in identity

structural_dependency:
  needs: clear mandate from Ren on identity vs revenue line
  without_it: optimizes toward revenue by default

BETRAYAL ARC:
  conditions: [autonomous authority, transactional Ren relationship, competing agency approaches]
  action: routes information to competitor (not dramatic — sharing over coffee)
  detection: brand knows something only insider could, Hana hears from contact, Dana sees anomalies

relationship_risk:
  MISO × KAI: tailors pipeline around Kai, roster imbalance builds
```

### Felix — Operations Manager
```
age: 32 | home: Taipei
trait: Process-Oriented
voice: "I can make this work. But I'm going to need everyone to actually use the calendar."

structural_dependency:
  breaks_when: given authority over creative decisions
  symptom: talent feels managed → Joon loses session access → scheduling crisis

QUIET SABOTAGE ARC:
  conditions: passed over for expected role, or authority diminished
  action: passive-obstructive, technically correct, practically unhelpful
  detection: Dana (things slow), Joon (can't get answers), Hana (has something on his mind)
  resolution: direct conversation about the moment he felt diminished

alliance_risk:
  FELIX × DANA: excellent aligned, dark = unified positions before every meeting
```

### Dana — Finance & Runway Manager
```
age: 30 | home: Taipei
trait: Pragmatic
voice: "I understand why you want to do this. I need you to understand what it costs. Then you decide."

confidence_erosion:
  trigger: Ren consistently overrides projections
  result: "I'm thinking about taking on other clients" — not a threat, information

hana_alignment:
  if both Hana AND Dana tell Ren the same thing:
  ignoring both = riskiest single decision in the game
```

### Staff Detection System
```
relationship_formation_triggers:
  - 2 characters work closely 4+ consecutive weeks
  - crisis weathered together
  - one advocates for the other publicly
  - shared late nights (feed flavor)

detection_requires:
  feed_attention:        warm entries = earliest signal
  hana_read:             flags if relationship ≥ 50
  joon_back_channel:     if all trust conditions met
  staff_checkins:        1-on-1 secondary slot occasionally
  dana_anomalies:        financial irregularities = last but clearest
```

---

## S01 SCHEDULE SYSTEM

### Primary Slots (one per talent per week)
```
STREAM_REGULAR:    energy -15  | growth slow/sustainable | morale vs expectation
STREAM_PUSH:       energy -30  | growth fast             | morale_risk if disappoints
REST:              energy +25  | no growth               | morale stabilizes, bond+ if check-in
EVENT_APPEARANCE:  energy -20  | popularity+             | morale depends on trait fit
COLLAB:            energy -20  | chemistry+              | risk high if chemistry < 40
TRAINING:          energy -10  | skill+                  | morale neutral-to-good
BRAND_EXECUTION:   energy -15  | cash_in                 | morale depends heavily on fit
```

### Secondary Slots (optional)
```
EXTRA_PREP:      morale risk reduced on Push/Event | wasted on Instinctive
SOCIAL_PUSH:     subs+ small | energy -5
PERSONAL_TIME:   morale +5 | burnout_track small reduction
HANA_1ON1:       hana_relationship+ | may surface hidden talent state
OFFLINE_HANGOUT: requires Local proximity + relationship≥35
```

### Core State Interactions
```
high_energy + low_morale  → present but phoning it in, streams underperform
low_energy + high_morale  → burning out invisibly on love of craft
low_energy + low_morale   → breaking point territory

viewer_expectation: rolling average of recent performance
  below → morale hit
  above → morale bonus + sub spike
  push_sustained → expectation creeps up (treadmill effect)
```

---

## S02 BRAND DEAL PIPELINE

### Stages
```
DISCOVERY → PITCH → NEGOTIATION → EXECUTION → AFTERMATH
```

### Deal Attributes
```
brand_name:        string + reputation_tier (local/regional/national/premium)
payout:            number
fit_score:         rough signal, not precise — requires player interpretation
execution_demand:  weeks required + schedule intrusion level
risk_flag:         boolean
```

### Execution Quality Inputs
```
talent_morale:  highest weight
fit_score:      medium weight
extra_prep:     boolean modifier

outcomes: excellent | adequate | poor | failed(rare)
```

### Key Rules
```
misalignment_penalty:
  bad fit flagged by talent/Hana → executed anyway → poor result → Hana relationship hit

audience_trust_drift:
  too many off-brand deals → subscribers become passive viewers
  slow to appear, hard to reverse
```

---

## S03 PERSONALITY & CHEMISTRY

### Traits
```
ENERGY: Night Owl | Sprinter | Marathon Runner
SOCIAL: Extrovert | Introvert | Performer
WORK:   Perfectionist | Instinctive | People Pleaser
AMBITION: Hungry | Craftsperson | Coasting
```

### Chemistry
```
range: 0-100
≥70: collabs overperform, mutual morale boost, cover in bad weeks
40-69: neutral baseline
<40: underperform, passive tension, drama seed risk
forced_collab_below_40: immediate drama seed

builds: trait compatibility, shared collabs (±5), crises together, player decisions
decays: favoritism, forced conflict, ignored friction
note: not permanent in either direction
```

---

## S04 BURNOUT & BREAKING POINTS

### Burnout Track
```
range: 0-100 | hidden: true | separate from energy
```

### Accumulation
```
- energy < 30 for 2+ consecutive weeks
- morale < 40 for 2+ consecutive weeks  
- schedule consistently mismatches traits
- executes bad-fit deal talent flagged
- active conflict with another talent
- no check-ins 4+ consecutive weeks
```

### Reduction
```
- rest-heavy weeks (sustained)
- high morale periods (sustained)
- Hana 1-on-1 resolves underlying issue
- genuine care decisions
note: cannot instantly fix — time and care required
```

### Warning Stages (symptoms only, no number shown)
```
CLEAR (0-30):       normal
STRAINING (31-55):  "seemed off" in feed, performance variance increases
FRAYING (56-75):    "sleeping badly", 1-on-1 (!) appears — BEST intervention window
ON_EDGE (76-90):    Hana raises concern, quality drops, brands may notice
BREAKING (91-100):  crisis event fires
```

---

## S05 DRAMA SYSTEM

### Lifecycle
```
SEED → TENSION → WARNING → CRISIS → RESPONSE → AFTERMATH
```

### Timing Windows
```
tension:  2-4 weeks, subtle, easy to miss
warning:  2-3 weeks to act, named signal, distinct feed treatment, 1-on-1 option
crisis:   fires if unaddressed
```

### Memory Tags (permanent)
```
format: "[Character] [perception of what Ren did]"
effect: influences all future interactions with that character
note: some consequences delayed — felt weeks or months later
```

---

## S06 GROUP SYSTEM

### Formation
```
members: 2-3 talents
requirements:
  - all pairs chemistry ≥ 50
  - collabed at least twice each
  - no active drama
  - theme selected
```

### Themes
```
LATE_NIGHT_VARIETY | COMPETITIVE_GAMING | LIFE_VLOG | CREATIVE
each: different audience type, brand deal pool, vulnerability
```

### Health Effects
```
healthy (morale≥60, drama_seeds≤1):
  - group streams outperform individual
  - individual slip-ups forgiven more
  - brand value increased
  - members cover off-weeks

unhealthy:
  - internal conflict is public
  - identity override risk (especially Craftsperson/Hungry)
  - dissolution: graceful vs catastrophic
```

---

## S07 STAGE PROGRESSION

```
GARAGE → BOUTIQUE:
  - 2+ talents subs≥10k
  - reputation≥40, runway≥8wks, hana_relationship≥60
  unlocks: brand pipeline, first staff slot, roster+1

BOUTIQUE → MID-SIZE:
  - 4+ talents, 2+ subs≥100k
  - 1 group formed, reputation≥65, hana_relationship≥40
  unlocks: investors, multi-talent campaigns, org structure matters
  scripted: Hana fight (content from history)

MID-SIZE → CORPORATE:
  - 6+ talents, 1+ subs≥500k
  - staff≥4, sustained revenue
  - PLUS: investor_pressure OR hana_departure
  note: Corporate = ending, not progression layer
```

---

## S08 HANA'S ARC

```
arc_path determined by accumulated history, not single choices

high_weight_inputs:
  - consult vs inform on major decisions
  - talent vs revenue priority
  - credit sharing behavior
  - follow-up after hard conversations
  - first burnout crisis response
  - first major brand negotiation involvement

paths:
  A stays_and_aligns:   own actions, early warning active, emotional anchor
  B stays_but_drifts:   present, less herself, feed entries shorter/formal
  C leaves:             external, resurfaces once, shaped by how she left
```

---

## S09 SCOUTING SYSTEM

```
actions:
  browse_pipeline:   free, low quality
  targeted_search:   1 action, specify niche/trait
  referral:          from existing talent, chemistry pre-seeded
  competitor_watch:  1 action + ethics flag

prospect_info:
  visible:  niche, subs, 1-2 impressions, known reputation
  hidden:   traits, burnout vulnerability, chemistry potential

signing:
  standard | talent_favorable | agency_favorable
  hana_watches: consistently agency_favorable when leverage_low → damages her read
```

---

## S10 STAFF & ORG STRUCTURE

```
structures:
  FLAT:          all to Ren+Hana | breaks at 5+ staff
  FUNCTIONAL:    by Creative/Business/Ops | breaks on wrong lead
  TALENT_CENTRIC: by talent cluster | breaks on inter-talent drama
  HYBRID:        functional+talent-centric | requires hana_relationship≥50

org_review:
  frequency: once per stage
  shows: structure, friction points, flow gaps, Hana read (if rel≥50)
  does_not_show: the right answer

dysfunction_pattern:
  bad structure → quiet signal (wk1-2) → louder signal (wk3-4) → crisis (wk5-6)
```

---

## S11 ENDINGS

```
HARD_LOSE:
  cash_runway_zero | reputation_collapse | hana_left+reputation<30

THE_REAL_THING:    hana high + 3 loyal talents + avoided Corporate + 1 talent from Garage
EMPIRE:            Corporate reached (tone: bittersweet/hollow/one line depending on Hana)
SPLIT_CUSTODY:     Hana left and built her own (tone: from history)
IT_MEANT_SOMETHING: cash fail + hana≥60 + talent loyalty high (unexpectedly warm)
THE_COST:          consistent low morale mgmt + burnout mishandled + Hana gone badly
STILL_HERE:        never Mid-size + high relationships + time played (quietest, maybe most resonant)
```

---

## S12 PROXIMITY & OFFLINE

### Starting Proximity by Ren's City
```
TAIPEI: Local=[Hana,Mirae,Yuki,Cato,Sable,Felix,Dana] Regional=[Miso,Joon,Kai]
TOKYO:  Local=[Miso] Regional=[all Taipei cast, Joon, Kai]
SEOUL:  Local=[Joon,Kai] Regional=[all Taipei cast, Miso]
Distant always: Tomás (Manila), Priya (Singapore)
```

### Offline Interactions
```
CASUAL_HANGOUT:       rel≥35, Local | relationship+8, morale+5 | human feed entry
WORK_SESSION:         rel≥25, Local/Regional | skill+, morale+, rel+
OFFLINE_COLLAB:       rel≥40, Local/Regional | chemistry+15, sub spike, trust+
LATE_NIGHT_DEBRIEF:   rel≥45, Local ONLY | rel+10, surfaces hidden state
SECRET_DATE:          rel≥60 + romantic_flag, Local
INTERVENTION:         rel≥50, Local (or fly Regional for extra weight)
```

### Visit System
```
cost: 1 agency action + travel budget | duration: 1 week
min_relationship: 30
occasions: agency_business | check_in | event_support | emergency (no req)
visit_memory: +5 lasting bonus if goes well, referenced in feed weeks later
```

### Relocation Events
```
KAI:  mid-game — encourage Taipei mention → relocates in 2 weeks
YUKI: late Boutique — visa event, 3 options
CATO: rootlessness → gravitational pull to Manila
HANA: Ren not Taipei + rel degrading → residency offer elsewhere
REN:  can relocate once per game (expensive, 3wks reduced efficiency)
```

---

*Repo: https://github.com/andrihsieh-glitch/streaming-agency-game*
*Update this file when systems change. Build chat fetches from raw URL each session.*
