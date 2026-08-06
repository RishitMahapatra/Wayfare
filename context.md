# Pomodoro Journey App — Product Context

> Single source of truth for what we're building, why, and what's in/out of scope. Reference this at the start of every dev session.

---

## 1. One-line pitch

A pomodoro focus timer reframed as a real-world travel journey: each focus session moves the player between actual cities using road/train/sea/air, unlocking countries, monuments, and rare natural-landmark "expedition" badges over time.

---

## 2. Product principles

- **Simplicity over feature count.** Follow the S-curve discipline: fewer, sharper mechanics beat more, cluttered ones.
- **Focus is the product; travel is the wrapper.** Every design decision must reinforce the actual pomodoro habit, not distract from it.
- **Data-driven, not hardcoded.** Cities, routes, costs, upgrades, badges all live in JSON/config files. Content scales by editing data, not rewriting code.
- **Ship the core loop first.** No leaderboards, streak freezes, mystery boxes, cosmetics, or social features in v1. Prove retention on the core loop before adding.
- **Anticipation over loss aversion.** Rewards are framed as "what will I unlock next," not "don't break your streak."
- **Competence signals over badge theater.** Personal-best times and completion rings matter more than a wall of trophies.

---

## 3. Core game logic

### 3.1 World structure

- **Nodes = cities**, defined in a JSON graph:
  ```
  { id, name, country, lat, lng, coastal: bool, initial_road_links: [city_ids] }
  ```
- **Fixed starter city set per country** (e.g. India: Delhi, Jaipur, Mumbai, Bangalore, Hyderabad, Chennai, Kolkata).
- **Cohesive network** — no domestic/international split. Inter-country travel is allowed by any mode as long as distance is within that mode's session-time cap.

### 3.2 Transport modes

Order (slowest/cheapest → fastest/costliest): **Road < Train << Sea = Air**

| Mode  | Availability                          | Session time cap |
|-------|---------------------------------------|------------------|
| Road  | Between any two connected land nodes  | Max 12 hr        |
| Train | Requires station unlock at both ends  | Max 12 hr        |
| Sea   | Coastal-to-coastal only, port unlock  | Max 12 hr        |
| Air   | Requires airport unlock at both ends  | Max 12 hr        |

Beyond 12 hr for a given mode, that mode becomes unavailable for the route — player must upgrade to a faster mode.

### 3.3 Route upgrade tiers

Each route × mode combo has **3 upgrade levels**, each reducing total pomodoro time and tightening cycle structure.

| Level | Example (DEL → Jaipur, Road) | Coin cost |
|-------|------------------------------|-----------|
| L1    | 3hr = 3× (50/10) cycles      | Free (default) |
| L2    | 2.5hr = 4× (30/10) cycles    | Distance × multiplier |
| L3    | 2hr = 4× (25/5) cycles       | Higher multiplier |

Higher levels pay out more coins per session (harder focus discipline = bigger reward). See Appendix A.2 for the exact multiplier values.

### 3.4 Stops & layovers

- User-configurable number of stops (road) or layovers (train/sea/air).
- Each stop/layover deducts a small coin "tax" from the total reward.
- Tradeoff: more stops = easier to fit into daily schedule, but lower payout.

### 3.5 Unlock economy

Two-layer gate:

1. **Time-gated (infrastructure):** airport/port/station at a city unlocks after cumulative focus hours logged at that city cross a threshold. No coin cost — see Appendix A.3.
2. **Coin-gated (routes):** once infrastructure unlocks, the **first outbound route on that mode is free**; every additional route from that hub costs coins.
3. Rule applies uniformly to road/train/sea/air.

### 3.6 Base & movement

- **Base = home city**, set once at signup.
- **Return to Base** is always available from any node (calculated fresh, not free but never gated).
- Player freely chooses per session: return to base, or continue outward.
- **Away-from-base surcharge**: routes from non-base cities cost more, capped at +50%:
  ```
  route_cost = base_cost × (1 + min(0.5, dist(current, base) / dist(base, farthest_unlocked)))
  ```
  Soft nudge back to base, not a hard restriction. `farthest_unlocked` is recalculated fresh at the time of the calculation — see Appendix A.1.

