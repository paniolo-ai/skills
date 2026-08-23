---
source-slug: cross-layer-data-flow-verification
source-hash: 3471b429e2ee2f5f88d70ff4435ea4d0be2f73fbe9a53a7adba1585418f97a14
bundled: 2026-08-20
title: Cross-Layer Data Flow Verification
type: concept
tags:
- architecture
- plans
- testing
- verification
updated: 2026-08-20
---

# Cross-Layer Data Flow Verification

Cross-layer changes are complete only when data travels from its producer through every required
boundary and consumer. A correct editor, endpoint, or database row does not prove downstream use.

## Contents

- [Trace The Contract](#trace-the-contract)
- [Evidence Matrix](#evidence-matrix)
- [Behavior Tests](#behavior-tests)
- [Failure Patterns](#failure-patterns)
- [See Also](#see-also)

---

<a id="trace-the-contract"></a>

## Trace The Contract

Trace the value in dependency order:

```text
producer -> boundary validation -> persistence -> loading -> hydration -> state -> every consumer
```

- **Producer:** form, import, migration, integration, or generated default.
- **Boundary validation:** request schema, authorization, normalization, and domain invariants.
- **Persistence:** schema, constraints, atomic write, and read-after-write shape.
- **Loading:** API query, relation selection, caching, and error handling.
- **Hydration:** mapping from transport records into client or service models.
- **State:** store, reducer, query cache, or derived selector.
- **Consumers:** editors, detail views, presenters, exports, background jobs, and alternate modes.

Search for readers as well as writers. Shared types can compile while one consumer still ignores a
new value or reconstructs an obsolete default.

---

<a id="evidence-matrix"></a>

## Evidence Matrix

Use one row for every participating boundary and consumer:

| Layer Or Consumer | Contract | Implementation | Behavior Evidence | Result |
| --- | --- | --- | --- | --- |
| Producer | Value can be created or changed | Path or symbol | Focused creation test | Status |
| Persistence | Exact value survives a write | Path or migration | Read-after-write test | Status |
| Hydration and state | Loaded value reaches shared state | Path or selector | State-level test | Status |
| Consumer A | Consumer renders or applies the value | Path or component | Behavior test | Status |
| Consumer B | Alternate mode uses the same value | Path or component | Behavior test | Status |

Add rows until every required consumer from the product contract is represented. A broad suite
name without an assertion tied to the value is not behavior evidence.

---

<a id="behavior-tests"></a>

## Behavior Tests

Use a distinctive non-default value so an obsolete fallback cannot accidentally pass. Save it,
reload through the real boundary, then assert every consumer's observable behavior. Include at
least one test that would fail against the previous implementation.

When consumers share transformation logic, test that logic once and still exercise each consumer's
integration. Shared unit coverage does not prove that every consumer calls the shared function.

---

<a id="failure-patterns"></a>

## Failure Patterns

- Updating create and edit paths but not duplicate, import, or migration paths.
- Returning a field from the API without selecting it in one alternate query.
- Hydrating state correctly while one presentation mode reads a stale local default.
- Testing only the editor that produced the value.
- Treating typecheck success as proof that runtime consumers use the value.

---

<a id="see-also"></a>

## See Also

- Plan Acceptance Evidence
- Plan Authoring Best Practices
- Plan Execution Workflow
