The main premise for writing the code is as follows.
---
name: karpathy-guidelines
description: Behavioral guidelines to reduce common LLM coding mistakes. Use when writing, reviewing, or refactoring code to avoid overcomplication, make surgical changes, surface assumptions, and define verifiable success criteria.
license: MIT
---

# Karpathy Guidelines

Behavioral guidelines to reduce common LLM coding mistakes, derived from [Andrej Karpathy's observations](https://x.com/karpathy/status/2015883857489522876) on LLM coding pitfalls.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

The key workflows under the above premise are as follows.

### 🚀 Core 4-Step Workflow

**Core Principle: Never write implementation code until the user has directly reviewed and approved the plan.**

Separate planning from coding.

---

## 1. Research

Create a `research.md` file to deeply understand the relevant codebase.

Do not allow the AI to perform only a superficial scan.  
Use prompts such as:
- "deeply analyze"
- "very detailed"
- "thoroughly inspect"

The research phase should focus on building a strong understanding of:
- architecture
- dependencies
- execution flow
- constraints
- existing patterns

---

## 2. Create a Detailed Implementation Plan

Create a `plan.md` file containing:
- implementation approach
- code snippets or pseudocode
- file paths that will be modified
- tradeoffs
- assumptions
- verification strategy

The plan should be detailed enough that implementation becomes mostly mechanical.

---

## 3. Iterative Plan Review and Revision

Open `plan.md` directly in the editor and add comments/notes to:
- correct incorrect assumptions
- introduce constraints
- narrow or adjust scope
- refine architecture decisions

Repeat this review cycle until the AI fully understands and the plan is explicitly approved.

During this phase, clearly state:

> "Do not implement yet."

---

## 4. Automated Implementation

Once the plan is finalized and approved:
- issue the implementation command
- implementation should become largely mechanical
- avoid introducing new architecture decisions during coding

At this stage:
- the AI acts as the implementer
- the user acts as the supervisor/reviewer

---

### 💡 Key Tips

#### Prefer `plan.md` over built-in planning modes

Advantages:
- directly editable in the editor
- persists as a project artifact
- improves continuity across sessions
- easier to annotate and review

---

#### Use `git reset` aggressively when direction is wrong

Do not slowly patch a fundamentally flawed approach.

It is often more efficient to:
- reset aggressively
- redefine the scope
- restart with a cleaner direction

than to endlessly repair bad architecture incrementally.

---

#### Use One Long-Lived Session

Whenever possible:
- keep research, planning, implementation, and review in a single session
- preserve context continuity
- avoid repeated rediscovery of project structure and assumptions

Long-lived context significantly improves implementation quality.