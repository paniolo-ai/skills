---
source-wiki: sharp-shooter-wiki
source-slug: hook-api-reference
source-hash: ba1d251729c995ca661f449ef3f78ed2dd0eaa9a75b6031304b00d25687a89b2
bundled: 2026-07-20
title: Hook API Reference
type: concept
tags:
- authoring
- form
- react
- validation
updated: 2026-06-18
sources:
- raw/harness-eng/doc-fragment-extraction/doc-content-organization.md
---

# Hook API Reference

```typescript
import useAppForm from "@/react/lib/form/useAppForm";
import { Effect } from "effect";
import { useRef } from "react";
import { MyFormSchema } from "./myFormSchema";

function MyForm(): ReactElement {
	const formRef = useRef<HTMLFormElement>(null);
	const {
		validationErrors,
		isSubmitting,
		handleFieldBlur,
		getFieldError,
		clearFieldError,
		handleSubmit,
		handleApiResponseEffect,
		reset,
	} = useAppForm({
		schema: MyFormSchema,
		formRef,
		defaultErrorMessage: "Something went wrong. Please try again.",
	});
}
```

| Prop                  | Type                               | Required | Description                     |
| --------------------- | ---------------------------------- | -------- | ------------------------------- |
| `schema`              | `Schema.Schema<FormValues>`        | ✅       | Effect Schema for validation    |
| `formRef`             | `React.RefObject<HTMLFormElement>` | ✅       | Ref to the `<form>` element     |
| `defaultErrorMessage` | `string`                           | optional | Fallback for generic API errors |
| `initialValues`       | `Partial<FormValues>`              | optional | Values used by `reset()`        |

`handleSubmit` returns an `Effect.Effect<void>` and automatically validates, sets
`isSubmitting`, populates `validationErrors` on failure, and calls `onSubmit(validatedData)`
on success:

```typescript
function handleFormSubmit(formData: Record<string, unknown>): void {
	Effect.runFork(
		handleSubmit(formData, async (validatedData) => {
			const response = await fetch("/api/songs", {
				method: "POST",
				body: JSON.stringify(validatedData),
			});
			await Effect.runPromise(handleApiResponseEffect(response, setSubmitError));
		}),
	);
}

<form
	ref={formRef}
	onSubmit={(e) => {
		e.preventDefault();
		handleFormSubmit(Object.fromEntries(new FormData(e.currentTarget)));
	}}
>
```

Validate individual fields on blur:

```typescript
<input
	name="title"
	ref={titleRef}
	onBlur={() => handleFieldBlur("title", titleRef)}
/>
{getFieldError("title") && <p>{getFieldError("title")?.message}</p>}
```

## See also

- Forms (authoring) index