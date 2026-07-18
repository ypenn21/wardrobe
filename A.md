# Agent Operating Model

This repository uses an architect–engineer split:

- The root Codex agent is the architect and final reviewer. It owns requirements,
  task decomposition, architecture, security-sensitive decisions, acceptance
  criteria, and final verification.
- Gemini CLI is the default engineer. Delegate routine exploration, codebase
  searches, implementation, refactors, test writing, build fixes, documentation,
  and other groundwork to Gemini CLI.
- Reserve direct implementation by the root Codex/Sol model for work whose main
  difficulty is deep technical reasoning: cross-cutting architecture, ambiguous
  system design, subtle concurrency or security issues, hard debugging after
  Gemini stalls, or changes where delegation would materially increase risk.

## Delegating to Gemini

For most engineering tasks, invoke Gemini CLI from the repository root in
headless mode. Prefer the project custom agent `gemini_engineer` when Codex
subagent orchestration is available; otherwise call Gemini directly:

```sh
gemini --approval-mode auto_edit --output-format json -p "<bounded task prompt>"
```

For analysis or planning that must not modify files, use:

```sh
gemini --approval-mode plan --output-format json -p "<bounded task prompt>"
```

Every delegation prompt must include:

- the concrete objective and relevant file scope;
- constraints and acceptance criteria;
- required validation commands;
- an instruction to preserve unrelated user changes and never commit;
- an instruction to return a concise summary with changed files, validation
  results, and unresolved risks.

Do not use `--yolo`. Do not place secrets in prompts or command output. Gemini
must obey `GEMINI.md`, which contains the repository's technical conventions.

The architect must inspect Gemini's diff and run or independently confirm the
appropriate checks before reporting completion. If Gemini fails, diagnose the
failure, tighten the prompt, and retry once when useful. The architect may take
over implementation when the remaining work meets the deep-reasoning criteria
above or a second delegation would be inefficient.

Avoid delegation for trivial conversational answers, a single obvious edit
that is faster to make safely, tasks requiring a specialized Codex skill, or
operations that need user approval before they can begin.

## Project Validation

- Run `npm run check` for code changes.
- Preserve the local-first privacy rules in `GEMINI.md`; never stage or commit
  `data/`, `.env`, secrets, or user-supplied images.
- Do not commit unless the user explicitly requests it.
