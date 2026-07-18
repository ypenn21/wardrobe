# Repository Guidelines

## Project Structure & Module Organization

Wardrobe is a local-first React application built with Vite. Client code lives in `src/`: `App.jsx` owns the gallery, `import-flow.jsx` implements the import wizard, and `OptimizedImage.jsx` wraps responsive image delivery. Keep component-specific styles beside that feature (`import-flow.css`); shared styling belongs in `src/styles.css`.

Node-side Vite middleware is under `scripts/`, including import-job APIs and the `/_ipx` image pipeline. Static assets live in `public/`, while documentation screenshots live in `docs/screenshots/`. Local wardrobe records and generated images belong in ignored `data/`, never source control. No dedicated test directory currently exists.

## Agent Orchestration

### Architect–Engineer Split
This repository uses a clear architect–engineer division of labor to coordinate tasks:
- **Codex/Sol (Architect & Final Reviewer):** Owns requirements, task decomposition, overall architecture, security-sensitive decisions, acceptance criteria, and final verification.
- **Gemini CLI (Default Engineer):** Handles routine exploration, codebase searches, implementation, refactoring, writing tests, fixing builds, updating documentation, and other groundwork.
- **Direct Implementation Criteria:** Direct implementation is reserved for the root Codex/Sol model when the primary challenge is deep technical reasoning (e.g., cross-cutting architecture, ambiguous system design, subtle concurrency or security issues, complex debugging after Gemini stalls, or high-risk changes).

### Delegating to Gemini
For most routine engineering tasks, invoke Gemini CLI from the repository root in headless mode. Prefer the custom agent `gemini_engineer` when Codex subagent orchestration is available; otherwise call Gemini directly.

Before delegating, inspect `.codex/agents/`. This repository defines the `gemini_engineer` custom agent in `.codex/agents/gemini-engineer.toml`; use its `name` field to select the agent. Invoke Gemini CLI directly only if the custom-agent spawn fails or the current Codex surface cannot select it, and explicitly report that fallback.

For implementation/modification tasks, use:
```sh
gemini --approval-mode auto_edit --output-format json -p "<bounded task prompt>"
```

For analysis or planning that must not modify files, use:
```sh
gemini --approval-mode plan --output-format json -p "<bounded task prompt>"
```

Every delegation prompt must explicitly include:
- The concrete objective and relevant file scope.
- Constraints and acceptance criteria.
- Required validation commands.
- An instruction to preserve unrelated user changes and never commit.
- An instruction to return a concise summary of changed files, validation results, and unresolved risks.

### Constraints & Operational Protocols
- **No Overrides:** Do not use `--yolo`. Never place secrets or API keys in prompts or command outputs. Gemini must strictly adhere to the technical conventions defined in `GEMINI.md`.
- **Review & Takeover:** The architect must inspect Gemini's diff and run or independently confirm all appropriate validation checks before completing a task. If Gemini fails, diagnose the failure, refine the prompt, and retry once if useful. The architect should take over implementation when remaining work requires deep reasoning or a second delegation is inefficient.
- **Delegation Exceptions:** Avoid delegating trivial conversational answers, simple obvious edits that are faster to make safely, tasks requiring a specialized Codex skill, or operations requiring user interaction or approval before starting.

## Build, Test, and Development Commands

- `npm install` installs the locked dependencies.
- `npm run dev` starts the Vite development server at `http://localhost:5173`.
- `npm run build` creates the production bundle in `dist/`.
- `npm run preview` serves that bundle locally on port 4173.
- `npm run check` performs the required validation; currently it runs the production build.

Run `npm run check` before opening a pull request.

## Coding Style & Naming Conventions

Use native ECMAScript modules and include file extensions in local imports. Follow the existing two-space indentation, semicolons, and double-quoted JavaScript strings. Name React components in PascalCase (`OptimizedImage`) and functions or variables in camelCase. Keep filenames aligned with existing patterns: PascalCase for standalone components and lowercase kebab-case for feature modules. Use vanilla CSS and existing design tokens; do not introduce Tailwind. Render application images through `OptimizedImage` instead of raw `<img>` elements.

## Testing Guidelines

There is no automated test framework or coverage threshold yet. Validate every change with `npm run check` and manually exercise affected gallery, import, or image-serving flows. If adding tests, colocate them with the feature using `*.test.jsx` or `*.test.mjs` and document any new command in `package.json`.

## Commit & Pull Request Guidelines

Recent commits use short, imperative, sentence-case subjects such as `Add modeled outfit generation skills`. Keep commits focused on one concern. Do not stage or commit changes unless explicitly requested by the user. Pull requests should explain behavior changes, list validation performed, link relevant issues, and include screenshots for visible UI changes.

## Security & Local Data

Never commit `.env`, API keys, `data/`, personal photos, or generated wardrobe assets. Use `.env.example` for documented configuration names without real credentials.
