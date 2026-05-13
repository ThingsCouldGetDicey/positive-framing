# Positive Framing

A skill for converting negative instructions ("don't", "never", "no") to their closest positive equivalent. Prevents concept leakage — the well-documented tendency of LLMs to activate negated concepts and execute them anyway.

## Why this matters

LLMs process text token by token. When you write "don't suggest workarounds," the model activates "suggest workarounds" in its representation *before* it processes the negation. The concept is already primed by the time the "don't" is interpreted.

This happens through three mechanisms:

1. **Token order**: The concept tokens are processed before the negation token
2. **Statistical weight**: Positive contexts vastly outnumber negated contexts in training data
3. **Context presence**: The concept tokens are physically in the context window, available for attention

## The fix

Replace negations with the closest positive equivalent that activates the desired behavior:

| Negative (leaks concept) | Positive (activates correct behavior) |
|---------------------------|---------------------------------------|
| Don't guess | Verify before stating |
| Don't suggest workarounds | Propose structural fixes only |
| Don't stop at the first explanation | Keep asking why until nothing deeper explains it |
| Never commit secrets | Store secrets in environment variables only |
| Don't improvise | Use verified sources only |

## Installation

Copy the `SKILL.md` file to your Letta Code skills directory:

```bash
mkdir -p ~/.letta/skills/positive-framing
cp SKILL.md ~/.letta/skills/positive-framing/
```

## Usage

In your Letta Code conversation:

```
Use the positive-framing skill.
```

Or: apply it as a default behavior — when the user gives a negative instruction, convert it to the closest positive equivalent before acting on it.

## Conversion guidelines

1. **Stay close to the original intent** — the positive statement must mean the same thing
2. **Activate the replacement behavior** — specify what to do, not just what to avoid
3. **Be specific** — name the exact tool, method, or approach
4. **Apply recursively** — check that the replacement doesn't contain negations
5. **Apply to all output** — instructions, skills, memory, code comments, prompts

## License

MIT
