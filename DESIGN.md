---
type: design
domain: Research
updated: 2026-07-01
---

# Grid Defense — Design Spec

Tower defense. Strongest mechanic-to-learning fit of the three: choosing the right **protective device** for each **fault** is both the game and the lesson. Layered on top is a **question-driven economy** — the player earns the gold that buys and upgrades towers by answering electrical-knowledge questions before each wave. All question content is **uploaded by the user as a CSV or JSON file** (see [[grid-defense/CONTENT-SCHEMA|CONTENT-SCHEMA.md]]); no code edits are needed to change or add questions. Conforms to [[SHARED-DESIGN]].

## Overview
Faults spawn at a **source** and travel a fixed conductor **path** toward the **panel/load** you defend (panel health = lives). Between waves you spend **gold** to place protective **devices** (towers) on tiles beside the path; a device neutralizes faults whose **type it matches** within range, and does nothing to types it doesn't cover. Waves escalate in count/mix/speed. You **earn gold by answering questions before each wave** — the better you do, the stronger a defense you can afford. Survive all waves to win.

Two things drive the game and can each be authored without touching engine code:
1. **The wave/tower content** (faults, devices, type-match table) — data in the `CONTENT` object.
2. **The question bank** (what's asked to earn gold) — an **uploaded CSV/JSON file**, validated against the schema.

Same engine, two skins:

## Skins
| | Theory — "Grid Defense" | Onboarding — "Panel Build" |
|---|---|---|
| Framing | abstract circuit; faults by physics name | a residential panel; wire it to code |
| Fault flavor | "ground fault", "arc fault", "surge" | real scenarios: "hair dryer in a wet bathroom", "damaged cord arcing", "nearby lightning surge" |
| Bonus objective | coordination/selectivity | NEC placement (GFCI at wet tiles, AFCI at dwelling tiles) |
| Question theme | which device stops which fault, amp ratings | which device the code requires here |

Skin stays a **first-class toggle** (menu + `?skin=` param). In the question bank, `skin` is an **optional** per-row tag (`theory` / `onboarding` / `both`, default `both`) so one uploaded file can serve both audiences; see [[grid-defense/CONTENT-SCHEMA|CONTENT-SCHEMA.md]].

## Faults (enemies)
| Fault | What it is | Countered by |
|---|---|---|
| Overload | sustained current above rating | correctly-**sized** breaker or fuse |
| Short circuit | bolted low-impedance fault | fuse / breaker (instantaneous) |
| Ground fault | current leaking to ground | **GFCI** |
| Arc fault | series/parallel arcing | **AFCI** |
| Surge | transient overvoltage | **SPD** (surge protective device) |

Each fault has type, HP, and speed. Wrong device type = no effect → the fault reaches the panel and costs health.

## Devices (towers)
| Device | Stops | Cost (indicative, tunable) | Teaching hook |
|---|---|---|---|
| Fuse | short, overload (in range) | 20 | one-shot: blows and must be "replaced" (cooldown) → single-use nature |
| Circuit breaker (15/20/30 A…) | overload, short | 30 | **amp-rating selection**: an oversized breaker misses smaller overloads; correct size trips |
| GFCI | ground fault only | 40 | people-protection vs equipment-protection distinction |
| AFCI | arc fault only | 40 | arc detection ≠ overcurrent |
| SPD | surge only | 45 | transient vs steady-state protection |
| Grounding/bonding node | support: reduces ground-fault severity, enables clearing | 35 | grounding underpins fault clearing |

**Upgrades** (spend gold on a placed tower): range, clear-rate, or amp-rating tier — indicative ~60% of base cost per tier, tunable. Selling optional. Prices are set so questions matter: you can't cover every fault type from the starting gold, so earning gold from questions is how you scale your defense.

## Onboarding skin — construction safety towers (HR-friendly)
The two skins share one engine but swap the entity set. The **theory** skin uses the electrical faults ↔ protective devices above. The **onboarding** skin (used for real new-hire training) reskins those into a **construction-safety** set that HR can put in front of employees.

**HR-friendly framing (applies to the whole game, enforced in the onboarding skin):**
- Enemies are **hazards** (job-site risks moving toward the crew), never monsters/enemies. Towers are **safety controls** that **mitigate / resolve / make safe** a hazard — never "attack", "kill", "shoot", or "destroy".
- A hazard that reaches the crew is a **near-miss / incident** that lowers the **Safety score** (the health analog) — no blood, no damage/death language, no gore.
- Tone is professional, positive, inclusive, and grounded in real onboarding (OSHA-style Focus Four + electrical safety). Copy reinforces safety culture (stop-work authority, report near-misses, PPE is the last line of defense). No edgy humor, no discriminatory or violent content.
- Art uses safety-orange / hi-vis yellow-green accents on the shared palette; hazards carry a clear warning glyph, not a scary face.

**Hazards (onboarding enemies):**
| Hazard | Real-world scenario | Made safe by |
|---|---|---|
| Electrical shock | temp power / energized conductor on site | **LOTO Station**, **GFCI Cart** |
| Arc flash | working near energized gear | **LOTO Station** (de-energize), **PPE Station** (arc-rated) |
| Fall from height | unprotected edge / floor opening | **Guardrail / Fall Protection** |
| Struck-by | moving equipment, swinging load, backing truck | **Spotter / Flagger**, **Barricade & Signage** |
| Slip / trip | cords, debris, spills, poor housekeeping | **Housekeeping Crew**, **Barricade & Signage** |
| Fire / hot work | welding sparks near combustibles | **Fire Watch / Extinguisher** |

**Safety controls (HR-friendly towers):**
| Tower | Makes safe (hazard types) | Onboarding teaching hook | Theory analog |
|---|---|---|---|
| LOTO Station (Lockout/Tagout) | shock, arc flash — **at the source** | de-energize and verify zero-energy before work | main/breaker (source control) |
| GFCI Cart (temp-power protection) | shock | GFCI on all temporary/site power | GFCI |
| PPE Station (hard hat, glasses, gloves, arc-rated) | arc flash; reduces struck-by severity | PPE is the **last** line of defense; arc-rated for arc | (mitigation) |
| Guardrail / Fall Protection | fall | guardrails, hole covers, PFAS at height | (onboarding-specific) |
| Spotter / Flagger | struck-by | spotter for blind lifts / backing; traffic control | (onboarding-specific) |
| Barricade & Signage | struck-by, slip/trip (warn + redirect) | delineate hazard zones; communicate the hazard | warning / area effect |
| Housekeeping Crew | slip/trip | a clean site is a safe site; manage cords/debris | (onboarding-specific) |
| Fire Watch / Extinguisher | fire / hot work | fire watch during **and after** hot work | (onboarding-specific) |
| **Toolbox Talk** (support) | buff: raises nearby controls' effectiveness / lowers hazard severity | daily pre-task safety briefing — training reduces risk | grounding/bonding node (support) |
| **Competent Person / Supervisor** (support) | enables clearing; grants **stop-work authority** in range | a competent person can stop unsafe work | grounding node (enabler) |

Type-matching works exactly like the theory skin: each control only makes safe the hazard types it covers (per the skin's `match` map); the wrong control visibly does nothing. Costs mirror the theory device tiers so the gold economy is identical across skins. The two support towers (Toolbox Talk, Competent Person) parallel the theory grounding/bonding node — they don't neutralize a hazard directly, they make the other controls better, which teaches that **training and supervision underpin every physical control**.

Onboarding placement bonus: path tiles tagged `wet` (GFCI/LOTO), `height` (fall protection), or `traffic` (spotter/barricade) grant a bonus when the code-/policy-required control is placed there — the same "right control in the right place" lesson as the theory skin's code placement.

## Economy & waves (summary → [[grid-defense/ECONOMY|ECONOMY.md]])
**Questions are the gold economy.** Two wave types on a fixed cadence:

| Wave | Type | Pre-wave questions | Gold from questions |
|---|---|---|---|
| 1–4, 6–9, 11–14, … (not ÷5) | **Normal** | **2** questions | each correct = small flat gold, no penalty for wrong |
| 5, 10, 15, … (every 5th) | **Boss** | **streak of up to 3** | escalating per correct-in-a-row; **a wrong answer ends the streak** (banked gold kept) |

Gold buys and upgrades towers; a small `startGold` seats 1–2 basic towers, and prices are set so full fault coverage isn't affordable early — so answering questions is how you scale. Wrong answers never cause game-over (aligns [[SHARED-DESIGN]] §4); the cost of a miss is opportunity cost. Full wave detail, streak rules, and the tunable balance table live in **[[grid-defense/ECONOMY|ECONOMY.md]]**.

## Content upload (how questions get in)
The whole point: **a non-technical user adds questions by uploading a file — no code, no rebuild.**
- The **Menu** scene has an **"📤 Upload Questions"** control (file picker for `.csv` / `.json`, plus a drag-and-drop zone).
- On drop: detect CSV vs JSON, parse **client-side** (works from `file://`, no server), validate against the schema, and show a **summary** ("Loaded 42 questions · skipped 3 — see reasons").
- Valid banks are persisted to **`localStorage`** so they survive reloads; a **"Reset to sample questions"** control restores the bundled set.
- Uploading **replaces** the active bank (simple mental model: "the file I upload is the questions"). Reset brings back defaults.
- The build **ships a bundled sample bank** so the game is fully playable before any upload.

Full field-by-field schema, formats, validation rules, and templates: **[[grid-defense/CONTENT-SCHEMA|CONTENT-SCHEMA.md]]**.

## Depth mechanics
- **Coordination/selectivity:** reward placing the right device at the right point (a main vs a branch); a downstream device clearing before an upstream one earns a bonus.
- **Code placement (onboarding):** path tiles tagged `wet` / `dwelling`; placing the code-required device there grants a bonus and teaches the rule.

## Scene flow
`Boot → Preload` (device/fault textures) `→ Menu` (skin toggle, objective, **Upload Questions**) `→ PreWave` (ask 2 questions on normal / streak of 3 on boss → award gold → show each `why`) `→ Shop/Place` (spend gold on towers + upgrades) `→ Wave` (fault spawn + resolution loop) → back to `PreWave` for the next wave `→ GameOver / Win` (score, faults stopped, gold earned, concept recap).

Skin resolved once at Menu. Question bank loaded at Menu (from `localStorage` or bundled default), refreshed if the user uploads.

## Art notes
Top-down blueprint grid; the conductor path drawn as a bright bus. Devices = distinct primitive icons with a type glyph + range ring on hover. Faults = colored shapes carrying a type glyph (ground=green, arc=amber, surge=violet, short/overload=red) and an HP pip. Panel = a breaker-box rectangle with a health bar. **Question panel:** centered card, prompt + 3–4 choice buttons, gold-coin reward flash on correct; boss streak shows a "×N streak" pip and the rising reward. **Upload:** a simple drop-zone card on the Menu with the accepted formats and a "download template" hint.

## Design decisions & open questions
Decided (recommended defaults, all reversible):
- Uploaded file = the **question bank only**; faults/devices/waves are the game's own `CONTENT`. Questions earn the gold; they are not the fault waves.
- `skin` is an **optional** per-row field, default `both` — keeps dual-skin without forcing every author to think about it.
- Upload **replaces** the active bank; **reset** restores bundled sample.
- Correct answer identified by **matching text** (`answer` = the exact text of one choice), not a numeric index — friendlier for non-technical authors.

Open for John:
- Balance numbers (`startGold`, `normalGold`, `bossGold`, costs) — placeholders above; tune after first playable.
- Should difficulty routing be **on by default** (boss pulls harder) or ignored unless the file is fully tagged?
- Boss streak: keep-banked-on-miss (current design) vs all-or-nothing (lose the streak's gold on a miss). Current: **keep banked**.

## Acceptance
Shared baseline in [[SHARED-DESIGN]] §7, plus:
- Placement works on desktop (click) and touch (tap); type-matching is correct (wrong device visibly does nothing); waves escalate and are winnable with correct play.
- **Normal waves ask 2 questions and award flat gold per correct; boss waves ask a streak of up to 3 with escalating gold that stops on a wrong answer.**
- **A user can upload a valid CSV or JSON question file and see it drive the pre-wave questions; an invalid file is reported (rows skipped with reasons), never a crash; bundled sample plays with zero upload.**
- Uploaded bank persists across reload; "reset to sample" works.

## Rationale
Fault→device matching is a genuine electrician competency (device selection, sizing, code placement) that is *identical* to the tower-defense decision, so learning and play are the same act. Making **questions the currency** doubles down on this: the player is rewarded for electrical knowledge with the exact resource (gold) that lets them build a better defense, so studying the material directly makes you win. Driving all questions from an **uploaded file** means instructors or the ElectriAI team can retarget the game to any topic or role — pull questions from the research corpus, a new NEC chapter, or a company's onboarding — without a developer in the loop.

## Files
[[grid-defense/ECONOMY|ECONOMY.md]] (gold & waves) · [[grid-defense/CONTENT-SCHEMA|CONTENT-SCHEMA.md]] (upload schema) · templates: `templates/questions-template.csv`, `templates/questions-template.json` · parent [[learning-games]]
