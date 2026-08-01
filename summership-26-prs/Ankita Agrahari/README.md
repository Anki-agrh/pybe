# 🚀 PyBe Summership Submission — Ankita Agrahari

## PR Title
`feat: Interactive Practice Sandbox, Live Branching Flowchart Visualizer & Double-Hover Code Linking`

---

## 📌 Summary

This submission significantly elevates the PyBe learning experience by transforming the static
"view AI solution" post-reasoning flow into an **active, timed code practice environment** —
complete with a live Python control flow visualizer that links interactively with the code editor.

Rather than immediately revealing the AI mentor's solution after reasoning submission, learners
are now gated through an intentional choice: attempt to write the Python code themselves in a
timed sandbox, or display the solution immediately. This design mirrors proven pedagogical
patterns from spaced repetition and active recall research.

The submission also refactors `main.jsx` from a monolithic 800+ line file into a properly
modular, industry-standard React component tree — making the codebase production-ready and
contributor-friendly.

---

## 🎯 Motivation

The original `main.jsx` had grown to house all rendering logic, runtime state, API helpers,
parser logic, and utility functions in a single file. This made it difficult to reason about,
test, or extend. Additionally, after submitting their reasoning concept, users were shown the
AI code answer immediately — with no opportunity to attempt writing it themselves first.

---

## ✨ Features Added

### 1. 🏗️ Choice Launcher Gate
After the AI mentor returns abstraction mapping results, users see a gate card:
- **"✍️ Try Solving It Myself"** → enters the timed sandbox practice flow
- **"👁️ Display Solution Immediately"** → reveals the AI mentor solution immediately (original behaviour preserved)

### 2. ⏱️ Configurable Practice Timer
- Custom duration input (1–60 minutes)
- **Zen Mode** 🐢 — disables countdown for pressure-free practice
- Timer counts down live in the editor header with a **critical pulse animation** when < 60 seconds remain
- When countdown hits zero, solution auto-reveals with a "Challenge Ended" status banner
- "Give Up & Reveal" escape hatch always available

### 3. 💻 Interactive Python IDE Sandbox
Full-featured code editor powered by **Pyodide v0.26.2** — running Python entirely in the browser:
- Syntax-aware line-numbered gutter with dynamic row count
- `Run Code` — executes user Python instantly in-browser via WebAssembly
- `Verify` — runs scenario-specific assertions to check code correctness
- Live console output panel capturing `stdout` and `stderr`
- Progressive hint system — click "Need a Hint" to reveal contextual clues one at a time
- Globals sandbox reset between runs to prevent state bleed across executions

### 4. 📊 Live Branching Control Flow Visualizer
As the user types code in the sandbox editor, a **live flowchart panel** renders their logic
structure in real-time using a purpose-built parser:

- **Start / End** terminal nodes bookend the diagram
- **Diamond decision nodes** (`❓ Is weight > 5?`) for `if`/`elif`/`else` branches
- **Side-by-side YES / NO columns** — traditional branching flowchart layout with labeled lanes
- **Loop nodes** (`🔁 For each:`, `🔁 While:`) for iteration blocks
- **Process nodes** (`📝 weight = 10`) for assignments and function declarations
- **Output nodes** (`🖨️ Print: Heavy`) for print statements
- End node correctly exits all nested branch scopes and renders centered at the bottom

**Parser architecture in `client/src/utils/parser.js`:**
- `parsePythonLogic(code)` — line-by-line lexical scanner identifying statement types via
  indentation depth and keyword matching; each node stores a `lineNo` for hover linking
- `buildASTFlow(flatNodes)` — recursive AST nesting engine grouping conditional siblings
  into `conditional_group` tree nodes with `thenBranch` and `elseBranch` arrays

### 5. 🖱️ Double-Hover Code–Flowchart Linking
A novel UX interaction creating a **bidirectional visual link** between code editor and flowchart:
- Each parsed node stores `lineNo` — its source line index in the user's code
- Hovering a flowchart node card triggers `onHoverLine(lineNo)` propagating up to workspace state
- The matching line number in the editor **gutter highlights** (yellow-green accent, bold, glow)
- The hovered **flowchart card lifts and scales** (`transform: scale(1.03)`) with border animation
- Works for all node types: assignments, prints, loops, and diamond decision nodes

### 6. 📏 Dual Resizable Split Panels
Replaced all fixed-width CSS grid structures with custom drag-to-resize dividers:
- **Outer resizer** — drag between the reasoning form panel and AI mentor results panel (30–80%)
- **Inner resizer** — drag between code editor and flowchart visualizer panel (30–80%)
- Both support **mouse drag** (desktop) and **touch drag** (mobile/tablet)
- Event listeners cleanly removed on `mouseup` / `touchend` to prevent stale handlers

