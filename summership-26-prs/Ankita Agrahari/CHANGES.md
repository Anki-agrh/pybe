# Feature Walkthrough — Ankita Agrahari

## Overview

This submission extends the base PyBe prototype by adding an active-learning practice path.
After the AI mentor returns abstraction mapping results, learners are no longer immediately
shown the model solution. Instead they pass through a choice gate and can optionally write
the solution themselves in a fully in-browser Python IDE, with a live flowchart that updates
as they type.

---

## 1. Choice Launcher Gate

After reasoning submission, a gate card replaces the immediate code reveal:

- **"✍️ Try Solving It Myself"** → enters the timed sandbox practice flow
- **"👁️ Display Solution Immediately"** → reveals the AI mentor solution immediately (original behaviour preserved)

This preserves full backward compatibility while adding a new active-learning path.

---

## 2. Configurable Practice Timer

Before the sandbox opens, learners configure their session:

- Custom duration input (1–60 minutes)
- **Zen Mode 🐢** — disables the countdown entirely for pressure-free practice
- Timer counts down live in the editor header with a critical pulse animation when < 60 seconds remain
- When countdown hits zero, solution auto-reveals with a "Challenge Ended" banner
- "Give Up & Reveal" escape hatch always available

---

## 3. Interactive Python IDE Sandbox

Full-featured code editor powered by **Pyodide v0.26.2** — Python runs entirely in the browser via WebAssembly:

- Syntax-aware line-numbered gutter with dynamic row count
- **Run Code** — executes user Python instantly in-browser, no server round-trip
- **Verify** — runs scenario-specific assertions to check correctness
- Live console output panel captures `stdout` and `stderr`
- Progressive hint system — click "Need a Hint" to reveal contextual clues one at a time
- Globals sandbox reset between runs to prevent state bleed across executions

---

## 4. Live Branching Control Flow Visualizer

As the user types code, a live flowchart panel renders the logic structure in real-time:

| Node Type | Appearance | Triggers |
|---|---|---|
| Start / End | Rounded terminal | Top and bottom of every diagram |
| Decision | Diamond `❓` | `if`, `elif`, `else` |
| Process | Rectangle `📝` | Assignments, function defs |
| Output | Rectangle `🖨️` | `print(...)` calls |
| Loop | Rectangle `🔁` | `for`, `while` |

- **Side-by-side YES / NO columns** — traditional branching flowchart layout
- End node correctly exits all nested branch scopes and aligns centered at the bottom

**Parser architecture (`client/src/utils/parser.js`):**
- `parsePythonLogic(code)` — line-by-line lexical scanner; detects statement types via indentation depth and keyword matching; each node stores a `lineNo` for hover linking
- `buildASTFlow(flatNodes)` — recursive AST nesting engine; groups conditional siblings into `conditional_group` tree nodes with `thenBranch` and `elseBranch` arrays

---

## 5. Double-Hover Code–Flowchart Linking

A bidirectional visual link between the code editor and the live flowchart:

- Each parsed node stores `lineNo` — its source line index in the user's code
- Hovering a flowchart node card triggers `onHoverLine(lineNo)` which propagates up to workspace state
- The matching gutter line number **highlights** in yellow-green with a subtle glow
- The hovered **flowchart card lifts and scales** (`transform: scale(1.03)`) with a border animation
- Works for all node types: assignments, prints, loops, and diamond decision nodes

---

## 6. Dual Resizable Split Panels

Two custom drag-to-resize dividers replace all fixed-width CSS grid structures:

- **Outer resizer** — between the reasoning form panel and the AI mentor results panel (30–80% bounds)
- **Inner resizer** — between the code editor and the flowchart visualizer panel (30–80% bounds)
- Both support mouse drag (desktop) and touch drag (mobile/tablet)
- Event listeners cleanly removed on `mouseup` / `touchend` to prevent stale handlers

---

## 7. Post-Challenge Comparison View

After the timer expires or the user gives up / solves the scenario:

- **Victory banner 🏆** — displayed when user solves before time is up
- **Timeout banner ⌛** — displayed when timer hits zero
- **Side-by-side comparison** — user's draft code (left) vs AI mentor model solution (right)
- AI solution explanation rendered below the comparison
- "Try Challenge Again" resets workspace state cleanly

---

## Files Changed

### New Files

| File | Purpose |
|---|---|
| `client/src/components/ChallengeWorkspace.jsx` | Self-contained practice IDE — timers, Pyodide compiler, resize sliders, hint system, verify assertions, post-challenge reveal |
| `client/src/components/CodeFlowChart.jsx` | Recursive flowchart renderer — `FlowNodesList` renders branch columns and diamond decision nodes; accepts `hoveredLine` + `onHoverLine` props |
| `client/src/components/EmptyResult.jsx` | Stateless fallback card shown before any reasoning is submitted |
| `client/src/utils/parser.js` | `parsePythonLogic` lexical scanner + `buildASTFlow` recursive AST tree builder |
| `client/src/utils/verification.js` | Scenario-specific Pyodide assertion runners |
| `client/src/utils/hints.js` | Static progressive hint database |

### Modified Files

| File | Changes |
|---|---|
| `client/src/main.jsx` | Reduced from 800+ lines to ~340 lines. Removed all sandbox state, Pyodide runtime, timer hooks, flowchart logic. Now manages only App shell, sidebar, reasoning form, API calls, and dashboard layout |
| `client/src/styles.css` | Added resizer bar styles, choice launcher cards, timer config controls, Zen Mode toggle, sandbox split layout, flowchart panel (header, body, diamond shape, branch columns, connector arrows), `gutter-active` and `flow-card-active` double-highlight classes |
| `client/index.html` | Added Pyodide CDN script tag for in-browser Python execution |

---

## Architecture

```
main.jsx (App Shell ~340 lines)
├── Sidebar
├── ReasoningForm
├── EmptyResult
└── ChallengeWorkspace
    ├── ChallengeLauncher (gate: try/reveal)
    ├── TimerConfig (duration + Zen Mode)
    ├── SandboxSplitLayout
    │   ├── CodeEditor (gutter + textarea + Run/Verify/Hint)
    │   ├── ResizerBar
    │   └── CodeFlowChart → FlowNodesList (recursive)
    │       ├── ConditionalGroup (diamond + Yes/No columns)
    │       └── StatementNode (process/output/loop/end)
    └── RevealedView (comparison + explanation)

utils/
├── parser.js       → parsePythonLogic + buildASTFlow
├── verification.js → verifyScenario (Pyodide assertions per scenario)
└── hints.js        → getScenarioHint (progressive clue lookup)
```
