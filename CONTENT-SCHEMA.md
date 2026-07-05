---
type: design
domain: Research
updated: 2026-07-01
---

# Grid Defense — Question Upload Schema

The contract a user's **CSV or JSON** question file must satisfy. The goal is that a **non-technical user changes or adds all questions by uploading one file** — no code edits, no rebuild. This document defines the fields, the two file formats, validation rules, how questions map to gameplay, and the upload flow. Companion to [[grid-defense/DESIGN|DESIGN.md]]; conforms to [[SHARED-DESIGN]].

Copy-and-fill templates live in `templates/questions-template.csv` and `templates/questions-template.json`.

## 1. What one row/object is
One row (CSV) or one object (JSON) = **one multiple-choice question** used in a pre-wave phase to earn gold. Minimum viable question: a prompt, the choices, and which choice is correct.

## 2. Field reference
| Field | Required | Type | Rules | Purpose |
|---|---|---|---|---|
| `question` | **yes** | text | non-empty | the prompt shown to the player |
| `choices` | **yes** | list | 2–6 options; CSV: `A\|B\|C\|D` (pipe-separated in one cell); JSON: array of strings | the answer buttons |
| `answer` | **yes** | text | must **exactly match one entry in `choices`** (case- and whitespace-insensitive) | marks the correct choice |
| `why` | no | text | shown after the player answers | one-line explanation that reinforces the concept |
| `difficulty` | no | enum | `easy` \| `medium` \| `hard` (default `medium`) | routes questions: normal waves prefer easy/medium, boss waves prefer medium/hard |
| `skin` | no | enum | `theory` \| `onboarding` \| `both` (default `both`) | which audience skin the question appears in |
| `category` | no | text | free text, or an ElectriAI schema code (`OCP`, `GBF`, `PDS`, …) | tagging/analytics; not required to play |
| `id` | no | text | unique if provided; auto-generated if absent | stable identifier for a question |

**Only `question`, `choices`, and `answer` are required.** Everything else has a sensible default, so the simplest valid file is three columns.

## 3. Correct-answer rule (important)
`answer` is the **text of the correct choice**, not a number. The loader matches it to `choices` after trimming whitespace and ignoring case. This is deliberate — non-technical authors shouldn't have to count from zero or track an index. If `answer` doesn't match any choice, the row is **skipped** and reported.

## 4. CSV format
- **UTF-8**, comma-separated, with a **header row**. Column names are **case-insensitive** and **order-independent** — the loader maps by header name.
- **`choices` uses `|` (pipe) inside a single cell**, e.g. `GFCI|AFCI|SPD|Fuse`. Pipe is used (not comma) so choices never collide with CSV's comma delimiter.
- Any field containing a comma must be wrapped in double quotes (standard CSV), e.g. a scenario prompt.
- Blank optional cells are fine — they fall back to defaults.

Example (`.csv`):
```csv
question,choices,answer,why,difficulty,skin,category
"A device that trips only on current leaking to ground is a…",GFCI|AFCI|SPD|Fuse,GFCI,"GFCI protects people from ground faults",easy,theory,GBF
"A bathroom receptacle requires which device?",GFCI|AFCI|Fuse,GFCI,"NEC requires GFCI at wet locations",easy,onboarding,GBF
"Bedroom branch circuits require…",AFCI|GFCI|SPD,AFCI,"AFCI detects arcing in dwelling circuits",medium,onboarding,OCP
"Which device is single-use and must be replaced after it operates?",Fuse|Circuit breaker|GFCI,Fuse,"A fuse blows once; a breaker resets",easy,both,OCP
"Nuisance-tripping a 15 A breaker on a heavy load is best fixed by…",Correct sizing / dedicated circuit|Bigger breaker only|Removing the breaker,Correct sizing / dedicated circuit,"Never upsize past the conductor's rating",hard,both,OCP
```

## 5. JSON format
- A top-level **array of objects**, or `{ "questions": [ … ] }` — both accepted; the array is canonical.
- `choices` is a real **array of strings**. Other fields match the table above.

Example (`.json`):
```json
[
  {
    "question": "A device that trips only on current leaking to ground is a…",
    "choices": ["GFCI", "AFCI", "SPD", "Fuse"],
    "answer": "GFCI",
    "why": "GFCI protects people from ground faults.",
    "difficulty": "easy",
    "skin": "theory",
    "category": "GBF"
  },
  {
    "question": "Which device is single-use and must be replaced after it operates?",
    "choices": ["Fuse", "Circuit breaker", "GFCI"],
    "answer": "Fuse",
    "why": "A fuse blows once; a breaker resets.",
    "difficulty": "easy"
  }
]
```

## 6. Validation & error handling
The loader is **lenient per row, strict per field** — a bad row is skipped, not fatal:
- **Row skipped** if: `question` empty; fewer than 2 or more than 6 `choices`; two choices have the same text (after trimming/case-folding — duplicates would make grading ambiguous); `answer` doesn't match any choice; `difficulty`/`skin` has an invalid (non-empty, non-enum) value.
- An **explicitly wrong** enum value (e.g. `difficulty: hardd`) is reported and the row skipped, so typos surface instead of silently mis-tagging. A **blank** optional field is not an error — it takes the default.
- After parsing, show a **summary**: `Loaded N questions · skipped M`, with the skipped rows and reasons listed.
- **Minimum to start: 5 valid questions.** Below that, the game keeps the **current** bank (the bundled sample if nothing was uploaded before) and warns. The summary also reports per-skin counts and warns when one skin's pool is small (<5).
- A malformed file (not parseable as CSV/JSON) is reported cleanly — never a crash or a silent empty game.

## 7. How questions map to gameplay
See [[grid-defense/DESIGN|DESIGN.md]] for the full economy. In short:
- **Normal wave (waves not divisible by 5):** 2 questions, each correct = flat gold.
- **Boss wave (every 5th):** streak of up to 3, escalating gold per correct-in-a-row, a wrong answer ends the streak (banked gold kept).
- **`difficulty`** (if present) routes which questions each wave type pulls; **`skin`** filters by the active audience; **`why`** is shown after answering to reinforce the concept.
- Questions are drawn without repeats until the pool is exhausted, then reshuffled.

## 8. Upload flow
1. **Menu → "📤 Upload Questions"** (file picker accepts `.csv` / `.json`) or drag a file onto the drop-zone.
2. File is read and parsed **client-side** (no server; works from `file://`). CSV via an inline parser that handles quotes and the `|` choice delimiter; JSON via `JSON.parse`.
3. Validate → show the summary panel.
4. On accept, the bank is stored in **`localStorage`** (key e.g. `griddef.questions.v1`, with source filename + count + timestamp) and becomes the active bank.
5. **"Reset to sample questions"** clears the stored bank and restores the bundled default.
6. Uploading a new file **replaces** the current bank.

## 9. Authoring tips (common mistakes)
- Put the correct answer's **exact text** in `answer` — copy it from `choices`.
- In CSV, separate choices with **`|`**, not commas.
- Quote any prompt that contains a comma.
- Keep 3–4 choices; 2 is allowed but easy; more than 6 is rejected; don't repeat the same choice text twice in one question.
- Leave `difficulty`/`skin`/`category` blank if you don't need them — defaults apply.
- Save CSV as **UTF-8** (Excel: "CSV UTF-8").

## Parent
[[grid-defense/DESIGN|DESIGN.md]] · [[learning-games]] · [[SHARED-DESIGN]]