### 3.7 Initial road network

- First launch: every city connected by **road only** to its **3 nearest cities** from the fixed starter list of its country.
- Hardcoded per city in the JSON (`initial_road_links`), not calculated at runtime.

---

## 4. Content & collectibles

### 4.1 Monuments ("Landmark Detours")

- Satellite nodes attached to a parent city (Taj Mahal → Agra, Christ the Redeemer → Rio).
- Unlocked as **optional short bonus quest** once player arrives at parent city.
- One extra ~25-min cycle to complete.
- Awards a **distinct landmark badge**, separate from the parent city's passport stamp, plus a coin payout — see Appendix A.7.

### 4.2 Natural landmarks ("Expedition Challenges")

- Everest, Sahara, Lake Victoria, etc.
- Unlocked via level/coin threshold, not tied to a specific city route.
- Long single **cumulative** pomodoro (e.g. 8hr for Everest), **resumable across days**.
- One-time, non-repeatable.
- Awards a **rare/legendary badge** — prestige content, badge-only (no coin payout).

### 4.3 Passport stamps

- One per **country**, awarded on first city visit in that country.
- Displayed on a "passport" screen — macro progress view.

### 4.4 Badges — three tiers

1. **City/country badges** (common) — visiting a new node.
2. **Landmark badges** (uncommon) — completing a monument detour.
3. **Expedition badges** (rare/legendary) — completing a natural-wonder challenge.

### 4.5 Completion rings

- Each country shows a **progress ring** (e.g. "4/7 cities visited").
- Apple Watch–style closure pull: users want to close the loop.
- Also applies at zone and world level.

### 4.6 Personal-best segment times

- For every route the player has traveled, store fastest completion time.
- Displayed as: "Fastest DEL → Jaipur: 2hr 45min, set 3 sessions ago."
- Personal competition, no cohort matching. Replaces the leaderboard itch with a healthier competence signal.

---

## 5. Cut from v1 (parked, revisit post-launch)

- Global/social leaderboards (need live user base to feel alive)
- Streak system (Duolingo-style flame + freezes) — good idea, defer to v1.1
- Mystery-route sessions (variable reward) — defer to v1.1
- Cosmetic IAP (card skins, mascot outfits) — defer to v1.1
- 3D globe / rich in-session scenery — deferred; v1 uses flat map with animated dot
- Live social features (co-focus, friend tagging) — deferred indefinitely

---

## 6. Visual system (v1)

- **Countries:** flat solid color, one hue per country (Radiooooo-style).
- **Natural landmarks:** small illustrated icon markers on the map (mountain silhouette, dune, water icon) — visually distinct from country fills, signaling "special content."
- **Monuments:** small pin/icon at actual coordinates, distinct from city pin.
- **In-session view:** static 2D map plate, animated dot/vehicle icon moving along a dashed route line, large timer, cycle progress dots.
- **Unlock reveal:** single Lottie animation (card flip / unblur / confetti burst), reused across every unlock, image swapped underneath.
- **No 3D, no per-scene AI-generated art in v1.**

---

## 7. Tech stack

- **Frontend:** Flutter (Dart) — single codebase for iOS + Android.
- **Backend:** Supabase (Postgres + auth + storage) — open source, generous free tier.
- **State:** local-first for timer/session; sync to Supabase for cross-device continuity and future social features.
- **Animation:** Lottie for reveal moments; Flutter's built-in `AnimationController` for timer/dot movement.
- **Data:** all cities/routes/costs/badges stored in JSON files bundled with app; loaded into memory at launch.

---

## 8. Revenue model (v1)

**Freemium, no ads.**

