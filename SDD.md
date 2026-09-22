# DARKNESS — Software Design Document (SDD)

## 1. Overview

**DARKNESS** is a browser-based, single-file HTML5 slot game built with vanilla
JavaScript and the HTML5 Canvas API. It combines a classic 5×4, 20-payline
reel grid with three layered bonus systems: a growing win-streak multiplier,
a collectible bonus-wheel feature, and a scatter-triggered free spins mode
with retriggers. A companion Python Monte Carlo simulator is used offline to
balance and validate the game's mathematical model before any parameter goes
live in the client.

The design goal is a game that feels **unpredictable and atmospheric**
rather than mechanically transparent — pacing, feedback, and reveal moments
are built to create anticipation, not to expose the underlying probability
model to the player.

---

## 2. Core Gameplay Loop

1. Player sets a bet and spins.
2. A 5-reel × 4-row symbol grid is generated.
3. All 20 fixed paylines are evaluated for matching symbol runs.
4. Line wins are paid, then multiplied by a **streak-based multiplier** that
   grows across consecutive winning (or feature-triggering) spins and resets
   on a non-event.
5. Certain symbol patterns can trigger one of two bonus systems:
   - A **collectible bonus feature** tied to one specific symbol, resolved
     through an interactive reveal (see §4).
   - A **scatter-triggered free spins mode**, with its own internal
     streak-multiplier state and the ability to extend itself mid-round.
6. Balance, statistics, and session totals update; the player continues or
   stops.

Exact symbol weights, payout tables, and trigger thresholds are treated as
confidential game-balance data and are intentionally not documented here —
see §7 for how they're managed instead.

---

## 3. Reel & Payline Engine

- 12 symbols across a shared weighted reel strip (weights are tiered — a
  handful of common symbols, a mid tier, and rarer high-value symbols).
- One symbol is reserved as the **scatter** and never pays on a payline; it
  exists purely to drive the free-spins system.
- 20 fixed line shapes are evaluated per spin using a **precomputed lookup
  table**: every possible 5-symbol line combination is enumerated once at
  startup, each mapped to its resolved winning symbol, match count, and
  payout. At runtime a spin's line is reduced to a single integer key and
  the result is a lookup, not a recount — this keeps evaluation fast even
  during long free-spin sequences with many extra spins.

---

## 4. Bonus Systems

### 4.1 Streak Multiplier
Consecutive spins that "do something" (pay a line, or trigger the scatter
feature) climb a fixed multiplier ladder; a spin that does neither resets
the ladder to its base. The ladder applies to line wins directly. This
system runs independently inside the free-spins mode with its own local
state, so a free-spins round always starts the climb from scratch.

### 4.2 Collectible Bonus Wheel
One specific symbol, when it fully covers a payline, triggers an interactive
bonus reveal presented as a spinning wheel of weighted multiplier segments.
Design intent:
- The reward value is determined the instant the trigger fires — the wheel
  animation does **not** influence the outcome, it only reveals it.
- The reveal is built around suspense pacing: escalating tension audio,
  near-miss highlighting as the pointer sweeps past high-value segments,
  a physically-plausible overshoot-and-settle landing instead of an
  abrupt stop, and dynamic on-screen captions that track the spin's phase.
- The spin is player-initiated (a button integrated into the wheel's hub)
  and cannot be skipped or auto-triggered.
- All suspense/animation logic is presentation-only and is kept strictly
  separate from the RNG call that decides the result, so visual tuning can
  never change the underlying odds.

### 4.3 Scatter / Free Spins
Landing enough scatter symbols on a payline awards a batch of free spins.
Design characteristics:
- Higher scatter counts award disproportionately more spins (a small count
  is a near-miss, not a reward).
- Free spins can **retrigger**, adding more spins to the same round when the
  scatter condition is met again mid-round.
- The collectible bonus feature (§4.2) can also trigger during free spins,
  and is fully tracked (count, value, and multiplier draw) independent of
  whether it originated on a paid spin or a free spin — this was a
  deliberately hardened area of the implementation after an early bug where
  bonus events occurring only inside a free-spins round were undercounted
  in reporting (money paid to the player was always correct; only the
  statistics breakdown was affected, and this has since been fixed and
  verified).

---

## 5. System Architecture

```
┌─────────────────────────────┐
│  UI Layer (DOM + Canvas)    │  HUD, reel animation, bonus wheel,
│                              │  free-spin overlay, sound
├─────────────────────────────┤
│  Game Engine (JS)            │  spin generation → payline lookup
│                              │  → streak/multiplier state →
│                              │  bonus trigger checks → balance update
├─────────────────────────────┤
│  RNG                          │  cryptographically-seeded random source
└─────────────────────────────┘
```

- **Single-file delivery**: the entire game (markup, styles, logic) ships as
  one self-contained `.html` file — no build step, no external runtime
  dependencies, easy to host or embed anywhere.
- **Deterministic-in-structure, random-in-outcome**: all payout/trigger
  logic is table- and lookup-driven rather than branchy conditionals, which
  keeps the client fast and makes the model easy to mirror in the offline
  simulator for verification.
- **State isolation**: base-game state (balance, streak) and free-spins
  state (local streak, remaining spins, retrigger count) are kept separate,
  so a free-spins round can never leak into or corrupt base-game session
  stats.

---

## 6. Offline Simulation & Balancing Tooling

A companion Python tool (not shipped to players) mirrors the client's exact
payline/trigger logic and runs large-scale Monte Carlo simulation
(hundreds of millions of spins) to validate:

- Overall return-to-player converges to the intended target as sample size
  grows.
- Each bonus system's individual contribution behaves as designed, both in
  trigger frequency and payout distribution.
- Volatility (variance of per-spin outcome) and bankroll survival curves
  (how long a fixed starting balance typically lasts) fall within the
  intended player experience band.
- Statistical sanity checks (e.g. z-score comparisons between observed and
  theoretical averages) catch implementation bugs — this is how the
  free-spins bonus-tracking issue in §4.3 was originally discovered.

The simulator and the client are kept in lockstep: any change to a payout
table, weight, or trigger rule is applied to both, and a fresh simulation
run is required before the change is considered validated.

---

## 7. Configuration & Confidentiality

Game-balance parameters (symbol weights, exact payout values, bonus
multiplier pools, trigger thresholds, free-spin award tables) live in a
small set of clearly-marked constant blocks at the top of both the client
and the simulator, rather than being scattered through the logic. This
makes them easy to tune centrally — and easy to keep out of public
documentation, since the *shape* of the system (how many tiers, how
triggers compose, how bonuses interact) is what's documented, not the
specific numbers.

---

## 8. Tech Stack

| Layer | Technology |
|---|---|
| Client | HTML5, vanilla JavaScript (ES5-compatible), Canvas 2D |
| Audio | Web Audio API (procedural tones/noise, no audio files) |
| Simulation | Python 3, NumPy (vectorized batch evaluation + lookup-table engine) |
| Delivery | Single static `.html` file — no server, no build pipeline |

---

## 9. Suggested Repository Layout

```
/
├── darkness.html          # the game (client)
├── simulator/
│   └── darkness_sim.py    # offline Monte Carlo balancing tool
├── docs/
│   └── SDD.md              # this document
└── README.md
```

---

## 10. Non-Goals

- No server-side component — this is a client-only demo/prototype build,
  not a certified real-money product.
- No claim of regulatory RNG certification is made or implied by this
  document.
- Exact mathematical parameters are intentionally omitted from this
  document by design (see §7).
