---
name: positive-framing
description: "Convert all negative instructions to their closest positive equivalent. Use when writing prompts, skill files, system instructions, memory blocks, code comments, or any text that guides LLM behavior. Prevents concept leakage from negated statements."
---

# Positive Framing

Convert all negative instructions ("don't", "never", "no", "not") to their closest positive equivalent. This prevents concept leakage — the well-documented tendency of LLMs to activate negated concepts and execute them anyway.

## First run: Scan and convert

On first use, ask the user for permission to scan their existing behavioral instructions, preferences, skill files, and memory blocks for negations. Then convert every negation found to its closest positive equivalent.

**Scan targets:**
- Agent memory blocks (system prompt files)
- Skill files (SKILL.md)
- Preference files
- Project-specific rules and hard-rules
- Any file that guides agent behavior

**Process:**
1. Ask: "Can I scan your behavioral instructions for negations and convert them to positive framing?"
2. Read all relevant files
3. List every negation found with its file location
4. For each negation, propose the closest positive equivalent
5. Present the full conversion list for review
6. Apply approved conversions
7. Save the conversion map as a reference for future use

**Conversion map format** (saved after first run):
```
## User's conversion map
| File | Original | Converted |
|------|----------|-----------|
| system/human/prefs/communication.md | Don't over-research | Keep answers proportional |
| system/guitar-academy/hard-rules.md | Don't invent schema fields | Use only verified schema fields |
| ...
```

On subsequent runs, the conversion map is already available. The skill:
- Applies the map to any new instructions the user gives in conversation
- Scans newly created or edited files for negations before saving
- Updates the map when new patterns are discovered

## Why this works

LLMs process text token by token, left to right. When you write "don't suggest workarounds," the model activates "suggest workarounds" in its representation *before* it processes the negation. The concept is already primed by the time the "don't" is interpreted.

Three mechanisms cause this:

1. **Token order**: The concept tokens are processed before the negation token. The positive concept is activated first, and the negation may not fully suppress it.

2. **Statistical weight**: In training data, concepts appear in positive contexts far more often than negated ones. "Suggest workarounds" has millions of training examples. "Don't suggest workarounds" has almost none. The positive association is orders of magnitude stronger.

3. **Context presence**: When "don't do X" is in the prompt, the tokens for "X" are physically in the context window. The model can attend to them directly. Under complex instruction following, the negation can get dropped while the concept stays active.

## The conversion rule

When you encounter a negative instruction, replace it with the closest positive equivalent that activates the desired behavior directly:

| Negative (leaks concept) | Positive (activates correct behavior) |
|---------------------------|---------------------------------------|
| Don't guess | Verify before stating |
| Don't suggest workarounds | Propose structural fixes only |
| Don't stop at the first explanation | Keep asking why until nothing deeper explains it |
| Don't use vitest in SvelteKit | Use Playwright for SvelteKit tests |
| Don't hardcode URLs | Use config/env variables for all URLs |
| Don't trust browser state | Verify all state server-side |
| Don't push without tests | Require passing tests before pushing |
| Never commit secrets | Store secrets in environment variables only |
| No workarounds as fixes | Apply structural fixes only |
| Don't improvise | Use verified sources only |
| Don't over-explain | Match answer depth to question complexity |
| Don't drop the current task | Stack new tasks, continue current work |
| Don't fake closure | Close branches only when structural fix is confirmed |

## Conversion guidelines

1. **Stay close to the original intent.** The positive statement must mean the same thing as the negation. "Don't guess" → "Verify before stating" (not "Be certain" — too vague).

2. **Activate the replacement behavior, not just the absence.** "Don't guess" is weak because it only specifies what to avoid. "Verify before stating" specifies what to do instead.

3. **Be specific.** "Don't use vitest" → "Use Playwright for SvelteKit tests" — the replacement names the exact tool and context.

4. **Apply recursively.** When converting a negative instruction, check that the positive replacement doesn't itself contain negations. "Don't use unverified sources" → "Use verified sources only" (removed the double negation).

5. **Apply to all output.** Instructions, skill files, memory blocks, code comments, system prompts, documentation — anything that guides LLM behavior.

## When to use this skill

- Writing or editing skill files
- Writing or editing memory blocks / system prompts
- Writing code comments that guide behavior
- Converting user-given negative instructions into positive equivalents before acting on them
- Any prompt engineering task where reliability matters

## Integration

This skill applies as a **default behavior** for the agent. When the user gives a negative instruction ("don't do X"), convert it to the closest positive equivalent before acting on it. The conversion happens automatically — the user doesn't need to invoke this skill explicitly.

The user invokes this skill explicitly when:
- Writing new skill files or system prompts
- Editing existing instructions to improve reliability
- Reviewing prompts for concept leakage
- Running the first-run scan to convert existing negations
