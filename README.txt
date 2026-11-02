# SLR(1) Parser, Step by Step

**Compiler Design Lab · Slot L37 + L38**
Ragunandhan T (24BCE0721) · Suyambulingam (24BCE0726) · Guruprasad V (24BCE0761)

A browser app that builds an SLR(1) parser for any context-free grammar **one visible step at a time**:
FIRST/FOLLOW sets, the LR(0) item sets, the ACTION/GOTO table, conflict detection and the parse of an
input string. Every step comes with the reason for it. Students can watch the construction (Teach),
practise it with instant marking (Practice), or attempt it as a timed test (Test).

It runs entirely in the browser: no server-side code, no database, no build tools.

---

## Installation

### Just use it (no installation)
1. Download or clone this repository.
2. Double-click **`SLR-Parser.html`**. It opens in any modern browser (Chrome, Edge, Firefox).

That's it. The whole app is in this one file and works offline. Only the fonts need an internet
connection; without one the browser falls back to its default fonts.

### For development (editing the code)
You need:
- **A browser**
- **[Node.js](https://nodejs.org/) 14 or newer**: to run the tests and rebuild `SLR-Parser.html`.
  There are no packages to install (`npm install` is not needed).
- **Python 3** (optional): only if you want to serve the files locally.

Open the source version of the app in either of these ways:

```bash
# Option 1: open the file directly
#   source/final/index.html

# Option 2: serve it locally, then open http://localhost:5173/final/index.html
python -m http.server 5173 --directory source
```

---

## How to use it

### 1. Enter a grammar and an input string
- Type one production per row, e.g. `S -> AA` and `A -> aA | b`. A new row appears as you type.
- Type the input string to parse, e.g. `aabb`, or tap the terminal buttons to build it.
- Press **Build parser**, or pick a grammar from **Load a grammar…** (examples from the Module 2 notes).
- **Text mode** lets you paste a whole grammar at once.

### 2. Pick a mode (buttons in the top bar)

| Mode | What it does |
|---|---|
| **Teach** | The worksheet fills itself in one step at a time, with the reason for each step written beside it. **Next step / Back** move one step, **Next section ⏭** jumps to the next part, **▶ Play** steps automatically. The ← and → keys also work. |
| **Practice** | Solve the whole worksheet yourself. Press **Check** on any part: entries you filled in are marked green or red, and empty ones count as "not filled yet". Under each wrong answer the correct entry is hidden; click it to **peek** (a peeked answer is not counted). **Show answer** fills in a whole part. |
| **Test** | The same worksheet, as a timed test. Press **Begin test** to unlock the answer boxes and start the stopwatch (there is no time limit; it only records how long you take). Teach and Practice are locked until you finish. **Submit test** marks everything at once and shows your score, time and **best result** for that grammar. **Print / Save as PDF** prints your marked attempt plus the full worked solution. |

The **☾ Dark / ☀ Light** button switches the theme. Your choice is remembered.

### 3. What the worksheet covers
1. Augmented grammar (production `(0) S' → S`)
2. FIRST and FOLLOW sets
3. Canonical collection of LR(0) items
4. SLR(1) parsing table (ACTION and GOTO)
5. Is the grammar SLR(1)? (conflicts are listed, never resolved silently)
6. Parse of the input string: stack, remaining input and action at every move

### Grammar notation

```
E -> E + T | T          arrows: ->  -->  →  ::=
A -> aA | b             compact form: aA is read as a A
Y -> bZ | ε             ε, eps, epsilon, # or an empty alternative
S -> 'if' C 'then' S    quotes keep a multi-letter terminal whole
```

The first rule's left side is the start symbol. `$` is reserved for end of input. Notation copied
from PDFs (`⟶ ⇒`, `∈ ϵ` for ε, curly primes) is cleaned automatically.

---

## Development workflow

1. **Edit** the source: `source/final/index.html` (the page) or `source/js/*.js` (the engine).
2. **Test** the engine:
   ```bash
   node source/tests/run-tests.js
   ```
   53 tests. Expected values come from the Module 2 lecture notes and the Dragon Book tables,
   not from the program's own output.
3. **Check it in the browser**: open `source/final/index.html` (see Installation).
4. **Rebuild the single-file app**: this copies every `source/js/*.js` file into the page:
   ```bash
   node source/bundle.js
   ```
   This regenerates `SLR-Parser.html`. Run it after every change, or the single file will be out of date.

### Feature switches
Some features can be turned off without deleting code. They are at the top of the main `<script>`
in `source/final/index.html`. Set one to `false`, then run `node source/bundle.js`.

| Switch | When `true` | When `false` |
|---|---|---|
| `PEEK` | Wrong Practice answers get a hidden correct entry you can click to reveal | Plain red marking only |
| `PARTIAL` | Practice marks only what you entered; empty answers count as "not filled yet" | Every empty answer is marked wrong |
| `TEACH_NAV` | Teach mode has **Next section** and **Play** | Only Back / Next step |

Copies of the page from before each of these features was added are kept in `backups/`.

---

## Project structure

```
SLR-Parser.html          the whole app in one file (built by source/bundle.js)
source/
├── final/index.html     the page: worksheet, Teach / Practice / Test modes, theme
├── js/
│   ├── grammar.js       grammar text → productions; input string → tokens
│   ├── slr.js           the engine: FIRST/FOLLOW, closure/goto, LR(0) items, table, conflicts, parse
│   ├── practice.js      answer checking and Test-mode best results (no page code; tested in Node)
│   ├── core.js          "what does stage X look like at step i?" for the page
│   ├── lesson.js        turns the engine's steps into one list that Teach mode walks through
│   ├── rows.js          the production-by-production grammar editor
│   ├── spoiler.js       the speck animation that hides peekable answers
│   └── examples.js      example grammars from the Module 2 notes
├── tests/run-tests.js   automated tests (Node, no dependencies)
└── bundle.js            builds SLR-Parser.html
docs/
├── DESIGN.md            architecture, data flow and algorithms
└── VIVA_QUESTIONS.md    likely viva questions with answers
backups/                 earlier versions of source/final/index.html
templates/               UI templates collected for future design changes
```

`app.js`, `viz.js`, `practice-ui.js` and `player.js` in `source/js/` are from an earlier version of the
interface and are not used by the current page.

### How it works, in one sentence
The engine (`grammar.js` + `slr.js`) builds the parser and **records every step it takes**; the page
(`final/index.html`) only replays those steps (Teach) or checks the student's answers against them
(Practice and Test).

---

## Limitations
- SLR(1) only. LR(0) is reported as a yes/no check; CLR(1) and LALR(1) are not built.
- When a grammar has conflicts, the parse uses the first entry in the conflicting cell and marks that move.
- The parser stops at the first syntax error (no error recovery).
- Best Test results are saved in the browser, so they are per device and per browser.
