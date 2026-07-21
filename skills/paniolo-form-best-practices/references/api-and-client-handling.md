---
source-wiki: sharp-shooter-wiki
source-slug: api-and-client-handling
source-hash: de7fa18d0bdfbe7dca2b10e1b9006a1e2b9c125af47664448c193d254bb405a2
bundled: 2026-07-20
title: API and Client Handling
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

# API and Client Handling

API handlers should fail expected validation problems with `ValidationError`.
When a specific field caused the failure, include `field`. Do not return
field-specific validation failures as `DatabaseError`, `ServerError`, or a
generic `500`.

```typescript
return Effect.fail(
	new ValidationError({
		message: "An event with this slug already exists.",
		field: "event_slug",
	}),
);
```

`errorToHttpResponse` converts this into a `400` response with the shared error
shape:

```json
{
	"success": false,
	"error": "An event with this slug already exists.",
	"field": "event_slug"
}
```

Client API wrappers should parse non-ok responses and preserve both the message
and field.

```typescript
const errorBody = await readSaveEventErrorBody(response);

return Effect.fail(
	new EventSaveApiError(`Failed to save event: ${errorBody.message}`, {
		statusCode: response.status,
		field: errorBody.field,
		fieldMessage: errorBody.message,
	}),
);
```

Feature hooks should catch typed field errors and push them into
`setValidationErrors`. When a field error is handled locally, do not navigate,
do not set the global store error for the same failure, and do not log it as an
unexpected client error.

```typescript
saveEvent(request).pipe(
	Effect.catchAll((error) => {
		if (error instanceof EventSaveApiError && error.field !== undefined) {
			setValidationErrors([
				{
					field: error.field,
					message: error.fieldMessage ?? error.message,
				},
			]);
			return Effect.succeed("");
		}

		return Effect.fail(error);
	}),
);
```

Use these typed API errors consistently:

- `ValidationError` for bad input or user-correctable business rules.
- `AuthenticationError` for unauthenticated requests.
- `AuthorizationError` for authenticated users lacking permission.
- `NotFoundError` for missing resources.
- `DatabaseError`, `ProviderError`, `FileUploadError`, or `ServerError` for
  unexpected failures.

Translate known database constraints into validation errors when they map to a
form field.

```typescript
if (isEventSlugUniqueError(publicResult.error)) {
	return yield * $(Effect.fail(makeEventSlugTakenError()));
}
```

## See also

- Forms (authoring) index