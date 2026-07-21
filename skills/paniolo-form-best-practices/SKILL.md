---
name: paniolo-form-best-practices
description: |
  Standardized form patterns using useAppForm and Effect helpers. Covers validation, submission, and immediate error clearing on field edit.
license: MIT
metadata:
  version: 0.1.0
tags:
- react
- forms
references:
- 'wiki: sharp-shooter-wiki:api-and-client-handling → references/api-and-client-handling.md'
- 'wiki: sharp-shooter-wiki:form → references/form.md'
- 'wiki: sharp-shooter-wiki:hook-api-reference → references/hook-api-reference.md'
- 'wiki: sharp-shooter-wiki:schema-definition → references/schema-definition.md'
user-invocable: true
---

**Requires:** file-read, terminal (linting/testing). No network access needed.

**Reference:** [form](references/form.md) —
full API reference, code examples, error taxonomy, accessibility, and checklist.

## Use When

Use this skill when building or modifying form validation/submission behavior in React, or
wiring API response handling through `useAppForm` and Effect helpers.

1. Use `useAppForm` as the primary form abstraction.
2. Keep schema validation and submission flow typed and effect-driven.
3. Reuse shared response/error handling helpers instead of ad-hoc form logic.
4. Validate with targeted form tests, then `npm run lint`.

**Output:** summarize form flow changes (validation, submit, response handling); note
user-visible error-handling behavior changes.

All forms use `useAppForm` from `@/react/lib/form/useAppForm`. Do not use raw React state
and `onSubmit` — the project has typed Effect/validation plumbing you must use.

## useAppForm Hook

Import, call with `schema` + `formRef`, destructure needed helpers. Returns `handleSubmit`
(an `Effect.Effect<void>`), `handleFieldBlur`, `getFieldError`, `clearFieldError`,
`handleApiResponseEffect`, `isSubmitting`, `reset`.

[hook-api-reference](references/hook-api-reference.md)

## Handling Submission

Run `handleSubmit` with `Effect.runFork`; call `handleApiResponseEffect` inside the submit
callback. The hook validates, sets `isSubmitting`, populates `validationErrors` on failure,
and calls `onSubmit(validatedData)` on success.

[hook-api-reference](references/hook-api-reference.md)

## Field Blur Validation

Call `handleFieldBlur(fieldName, fieldRef)` in the field's `onBlur` handler; render errors
with `getFieldError(fieldName)`.

[hook-api-reference](references/hook-api-reference.md)

## Clearing Errors on Change

Call `clearFieldError(fieldName)` immediately inside every field change handler to give
responsive feedback that the user is fixing the problem.

## API Response Handling

`handleApiResponseEffect` maps API responses to form errors. The API must return one of
these JSON shapes for routing to work:

```json
{ "type": "setFieldError",   "field": "email", "message": "Already in use" }
{ "type": "setGeneralError", "message": "Server error" }
```

[api-and-client-handling](references/api-and-client-handling.md)

## Defining a Form Schema

Colocate an Effect Schema in `<feature>FormSchema.ts` next to the component.

[schema-definition](references/schema-definition.md)

## Do Not

- ❌ Use raw `useState` for validation errors — use `useAppForm`
- ❌ Call `e.target.value` in submit handlers — read from `FormData` or the validated object
- ❌ Write `try/catch` for form validation — `handleSubmit` handles it
- ❌ Mix `useAppForm` with React Hook Form or other form libraries

## References

- [form](references/form.md)
- [Effect TS Patterns](../paniolo-effect-ts-patterns/SKILL.md)
- Source: `@/react/lib/form/useAppForm.ts`

## Skill Handoffs

- Effect-style response handling → also load `effect-ts-patterns`.
- React-heavy component work → also load `react-best-practices`.