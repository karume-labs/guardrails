---
name: karume-guardrails
description: Apply Karume Labs guardrails to implementation and review tasks in projects that adopt this repository's rules. Use when the task involves TypeScript, React, Next.js, Elysia, oRPC, Drizzle, auth, validation, data fetching, UI styling, or the project toolchain; do not use for unrelated projects.
---

# Karume Guardrails

Route the task to the smallest relevant set of guardrails. This skill complements `AGENTS.md`.

## Goal

Correctly apply the target project's active guardrails based on their defined priority. Project-local instructions always take precedence.

## Preparation

- Locate and review the project's `AGENTS.md` and any domain-specific instructions.
- Identify the project's guardrail source (e.g., `rules/AGENT-USAGE.MD` in this repository, or its mirrored copy in consumers).
- Follow the defined load order from the source: read always-on rules first, followed by relevant stack/task-specific rules.
- Review matching patterns in `examples/` before creating a new architectural shape.

<example>
**Task:** Create a new React component for user settings.
**Action:** The agent reads `AGENTS.md` to understand conventions, checks `rules/AGENT-USAGE.MD` to see the loading priority, then reads `rules/REACT.MD` and `rules/STYLING.MD`. The agent reviews `examples/` for existing UI patterns before writing the component.
</example>

## Editing Guidelines

- Preserve project-local conventions and make the smallest change that solves the task.
- Keep rule files focused and under roughly 150 lines. Update `AGENTS.md` and `README.MD` when adding or splitting a rule.
- Maintain a single, centralized source of truth for all rules across tools and agents.

## Verification

- Run the target project's documented format, lint, typecheck, and relevant tests.
- Only report commands and checks that were successfully executed.
