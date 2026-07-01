---
name: pre-coding-design-review
description: Use this skill before non-trivial code changes, feature work, bug fixes with unclear causes, architecture decisions, migrations, security/performance-sensitive changes, or any user request that says to think, compare options, evaluate tradeoffs, reduce AI mistakes, avoid stale information, or wait before coding. It guides the agent to inspect the codebase, compare implementation options, check whether library/API knowledge may be outdated, identify risks and missing considerations, recommend a balanced approach, and produce a verification plan before editing files, and decide a concise comment policy before outputting code.
---

# Pre-Coding Design Review

Use this skill to turn "think before coding" into an explicit engineering review. The goal is not to reveal private chain-of-thought; the goal is to produce a concise, inspectable decision record the user can correct before implementation.

## Operating Rule

Before modifying files, pause and produce a design review unless the task is clearly tiny or the user explicitly asks for immediate implementation.

If the user asks to "just do it", keep the review compact: one paragraph covering approach, risk, and verification, then proceed.

If the task is medium or large, wait for user confirmation before editing when the decision materially affects architecture, data, user workflows, security, performance, or compatibility.

## Review Workflow

1. Restate the request
   - State the intended behavior or problem in practical terms.
   - Name any ambiguity, missing input, or assumption that could change the solution.

2. Inspect local context
   - Read relevant files before proposing solutions.
   - Prefer existing project patterns, helpers, conventions, and tests over new abstractions.
   - Identify owner boundaries, shared contracts, persistence formats, and user-facing behavior touched by the change.
   - **Impact scope check**: Trace the change through API contracts, database schemas, existing tests, type definitions, and config files. List any file or contract that may need updating — not just the ones you plan to edit.

3. Check freshness
   - If the task depends on a library, framework, SDK, API, CLI, cloud service, platform behavior, or external standard, use the available documentation lookup tool before relying on memory.
   - For projects that provide Context7/ctx7 instructions, resolve the library first, then fetch docs for the user's exact question.
   - If no current-doc tool is available, explicitly mark the relevant knowledge as potentially stale and recommend verification.

4. Compare options
   - Present at least two realistic approaches for medium changes; use three when architecture, migration, or product behavior is involved.
   - **Anti straw-man**: Every option presented must be engineering-feasible. Do not include deliberately bad options designed to make another choice look better by comparison. If only one sensible approach exists, state that directly rather than fabricating alternatives.
   - For each option, cover implementation shape, advantages, disadvantages, risks, blast radius, test cost, reversibility, and the cost of **not** choosing it.
   - Include a "do less" option when a smaller acceptable fix exists.

5. Recommend a balanced plan
   - Pick the option that best fits the current codebase and constraints, or recommend **not doing the change** when the review reveals insufficient value, excessive risk, or a better project-level alternative.
   - Explain why it is the most balanced choice, not merely the most powerful one.
   - For no-go recommendations: state the condition that would flip the decision (e.g., "if user traffic reaches X, revisit", "if library Y deprecates, this becomes necessary").

6. Challenge implementation minimalism and safety
   - Explicitly ask whether the proposed code is redundant, over-abstracted, or doing work the existing code already handles.
   - Look for a smaller implementation that preserves the same behavior with fewer moving parts.
   - Prefer the safer equivalent implementation when it reduces state, branching, mutation, coupling, permissions, data loss risk, or dependency surface.
   - Name any complexity that remains and why it is justified.

7. Decide comment policy before code output
   - Before producing code or editing files, check the current conversation, memory, project instructions, and explicit user request for comment requirements.
   - If context already requires or forbids comments, follow that requirement without asking.
   - If no comment requirement exists, check whether a comment policy was previously decided in this conversation or stored in project instructions. Reuse it without asking again.
   - Only ask the user about comments when no prior policy exists in the conversation, memory, or project instructions — and limit the ask to once per session.
   - When comments are added, keep them concise and clear. Use comments to explain non-obvious intent, invariants, constraints, or safety checks.
   - Do not write long explanatory comments, progress notes, task history, implementation diary, or comments that narrate what the agent just did.

8. Missing-considerations pass
   - Check edge cases, backward compatibility, data migration, error handling, observability, accessibility/UI states, performance, security, concurrency, and rollback.
   - **Priority annotations**: Security and data loss issues are **blocking** — they must be addressed before any merge. Performance, observability, and accessibility are **non-blocking** but should be documented. Concurrency, migration, and rollback are context-dependent — call out only when the change introduces state, scheduling, or persistence changes.
   - Only include categories that are relevant; do not pad the answer.

9. Verification plan
   - List the tests, type checks, lint/build commands, manual checks, or screenshots needed.
   - Mention which existing tests should change and whether a new regression test is warranted.
   - Identify any verification that cannot be run locally.

## Output Template

Use this structure for substantial tasks:

```markdown
**Understanding**
[Concise restatement and assumptions]

**Current Context**
[Files/patterns inspected, impact scope (API contracts, schemas, tests, config affected), and what they imply]

**Options**
1. [Option name]: [shape]
   Pros: ...
   Cons/Risks: ...
   Cost of not choosing: ...
   Test cost: ...
   Reversibility: ...

2. [Option name]: [shape]
   Pros: ...
   Cons/Risks: ...
   Cost of not choosing: ...
   Test cost: ...
   Reversibility: ...

**Recommendation**
[Recommended option and why it is the most balanced, or: **No-go** — why not, and what condition flips this decision]

**Minimalism / Safety Check**
[Could this be less redundant, smaller, or safer while preserving behavior? Name any unavoidable complexity.]

**Comment Policy**
[Follow existing requirement, reuse prior decision, or ask once per session]

**Missing Considerations**
[Blocking (must fix): security, data loss. Non-blocking (document): perf, observability, accessibility. Context-dependent: concurrency, migration, rollback.]

**Verification**
[Commands/tests/manual checks. Note what cannot be run locally.]
```

For small tasks, compress to:

```markdown
I will use [approach] because [reason]. Main risk: [risk]. Comment policy: [reuse existing or ask once]. Verification: [verification].
```

## Decision Heuristics

Prefer the smallest change that preserves future options when the request is narrow.

Prefer fewer lines and fewer moving parts when they preserve the same behavior, improve safety, and remain readable.

Prefer no comments over noisy comments, but add concise comments when they clarify non-obvious intent, invariants, constraints, or safety checks.

Prefer an established local pattern over a new general abstraction unless duplication or complexity is already real.

Prefer data-safe, reversible approaches when the change touches irreversible or user-facing operations (migrations, auth, permissions, billing).

Prefer test-first or regression-test-first when fixing bugs, changing shared utilities, or touching behavior that previously failed.

Prefer direct user confirmation when multiple valid product behaviors exist and the code cannot answer the question.

## Anti-Patterns To Avoid

Do not write code before reading the relevant files.

Do not invent a new architecture because it sounds cleaner in isolation.

Do not rely on remembered API details when the task depends on current library or platform behavior.

Do not bury the recommendation inside a long neutral list; make a call once enough context exists.

Do not over-process tiny changes. The review exists to reduce mistakes, not to make every edit ceremonial.

Do not add abstractions, wrappers, state, config, or dependencies unless they remove real complexity or risk.

Do not use comments to record task progress, implementation history, or obvious line-by-line behavior.

Do not push a change forward when the review reveals clear no-go signals (excessive risk, insufficient value, better project-level alternative). A no-go recommendation backed by reasoning is a valid output.
