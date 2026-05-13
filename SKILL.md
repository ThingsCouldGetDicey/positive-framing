---
name: positive-framing
description: "Convert all negative instructions to their closest positive equivalent. Use when writing prompts, skill files, system instructions, memory blocks, code comments, or any text that guides LLM behavior. Prevents concept leakage from negated statements. Supports /prune for deduplication."
---

# Positive Framing

Convert all negative instructions ("don't", "never", "no", "not") to their closest positive equivalent. This prevents concept leakage — the well-documented tendency of LLMs to activate negated concepts and execute them anyway.

## First run: Scan, convert, and prune

On first use, ask the user for permission to scan their existing behavioral instructions for two issues:

1. **Negations** — "don't", "never", "no", "not" instructions that leak concepts (see "Why this works" below for the mechanism)
2. **Duplicates** — the same rule appearing in multiple files, which weakens the signal and can cause conflicting interpretations

**What pruning does:** When the same rule appears in multiple files (e.g. "store secrets in env vars only" in hard-rules, coding prefs, testing policy, AND the predeploy reviewer), the LLM sees the same instruction repeated 4 times. This doesn't make the rule 4x stronger — it wastes context tokens and can cause the model to interpret each copy slightly differently, leading to inconsistent behavior. Pruning keeps the strongest, most specific version in one canonical file and removes the rest. The rule is still enforced — it just lives in one place instead of four.

**Scan targets:**
- Agent memory blocks (system prompt files)
- Skill files (SKILL.md)
- Preference files
- Project-specific rules and hard-rules
- Any file that guides agent behavior

**Process:**
1. Ask: "Can I scan your behavioral instructions for negations and duplicates, then convert and prune them? Here's what each step does:

   **Negation scan** — Finds every 'don't/never/no/not' instruction and proposes a positive equivalent that activates the correct behavior directly (preventing concept leakage).

   **Prune** — Finds rules that appear in multiple files, keeps the strongest version in one canonical location, and removes the duplicates. The rule is still enforced — it just lives in one place instead of several, saving context tokens and preventing conflicting interpretations."

2. Read all relevant files
3. List every negation found with its file location
4. For each negation, propose the closest positive equivalent
5. List every duplicate rule found across files, noting which file should be the canonical home
6. Present the full conversion + prune list for review
7. Apply approved conversions and removals
8. Save the conversion map and prune map as references for future use

**Conversion map format** (saved after first run):
```
## User's conversion map
| File | Original | Converted |
|------|----------|-----------|
| system/human/prefs/communication.md | Don't over-research | Keep answers proportional |
| system/guitar-academy/hard-rules.md | Don't invent schema fields | Use only verified schema fields |
| ...
```

**Prune map format** (saved after first run):
```
## User's prune map
| Rule | Canonical home | Removed from |
|------|---------------|-------------|
| Structural fixes only | persona.md | hard-rules.md, workflow.md |
| Use Playwright for SvelteKit | testing-policy.md | hard-rules.md, coding.md |
| ...
```

On subsequent runs, the conversion map and prune map are already available. The skill:
- Applies the map to any new instructions the user gives in conversation
- Scans newly created or edited files for negations before saving
- Detects duplicates when new rules are added to multiple files
- Updates the maps when new patterns are discovered

## /prune command

The `/prune` command runs deduplication independently of the negation scan.

**What it does:** Finds rules that appear in 2+ files, keeps the strongest version in one canonical location, and removes the duplicates. The rule is still enforced — it just lives in one place instead of several. This saves context tokens and prevents the model from interpreting slightly different copies of the same rule inconsistently.

Use it when:
- Rules have been added to multiple files over time and need consolidation
- A new rule was written in the wrong file
- The user asks to clean up or consolidate their behavioral instructions

**Prune process:**
1. Read all behavioral files
2. Identify rules that appear in 2+ files with the same intent
3. For each duplicate group, determine the canonical home:
   - **persona.md**: Behavioral principles (how I work)
   - **hard-rules.md**: Project-specific non-negotiables
   - **testing-policy.md**: Testing rules
   - **coding.md**: Coding preferences (only things distinct from hard-rules)
   - **workflow.md**: Workflow preferences (only things distinct from persona)
   - **predeploy-reviewer.md**: Deployment gate rules
4. Keep the strongest, most specific version of each rule in its canonical home
5. Remove duplicates from other files (or replace with [[path]] reference if context is needed)
6. Present the prune list for review before applying

**Prune guidelines:**
- Keep the version that is most specific and actionable
- If a rule is in hard-rules.md, it's authoritative — remove from persona/coding/workflow
- If a rule is in persona.md as a behavioral principle, remove from workflow/coding
- Use [[path]] references when a file needs to point to the canonical rule
- Preserve all unique content — only remove exact or near-exact duplicates

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
- Running `/prune` to deduplicate behavioral rules
- Any prompt engineering task where reliability matters

## Integration

This skill applies as a **default behavior** for the agent. When the user gives a negative instruction ("don't do X"), convert it to the closest positive equivalent before acting on it. The conversion happens automatically — the user doesn't need to invoke this skill explicitly.

The user invokes this skill explicitly when:
- Writing new skill files or system prompts
- Editing existing instructions to improve reliability
- Reviewing prompts for concept leakage
- Running the first-run scan to convert existing negations and prune duplicates
- Running `/prune` to deduplicate rules across files
