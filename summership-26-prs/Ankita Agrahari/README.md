# PyBe — Interactive Practice Sandbox & Live Flowchart Visualizer

PyBe is a scenario-driven Python learning prototype. This submission extends the base prototype
with an active-learning practice environment: instead of immediately revealing the AI mentor's
solution, learners can choose to write the Python code themselves inside a timed in-browser IDE,
watch a live control flow diagram update as they type, and compare their solution with the model
answer at the end.

> View the full feature walkthrough: [CHANGES.md](CHANGES.md)

## What's New in This Submission

- **Choice Launcher Gate** — after reasoning submission, learners choose between attempting the code themselves or viewing the solution immediately
- **Configurable Practice Timer** — custom countdown (1–60 min) or Zen Mode (no pressure)
- **In-Browser Python IDE** — powered by Pyodide (WebAssembly), with Run, Verify, and progressive Hints
- **Live Branching Flowchart** — renders decision diamonds, YES/NO columns, loops, and output nodes in real-time as the user types
- **Double-Hover Code–Flowchart Linking** — hovering a flowchart node highlights the matching line in the editor gutter, and vice versa
- **Dual Resizable Split Panels** — drag-to-resize dividers between the reasoning panel and results, and between the editor and flowchart
- **Post-Challenge Comparison View** — side-by-side user code vs AI mentor solution with victory / timeout banners

## Features (Base)

- Scenario browser with difficulty, concept, and search filters
- Interactive learning session: learner reasoning, abstraction mapping, Python construct generation, prompt scoring
- Dashboard with progress, concept mastery, misconceptions, and recent sessions
- Roadmap view covering V0 through V3
- JSON-file backed API with seed data

## Tech Stack

- React + Vite (frontend)
- Node.js + Express (backend)
- Pyodide v0.26.2 (in-browser Python via WebAssembly)
- Plain CSS, no auth, no external database
- JSON file storage (`server/src/data/db.json`)

## Prerequisites

- Node.js 18+

## Setup

1. Install dependencies:

```bash
npm run installAll
```

2. Configure the server environment:

```bash
cp server/.env.example server/.env
```

3. Seed sample data:

```bash
npm run seed
```

4. Run the app:

```bash
npm run dev
```

- Frontend: http://localhost:5173
- API: http://localhost:5000/api

## Project Structure

```
client/
├── src/
│   ├── components/
│   │   ├── ChallengeWorkspace.jsx   ← NEW: timed sandbox IDE + timer + hints + verify
│   │   ├── CodeFlowChart.jsx        ← NEW: recursive live flowchart renderer
│   │   └── EmptyResult.jsx          ← NEW: fallback empty state card
│   ├── utils/
│   │   ├── parser.js                ← NEW: Python AST lexer + tree builder
│   │   ├── verification.js          ← NEW: scenario-specific Pyodide assertions
│   │   └── hints.js                 ← NEW: progressive hint database
│   ├── main.jsx                     ← MODIFIED: cleaned up, modular (800→340 lines)
│   └── styles.css                   ← MODIFIED: flowchart, sandbox, resizer styles
server/
└── src/
    └── index.js                     ← MODIFIED: minor route cleanup
```

## Notes

The AI behavior in this prototype is deterministic and local — no OpenAI keys required.
Python code execution runs entirely in the browser via Pyodide WebAssembly, so no backend
sandbox server is needed. Learning data is stored in `server/src/data/db.json`.
