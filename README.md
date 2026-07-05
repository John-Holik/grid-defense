# Grid Defense

Tower-defense learning game where the player stops electrical **faults** by placing the right **protective device**. Correct quiz answers are the gold economy: 2 questions per normal wave, an escalating streak of up to 3 on boss waves.

Single self-contained HTML file (Phaser 3 via CDN, all textures generated at runtime). No build step, no assets.

## Play

Open `grid-defense.html` in a browser. `?skin=onboarding` opens the construction-safety **Safe Site** skin (LOTO, GFCI, PPE, fall protection) instead of the electrical-theory skin.

- 15-wave campaign or endless mode, 4 unlockable maps, boss every 5th wave
- Player profile with XP/levels/ranks, per-category accuracy, run save, question mastery (Quizlet-style: 2 correct in a row, misses raise the bar)
- In-game field guide, tile tooltips, and upgrade hints

## Custom questions

Questions are **uploaded, not coded**: CSV or JSON from the menu (persisted to `localStorage`). Schema in `CONTENT-SCHEMA.md`; starter files in `templates/`. A 49-question sample bank is bundled.

`instructor.html` is a companion authoring tool: create/edit/import/export questions with validation and in-game-style preview, then install the bank directly into the game.

## Docs

- `DESIGN.md` — game design and question-driven economy
- `CONTENT-SCHEMA.md` — question file format
- `ECONOMY.md` — balance model
