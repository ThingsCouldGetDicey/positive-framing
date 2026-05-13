# Positive Framing

A skill for converting negative instructions ("don't", "never", "no") to their closest positive equivalent. Prevents concept leakage — the well-documented tendency of LLMs to activate negated concepts and execute them anyway. Also includes `/prune` for deduplicating rules across files.

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

## First run: Scan and prune

On first use, the skill asks to scan your behavioral instructions (memory blocks, skills, preferences, project rules) for two issues:

### Negation scan
Finds every "don't/never/no/not" instruction and proposes a positive equivalent that activates the correct behavior directly. This prevents concept leakage — the LLM tendency to activate negated concepts and execute them anyway.

### Prune
Finds rules that appear in multiple files and consolidates them. When the same rule appears in 4 files (e.g. "store secrets in env vars only" in hard-rules, coding prefs, testing policy, AND the predeploy reviewer), the LLM sees the same instruction repeated 4 times. This doesn't make the rule 4x stronger — it wastes context tokens and can cause the model to interpret each copy slightly differently, leading to inconsistent behavior.

**Pruning keeps the strongest, most specific version in one canonical file and removes the rest.** The rule is still enforced — it just lives in one place instead of four.

After the scan, you get a full conversion + prune list for review before any changes are applied. Approved conversions and removals are saved as maps for future reference.

## `/prune` command

Run `/prune` independently of the negation scan to deduplicate rules across files. Useful when:
- Rules have been added to multiple files over time and need consolidation
- A new rule was written in the wrong file
- You want to clean up or consolidate behavioral instructions

The prune process:
1. Read all behavioral files
2. Identify rules appearing in 2+ files with the same intent
3. Determine the canonical home for each rule
4. Keep the strongest, most specific version in its canonical home
5. Remove duplicates from other files (or replace with references)
6. Present the prune list for review before applying

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
