# AGENTS.md

## Repository Purpose
This repository serves as a showcase for **Agent-Based Software Engineering**. It demonstrates how AI agents (LLMs) can autonomously plan, write, test, refactor, and deploy a web application based on strict prompt constraints and automated feedback loops.

The target product is a lightweight, responsive, single-player **PONG** game built with pure web technologies and deployed automatically via static hosting.

---

## 1. Technical Stack & Constraints

- **Language:** TypeScript (Strict mode enabled)
- **Bundler:** Vite
- **Rendering:** HTML5 Canvas 2D API (No heavy external game/physics engines)
- **Physics Engine:** Custom lightweight AABB (Axis-Aligned Bounding Box) vector math
- **Testing:** 
  - `Vitest` for fast unit testing (physics, collision, scoring logic, AI calculations)
  - `Playwright` for E2E and canvas mounting tests
- **CI/CD & Hosting:** GitHub Actions deploying to GitHub Pages (100% free, static output in `dist/`)

---

## 2. Target Device & Mobile Specifications

The application must be fully responsive across desktop and mobile devices, with primary optimization for the **Google Pixel 5**:

- **Viewport Size:** $1080 \times 2340$ px physical resolution ($393 \times 851$ CSS pixels @ $2.75x$ DPR).
- **Controls:**
  - **Desktop:** Keyboard Arrow keys (`Up` / `Down`) or `W` / `S`.
  - **Mobile:** Touch drag on the player side of the screen, or dedicated touch control regions.
- **Aspect Ratio Handling:** Responsive canvas scaling that locks aspect ratio (e.g., 4:3 or vertical layout depending on orientation) to prevent stretch distortion on high-DPI mobile screens.

---

## 3. Architecture & Code Quality Guidelines for Agents

To keep the project clean for agentic iteration, code must follow strict separation of concerns:
src/
├── core/           # Pure, headless game logic (100% unit-tested)
│   ├── Engine.ts   # Main loop timer & state management
│   ├── Physics.ts  # Collision detection & movement math
│   ├── Ball.ts     # Ball state & reflection logic
│   └── Paddle.ts   # Paddle movement & bounds locking
├── ai/             # Computer opponent logic
│   └── Computer.ts # Predictive AI tracking & stride interpolation
├── input/          # Input abstractions (Touch & Keyboard drivers)
├── renderers/      # Canvas drawing context routines
└── index.ts        # App entry point & DOM wiring

### Mandatory Rules for Agents
1. **Headless Logic First:** Mechanics inside `src/core/` and `src/ai/` must NOT import or rely on DOM elements (`HTMLCanvasElement`, `window`, etc.). They must accept raw numbers/vectors and return calculated states.
2. **Deterministic Physics:** Frame calculations must use delta time ($dt$) to maintain consistent movement speeds regardless of refresh rates (60Hz vs 120Hz display panels).
3. **100% Testability:** Every physics rule, paddle collision, scoring event, and AI adjustment must have a corresponding `.test.ts` file in `tests/`.
4. **Zero Heavy Dependencies:** Do not introduce Box2D, Phaser, or Matter.js unless explicitly instructed. Keep bundle sizes minimal.

---

## 4. Single-Player Computer AI Mechanics

The computer paddle controls the opposing side with humanized AI logic:

- **Stride Tracking:** Calculates the ball's incoming trajectory and moves the paddle towards the target point using a configurable speed ceiling.
- **Error Margin:** Includes an adjustable accuracy factor (e.g., variance in target position) to allow the player to score.

---

## 5. CI/CD & Deployment Instructions

Deployment is fully automated using GitHub Actions and GitHub Pages.

### Local Development Commands
- `npm run dev`: Start local Vite server
- `npm run test`: Run unit tests via Vitest
- `npm run test:e2e`: Run E2E tests via Playwright
- `npm run build`: Compile TypeScript and build static assets to `dist/`

### Automated Workflow (`.github/workflows/deploy.yml`)
1. On push to `main`:
2. Install dependencies & run linter (`npm run lint`).
3. Run unit tests (`npm run test`) and E2E tests (`npm run test:e2e`).
4. Execute `npm run build` to output static site.
5. Upload `dist/` folder via `actions/upload-pages-artifact`.
6. Deploy bundle to GitHub Pages via `actions/deploy-pages`.

---

## 6. Prompt Templates for Agent Iterations

When assigning tasks to an AI agent in this repo, structure prompts using this template:

> **Agent Task Template:**
> - **Objective:** [Clear description of feature or fix]
> - **Target Layer:** [`src/core/`, `src/ai/`, `src/input/`, etc.]
> - **Verification Requirement:** [Must include Vitest test covering X scenario]
> - **Constraint:** Ensure mobile performance and zero DOM dependencies in core logic.