- **Free tier:** full core loop, capped at 1–2 countries/zones organically.
- **Paid (Explorer Pass subscription):** unlimited daily sessions, all upgrade tiers, faster coin earn rate, exclusive card art (later).
- **Coin top-ups (IAP):** buy coins directly instead of grinding — impulse-purchase secondary lever.
- Cosmetics deferred to v1.1.

---

## 9. Success signals (what we watch post-launch)

- D1 / D7 / D30 retention on the core loop.
- Average sessions/day per active user.
- Coin economy balance: are players earning enough to unlock at a satisfying pace, or grinding?
- Passport completion rate per country — is the completion ring pull working?
- Segment-time revisits: are players re-running the same route to beat their PB?

If retention holds without leaderboards or streaks, we've validated the anticipation model. Then, and only then, layer in v1.1 mechanics.

---

## Appendix A — Clarifications (2026-08-06)

Resolved during test-version scoping. These are binding — treat them as part of the spec, not open questions.

### A.1 Away-from-base surcharge denominator (§3.6)

`farthest_unlocked` in the surcharge formula is **dynamic**, recalculated fresh at the moment of each cost calculation — not snapshotted at unlock time. The surcharge is a soft nudge, not a pricing guarantee; players see the cost at decision time, not a price list that changes under them. Snapshotting would add state-tracking complexity for zero user-facing benefit.

### A.2 Route upgrade multipliers (§3.3)

Defined centrally in `config.json`, never hardcoded per route:

```json
"upgrade_multipliers": {
  "L2_cost": 1.5,
  "L3_cost": 2.5,
  "L2_payout_bonus": 1.1,
  "L3_payout_bonus": 1.2
}
```

- Upgrade cost = `base_distance_cost × L2_cost` (or `L3_cost`)
- Payout = `base_coin_reward × L2_payout_bonus` (or `L3_payout_bonus`)

### A.3 Infrastructure unlock cost (§3.5)

Confirmed: infrastructure (airport/port/station) is **purely cumulative-hours-gated, zero coin cost, ever**. Coins only ever gate *routes* (the 2nd+ outbound route from a hub). Clean separation: time unlocks capability, coins unlock reach.

### A.4 Expedition persistence sequencing (§4.2, test-version Phases 12/14)

Local-only persistence (Hive/SharedPreferences) is acceptable as a stopgap for expedition progress starting Phase 12. Supabase sync wraps it properly in Phase 14. The dev-mode "no persistence assumptions" rule means "include a reset button," not "don't persist at all" — local persistence is expected and normal, just resettable.

### A.5 Notifications under time multiplier (test-version Phase 2)

Real OS notifications are suppressed entirely in dev mode; log lines only. Dev tab exposes a toggle: `notifications: real | log-only`, defaulting to `log-only` whenever the time multiplier is above 1x. Prevents notification spam and keeps testers focused on the log during rapid-fire testing.

### A.6 "Instant" time-multiplier semantics (test-version Phase 2)

"Instant" does not mean skipping straight to session-complete. It ticks at effectively-infinite speed (~1ms per tick) but still passes through every state transition sequentially (focus → break → focus → complete), firing and logging each one. This keeps the state machine genuinely exercised so the Phase 2/4/12 checkpoints remain meaningful.

### A.7 Monument detour reward (§4.1)

Monument detours award **both** a coin payout and the landmark badge — not badge-only. Payout = 50% of a normal single-cycle session reward. Rationale: a 25-minute focus session with zero coin reward feels punishing given the real focus time invested. Badge-only remains correct for expeditions (§4.2) — those are prestige, rare, opt-in endurance content — but a quick detour should still feed the coin loop.

### A.8 Dev-tab force-unlock scope (test-version Phase 5)

One universal "Force-unlock" panel, not separate controls per system. It's a searchable list of every unlockable entity in the game — city infrastructure, route tiers, expeditions, monuments — each with its own toggle. Scales as content grows; the Phase 12 Everest shortcut is just another row in this list, not a bespoke button.
