# Lecture Notes Organizer

A Claude skill that turns messy lecture notes, transcripts, or slide text into clean, consistently formatted study notes for exam prep. It keeps the instructor's exact terminology and flags anything unclear, missing, contradictory, or possibly wrong instead of guessing.

Built for college students preparing for exams.

## Why this exists

Raw lecture notes are hard to study from. They're full of shorthand, half-finished thoughts, undefined abbreviations, and the occasional misheard fact. A general "summarize my notes" request often makes this worse: an AI may quietly reword definitions, fill gaps from general knowledge, or "fix" errors without telling you. On exam day, you're graded against your professor's wording, not a textbook's.

This skill is built on three principles:

1. **Consistency.** Every set of notes has the same seven sections in the same order, however you phrase the request. Notes from different lectures line up, so gaps are easy to spot.
2. **Fidelity to the source.** Terms and definitions come from your material, word for word. Nothing from outside knowledge gets mixed into the notes.
3. **Honest flagging.** Problems are marked, not silently fixed, so you know exactly what to check with your professor, textbook, or a classmate before the exam.

## What the output looks like

Every set of study notes follows this structure:

| Section | What it contains |
|---|---|
| **Overview** | 2–4 sentences on what the lecture covers and how the ideas connect |
| **Key Terms** | A table of every important term, copied exactly from the source, with its definition |
| **Main Concepts** | Numbered subsections in lecture order, with key points as bullets and exam hints highlighted |
| **Formulas & Equations** | Each formula as written, with what each symbol means |
| **Examples & Applications** | Examples from the lecture and the concept each one illustrates |
| **Flagged Items** | A table of every problem found, where it is, and what to check |
| **Review Questions** | 3–6 questions answerable from the notes, for active recall practice |

Sections with no content say `None in source.` so you know nothing was skipped by accident.

### Flag types

Problems are marked inline in the notes (e.g. `[F1]`) and explained in the Flagged Items table using one of five labels:

| Type | Used when |
|---|---|
| `UNCLEAR` | Wording is ambiguous, garbled, or marked with "??" |
| `INCOMPLETE` | A thought trails off, a list is cut short, or "missed this" appears |
| `CONFLICT` | Two parts of the source contradict each other |
| `POSSIBLE ERROR` | A statement contradicts well-established knowledge in the field |
| `UNDEFINED TERM` | A term, abbreviation, or symbol is used but never defined |

For example, if your notes say "net gain of 4 ATP from glycolysis," the notes keep that wording and flag it as a `POSSIBLE ERROR`, noting that standard texts give 2 ATP net. You might have misheard, or your professor might be using a specific convention, so you decide.

## Repository structure

```
lecture-notes-organizer/
├── SKILL.md                         # Skill definition: when to trigger + step-by-step instructions
├── assets/
│   └── study-notes-template.md      # The exact output skeleton
├── references/
│   └── example.md                   # Worked example: messy input → finished notes
└── scripts/
    └── validate_notes.py            # Checks every output for format and accuracy rules
```

- **`SKILL.md`** has the frontmatter `name` and `description` that tell Claude when to use the skill, followed by six steps for producing the notes.
- **`assets/study-notes-template.md`** is the fill-in-the-blanks structure Claude follows.
- **`references/example.md`** shows a full psychology lecture example, including flags, so Claude can match the expected level of detail.
- **`scripts/validate_notes.py`** is a validator Claude runs on its own output before delivering it, and fixes anything it reports.

## Installation

**Claude.ai:** download `lecture-notes-organizer.skill` (a zip of this folder) and upload it through the skills section of Claude's settings. If your organization allows skill creation, opening the `.skill` file in a Claude chat also shows a **Save skill** button.

**Claude Code:** copy the `lecture-notes-organizer` folder into `~/.claude/skills/` (for all projects) or `.claude/skills/` inside a project.

To rebuild the `.skill` file from this repo, zip the folder:

```bash
zip -r lecture-notes-organizer.skill lecture-notes-organizer
```

## Usage

Once installed, you don't need to name the skill. Paste or upload your notes and ask in your own words:

- "midterm is thursday can u turn my bio notes into something i can study from"
- "Organize the following lecture transcript."
- "Make a study guide from these slides." (with a PDF attached)
- "fix these notes, they're a mess"

Claude produces a Markdown file named `study-notes-<topic>.md` and tells you how many items were flagged and which one matters most before the exam.

## The validator

`validate_notes.py` can also be run by hand on any notes file:

```bash
python scripts/validate_notes.py study-notes-cellular-respiration.md --source my-notes.txt
```

It checks that:

- all seven sections are present, in the correct order;
- every Key Term appears word-for-word in the source (if `--source` is given), which catches swapped-in synonyms;
- flags are numbered F1, F2, … with no gaps and use one of the five allowed types;
- every inline `[F#]` marker has a matching table row, and every row has a marker;
- there are 3–6 review questions;
- tables aren't broken by stray `|` characters, such as absolute-value bars in math notation.

It prints `PASS` or `FAIL` with a list of specific errors. It needs only Python 3 and has no extra dependencies.

## Testing

The skill was tested on two deliberately different inputs, each with problems planted in it, using differently worded requests to confirm the output doesn't depend on how the user phrases things.

| Test | Input style | Request | Format check | Content checks |
|---|---|---|---|---|
| Biology: cellular respiration | Shorthand student notes | "midterm is thursday can u turn my bio notes into something i can actually study from" | PASS | 6/6 |
| Economics: price elasticity of demand | Spoken lecture transcript with filler and admin announcements | "Organize the following lecture transcript." | PASS | 6/6 |

Both runs produced the same seven-section structure. The planted problems were caught in both:

- **Biology:** a wrong ATP count (flagged, not silently corrected), a contradiction about where the electron transport chain is located, a gap where the student "missed the rest," and undefined abbreviations.
- **Economics:** reversed definitions of elastic and inelastic demand, a formula the professor said would be "on the slides," and undefined terms (XED, YED). Filler words and the homework reminder were removed, and the professor's "this will be on the midterm" hint was kept.

Testing also uncovered a bug: absolute-value notation like `|PED|` broke the Markdown tables. The skill and validator were updated to handle it, and all tests were re-run.

## Limitations

- The skill organizes and flags. It does not research or fill gaps, so incomplete notes produce notes with visible gaps.
- `POSSIBLE ERROR` flags depend on Claude's general knowledge and may occasionally miss an error or flag something that is correct in your course's context. Treat them as prompts to double-check, not verdicts.
- Testing so far has covered two subjects. Results on very long multi-lecture inputs or highly specialized fields haven't been formally tested.
