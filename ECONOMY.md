---
type: design
domain: Research
updated: 2026-07-01
---

# Grid Defense — Economy & Wave Structure

How the player earns and spends **gold**. The defining choice: **questions are the income engine.** You answer electrical-knowledge questions before each wave; correct answers pay gold; gold buys and upgrades the towers that stop faults. So studying the material directly makes you stronger. Companion to [[grid-defense/DESIGN|DESIGN.md]]; questions come from an uploaded file per [[grid-defense/CONTENT-SCHEMA|CONTENT-SCHEMA.md]]. Conforms to [[SHARED-DESIGN]] (§4: learning is never punished with a hard wall).

## Wave cadence — two wave types
| Wave # | Type | Pre-wave questions | Gold rule |
|---|---|---|---|
| not divisible by 5 (1–4, 6–9, 11–14, …) | **Normal** | **2** questions | each correct = flat gold, no penalty for wrong |
| every 5th (5, 10, 15, …) | **Boss** | **streak of up to 3** | escalating gold per correct-in-a-row; **first wrong ends the streak** |

- Normal waves: standard fault count/mix, escalating slightly each wave.
- Boss waves: tougher, denser wave (higher HP / more fault types / a mini-boss fault). The escalating boss-question gold is what lets you afford the defense the boss demands, so the harder round and the bigger payout line up.

## Normal wave — flat reward, no risk
- Before the wave, ask **2 questions**, one at a time, drawn from the active bank.
- Each correct → **+`normalGold`**. Two questions → up to **+2 × `normalGold`** for the wave.
- Wrong → nothing gained. **No penalty, no game-over.** The cost of a miss is opportunity cost: less gold → weaker defense.
- After each answer, show the question's `why` line, then continue.

## Boss wave — press-your-luck streak
- Before the boss wave, ask up to **3 questions in sequence**.
- Each **correct-in-a-row** pays an **escalating** amount from `bossGold` (index = current streak length).
- A **wrong answer ends the streak immediately** — remaining questions are not asked. **You keep everything already banked**; you just stop earning more this round.
- Rewards confidence/mastery on the harder round without ever hard-walling the player. (Open decision in DESIGN: keep-banked-on-miss vs all-or-nothing; current design keeps banked.)

Example streak (with starter numbers below): 1 correct = +25 · 2 in a row = +25+40 = +65 · 3 in a row = +25+40+60 = **+125**. Miss the 2nd = keep the +25 and stop.

## Starter balance (all tunable in one block)
**Playtest-tuned 2026-07-03** (autoplay campaigns: 100% accuracy wins 15/15 with ~4/10 health left; ~75% reaches the wave-10 boss; ~50% falls around the first boss — knowledge maps directly to progress). Values live in the build's `BAL` block.

| Constant | Value | Note |
|---|---|---|
| `startGold` | 40 | seats ~1–2 basic towers |
| `normalGold` | 12 / correct | 2 questions → max +24 / normal wave |
| `bossGold` | `[30, 45, 65]` | streak rewards; index = correct-in-a-row (max +140 for a perfect streak) |
| tower costs | Fuse 20 · Breaker 30 · GFCI 40 · AFCI 40 · SPD 45 · Grounding node 35 | priced so full fault coverage isn't affordable early |
| upgrade cost | ~0.6 × base / tier | range / clear-rate / amp-rating tier |
| escalation | +0.6 enemies/wave · +5% HP/wave · boss ×1.3 count + a heavy ×4-HP hazard | the pressure curve the income has to race (onboarding hazards run ~12% lighter — 6 types spread the towers thinner than theory's 5) |

Prices are set so **you can't cover every fault type from `startGold`** — earning gold from questions is how you scale. This is what makes the questions matter mechanically, not just cosmetically.

## Spending gold
Between waves (Shop/Place phase): buy new towers from the Devices table, or upgrade a placed tower (range, clear-rate, amp-rating tier). Selling optional. Amp-rating upgrades on a breaker tie directly into the sizing lesson — an oversized breaker misses smaller overloads.

## Difficulty routing (optional)
If the uploaded questions carry a `difficulty` tag (`easy`/`medium`/`hard`), normal waves prefer `easy`/`medium` and boss waves prefer `medium`/`hard`. If the bank has no difficulty tags, it's one flat pool. See [[grid-defense/CONTENT-SCHEMA|CONTENT-SCHEMA.md]] §7.

## Pool management
Draw without repeats until the pool is exhausted, then reshuffle; never ask the same question twice in a row. Minimum **5 valid questions** to start (else the current bank — the bundled sample if none was uploaded — is kept and the user is warned).

## Why this design
Making questions the currency doubles down on "the mechanic teaches the concept": the player is rewarded for electrical knowledge with the exact resource (gold) that builds a better defense. It also cleanly separates **content** (the uploaded question bank — swappable by any instructor, no dev) from **systems** (waves, gold, towers — the tunable engine).

## Parent
[[grid-defense/DESIGN|DESIGN.md]] · [[grid-defense/CONTENT-SCHEMA|CONTENT-SCHEMA.md]] · [[learning-games]] · [[SHARED-DESIGN]]