### 7. 🏆 Post-Challenge Comparison View
After timer expires or user gives up / solves the scenario:
- **Victory banner** 🏆 — displayed when user solves before time is up
- **Timeout banner** ⌛ — displayed when timer hits zero
- **Side-by-side comparison** — user's draft code on the left, AI mentor solution on the right
- AI solution explanation rendered below the comparison
- "Try Challenge Again" button resets to the locked gate state

---

## 🗂️ Files Changed

### New Files Created

| File | Purpose |
|---|---|
| `client/src/components/ChallengeWorkspace.jsx` | Self-contained practice IDE — all hooks: timers, Pyodide compiler, resize sliders, hint system, verify assertions, post-challenge reveal |
| `client/src/components/CodeFlowChart.jsx` | Recursive flowchart renderer — `FlowNodesList` renders branch columns and diamond decision nodes; accepts `hoveredLine` + `onHoverLine` props |
| `client/src/components/EmptyResult.jsx` | Stateless fallback card shown before any reasoning is submitted |
| `client/src/utils/parser.js` | `parsePythonLogic` lexical scanner + `buildASTFlow` recursive AST tree builder |
| `client/src/utils/verification.js` | Scenario-specific Pyodide assertion runners — validates user code against expected states |
| `client/src/utils/hints.js` | Static progressive hint database — maps scenario titles to ordered hint arrays |

### Modified Files

| File | Changes |
|---|---|
| `client/src/main.jsx` | Reduced from 800+ lines to ~340 lines. Removed all sandbox state, Pyodide runtime, timer hooks, flowchart logic. Now manages only: App shell, sidebar collapse, scenario filtering/search, reasoning form, API calls, dashboard layout |
| `client/src/styles.css` | Added: resizer bar styles, choice launcher cards, timer config controls, Zen Mode toggle, sandbox split layout, flowchart panel (header, body, diamond shape, branch columns, connector arrows), `gutter-active` and `flow-card-active` double-highlight classes |
| `client/index.html` | Added Pyodide CDN script tag for in-browser Python execution |

---

## 🏛️ Architecture

```
main.jsx (App Shell ~340 lines)
├── Sidebar (scenarios list, search, filters, collapse toggle)
├── ReasoningForm (concept mapping form + API call)
├── EmptyResult (fallback state)
└── ChallengeWorkspace (mounts after AI result returned)
    ├── AbstractionMap (concept pattern badges)
    ├── ChallengeLauncher (gate: try/reveal choice)
    ├── TimerConfig (duration input, Zen mode toggle)
    ├── SandboxSplitLayout (active practice state)
    │   ├── CodeEditor (gutter + textarea + actions)
    │   ├── ResizerBar (draggable divider)
    │   └── CodeFlowChart (live visualizer)
    │       └── FlowNodesList (recursive renderer)
    │           ├── ConditionalGroup (diamond + Yes/No columns)
    │           └── StatementNode (process/output/loop/end)
    └── RevealedView (side-by-side comparison + explanation)

utils/
├── parser.js       → parsePythonLogic + buildASTFlow
├── verification.js → verifyScenario (Pyodide assertions per scenario)
└── hints.js        → getScenarioHint (progressive clue lookup)
```

---

## 🧪 Testing Checklist

- [x] Choice launcher renders correctly after reasoning submission
- [x] Timer counts down and auto-reveals solution at zero
- [x] Zen Mode disables countdown and shows Zen badge in header
- [x] Run Code compiles Python via Pyodide and displays stdout in console
- [x] Verify button runs scenario assertions and marks success/failure
- [x] Flowchart renders: Start → assignment → decision diamond → Yes/No columns → End
- [x] End node exits branch scope and renders centered at the bottom
- [x] Hovering flowchart card highlights corresponding gutter line number
- [x] Both resizer bars drag correctly on mouse and touch events
- [x] Victory banner shows when user solves before timer expires
- [x] Timeout banner shows when timer hits zero
- [x] Side-by-side comparison shows user code vs AI mentor solution
- [x] "Try Again" resets workspace state cleanly
- [x] Sidebar collapse/expand works correctly
- [x] All scenario filtering and search still functional
- [x] No regressions in dashboard analytics, roadmap, or session list panels

---

## 💡 Key Design Decisions

1. **Pyodide singleton scoped to `ChallengeWorkspace`** — lazy-loads only when a user enters the
   sandbox, not on every page load, keeping initial load time fast.

2. **`buildASTFlow` is purely functional** — no side effects, takes a flat node array and returns
   a nested tree. Can be independently unit tested without any React dependency.

3. **`hoveredLine` state lives in `ChallengeWorkspace`** and is passed as props to both the gutter
   renderer and `CodeFlowChart`, keeping data flow strictly unidirectional (React best practice).

4. **`lineNo` stored on every parser node** — enables the double-hover linking without any DOM
   querying, making the link purely data-driven and React-idiomatic.
