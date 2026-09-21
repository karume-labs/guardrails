---
name: karume-guardrails
description: Apply Karume Labs guardrails to implementation and review tasks in projects that adopt this repository's rules. Use when the task involves TypeScript, React, Next.js, Elysia, oRPC, Drizzle, auth, validation, data fetching, UI styling, or the project toolchain; do not use for unrelated projects.
---

# Karume Guardrails

Route the task to the smallest relevant set of guardrails. This skill
complements `AGENTS.md`; it does not replace project-local instructions.

## Before editing

1. Read the target project's `AGENTS.md` and any more-specific instruction
   files. Project-local instructions take precedence.
2. Locate the guardrail source the project uses. In this repository, read
   `rules/AGENT-USAGE.MD`; in a consuming project, use its mirrored copy.
3. Follow that file's load order: read the always-on rules, then only the
   stack- and task-specific rules that apply. Read the matching file in
   `examples/` before inventing a new shape.

## While editing

- Preserve project-local conventions and make the smallest change that solves
  the task.
- Keep rule files focused and under roughly 150 lines. Update `AGENTS.md` and
  `README.MD` when adding or splitting a rule.
- Do not add tool-specific copies of this skill or duplicate rule content.

## Before finishing

- Run the target project's documented format, lint, typecheck, and relevant
  tests.
- Report commands that do not exist or could not run; do not claim
  verification that was not performed.
