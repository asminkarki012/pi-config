---
name: my-payload-workflow
description: Apply the user's personal coding and collaboration preferences when implementing, debugging, or reviewing PayloadCMS projects. Use alongside the PayloadCMS skill; this skill adds preferences, not Payload API reference material.
compatibility: Pi coding agent; PayloadCMS projects
---

# My PayloadCMS Workflow

Personal overlay for PayloadCMS work. It complements the `payload` skill; it does not replace its technical guidance.

- For Payload APIs, collections, fields, hooks, access control, Local API behavior, transactions, and generated types, use the `payload` skill and docs for the installed version. For scope discipline, clarifying ambiguity, simplicity, and general verification, follow `using-agent-skills`.
- Follow this repository's own instructions and architecture. This skill adds preferences; it does not override explicit task requirements or repository rules.

## My coding preferences

### Payload relationship IDs

When a Payload relationship value must be normalized to an ID, prefer `extractID` from `payload/shared` over repeating manual `typeof` checks or accessing `.id` directly. Add a null guard when the value can be nullish.

```ts
import { extractID } from 'payload/shared'

const idOf = (value: string | { id: string } | null | undefined): string | null =>
  value == null ? null : extractID(value)
```

### Optional relationship IDs

An unset relationship is normal — a student group before its exam exists, an entry whose user was deleted. Do not write the null check at every call site. Use `extractOptionalID` from `@/lib/ids`, which returns `undefined` for an absent value and delegates to `extractID` otherwise.

```ts
// src/lib/ids.ts
export const extractOptionalID = <IDType extends ID>(
  value: { id: IDType } | IDType | null | undefined,
): IDType | undefined => (value == null ? undefined : extractID(value))
```

```ts
// consumer — no ternary, no typeof
import { extractOptionalID } from '@/lib/ids'

const examId = extractOptionalID(group.exam)
if (!examId) return formatted
```

Rules:

- `undefined` means "not provided"; `null` is reserved for a real answer of "none" (for example `resolveTuitionColumnIndex` returning `null` for "this file has no tuition column"). Keep the two apart so callers do not have to write `== null` everywhere.
- Use `extractID` directly when the relationship is required by the schema; use `extractOptionalID` whenever the type allows `null` or `undefined`.
- In Moodle-facing code this is the same value that becomes a national ID or a Moodle user id; a missing one must be reported, never defaulted.

### Named constants instead of repeated literals

Never repeat a string or number literal that carries meaning (status values, question types, roles, slugs, collection slugs) across files. Declare it once, freeze it, export it, and import it everywhere.

```ts
// src/constants/question.ts
export const QUESTION_TYPE = Object.freeze({
  MCQ: 'mcq',
  OPEN: 'open',
} as const)

export type QuestionType = (typeof QUESTION_TYPE)[keyof typeof QUESTION_TYPE]
```

```ts
// consumer
import { QUESTION_TYPE } from '@/constants/question'

if (question.type === QUESTION_TYPE.MCQ) {
  // ...
}
```

Rules:

- Prefer `as const` (optionally wrapped in `Object.freeze`) for a fixed set of values. `as const` alone already gives literal types and blocks writes at compile time; add `Object.freeze` when the object crosses into untyped runtime code.
- Derive the union type from the constant instead of writing the union separately, so the two cannot drift.
- Reuse the same constant in Payload field config, so options and comparisons stay in sync:

```ts
{
  name: 'type',
  type: 'select',
  options: [
    { label: 'Multiple choice', value: QUESTION_TYPE.MCQ },
    { label: 'Open', value: QUESTION_TYPE.OPEN },
  ],
}
```

- Keep shared constants in one place (for example `src/constants/*.ts`) and import from there. Do not redeclare the same constant in two files, and do not export a constant from a component or collection file for other modules to reach into.
- Do not create a constant for a literal used exactly once and only locally; that adds indirection without removing duplication.

### Readability

- Prefer clear code over compact or clever code.
- Avoid comments that merely restate the code. Comment when explaining why a non-obvious choice exists, an important constraint, or a real edge case.
- Keep ternaries simple. For complex conditions or branches, prefer `if`/`else` or a clearer named value.

## Repository safety

- Before editing, inspect `git status` and relevant diffs. Preserve existing work; never reset, clean, or overwrite it as a shortcut.
- Do not stage or commit unless I ask. If I ask you to stage changes, stage only the requested files.
- Never expose `.env` contents, credentials, tokens, or private user data.
- If I ask to discuss or analyze only, do not edit files.

### Scope Discipline — Minimal Changes Only
- **Stay within scope.** Make only changes necessary to fulfill the explicit task. Do not add features, refactor, clean up, or modify unrelated code.
- **Keep changes minimal.** Follow existing architecture and patterns; modify the fewest files possible.
- **Preserve existing behavior.** Avoid unintended changes outside the requested functionality.
- **Use judgment.** Make necessary supporting changes independently without asking about routine implementation decisions.
- **Flag unrelated issues.** Report them separately; do not fix them unless asked.
- **Ask before expanding scope.** If substantial changes beyond the task are necessary, explain why and get approval first.
- **Verify the diff.** Remove unrelated changes you introduced and report changed files and relevant tests.

## Testing

For behavior changes and bug fixes, write or update a focused regression test that reproduces the behavior, confirm it fails for the expected reason, then make the smallest fix. Run the repository's actual typecheck, lint, and build commands as appropriate, and report unrelated failures accurately.
