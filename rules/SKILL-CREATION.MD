# Skill Creation

Guidance for creating and maintaining skills or rules for AI agents.

## Why this file

To ensure skills remain effective, maintainable, and aligned with the capabilities of modern AI models, reducing prompt bloat and maximizing reasoning.

## 1. Don't Repeat Yourself (DRY)

- Consolidate information to a single location instead of stating it across multiple skills or multiple times within the same skill.
- Each piece of information should have one home to make it easier to maintain.
- Be thoughtful about where that home should be: does it warrant its own skill, or does it make sense inside an existing one?
- When another skill needs that information, link to the respective skill instead of redefining it.

## 2. Respect the Model's Intelligence

- Focus on the **goal** of the task and provide all the **context** necessary for the model to make intelligent decisions.
- Clearly define what "done" looks like so the model knows when to stop and when to keep working.
- Trust the model's reasoning by focusing on flexible outcomes rather than detailing rigid steps on how the model should behave.
- **Exceptions:** Rigid rules or guardrails should only be added when there's evidence they are necessary. Explain the purpose of each rule so the model understands *why* it's there.
- **Cleanup:** Periodically remove guardrails and rules and test the skills without them. More capable models need fewer rules.
- **Skip metacognition prompts:** Rely on the model's natural reasoning instead of including guidance like "think carefully" or "use a thinking block". The latest models know how much to think.

## 3. Handling Complex Systems

- When the AI needs to access a complex system (e.g., task management software like Notion, CRM like Salesforce), create a separate skill detailing the structure of that system.
- Any skill that asks the AI to work in that system should link to that separate skill.
- For less complex systems (e.g., Google Drive, Gmail, Slack), a separate skill might not be necessary. However, whenever the AI accesses an outside system, it should always be instructed to **look around first**.
- Looking around to see how a system is structured improves efficiency and accuracy. If the system turns out to be sufficiently complex, the AI should map it out and create a skill.

## 4. Prompting Best Practices

- **Use clear formatting:** Use markdown, headers, and XML tags to structure the skill logically.
- **Provide examples:** Include few-shot examples of what good outputs or actions look like. Examples are often more effective than abstract instructions.
- **Be direct:** State clearly what the AI should do. Omit polite filler words (please/thank you) in favor of concise instructions.
- **Tell, don't forbid:** Whenever possible, frame rules as positive instructions ("Do X") rather than negative constraints ("Don't do Y").

## 5. Iterative Refinement

- Start simple. Build skills iteratively based on observed failure modes rather than anticipating every edge case upfront.
