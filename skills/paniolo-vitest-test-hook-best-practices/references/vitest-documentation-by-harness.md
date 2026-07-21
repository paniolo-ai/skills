---
source-wiki: sharp-shooter-wiki
source-slug: vitest-documentation-by-harness
source-hash: 3fea1b13358652a6446ff46460bf1d365a2934e2b6ae5cf19973c0db98c2fdad
bundled: 2026-07-20
title: Documentation by Harness
type: concept
tags:
- authoring
- vitest
- testing
- react
- hooks
updated: 2026-06-18
sources:
- raw/harness-eng/doc-fragment-extraction/doc-content-organization.md
---

# Documentation by Harness

A **Harness component** is a small React component that mounts the hook and exposes its outputs
through DOM elements. **Every hook test file must include at least one**, regardless of whether
`renderHook` covers all behavior.

**Purpose:** AI agents and future developers need to see how the hook integrates into real UI code.
The Harness shows _how you actually use it_. This is the "Documentation by Harness" pattern.

The Harness must be **thorough and complete** (see completeness checklist below). At minimum:

- Each prop the hook accepts — what it represents
- Each return value wired into JSX — what state it controls or element it attaches to
- Each handler — what event it responds to and what it does
- Any non-obvious wiring (ref attachment, conditional rendering, portals)

Note: Harness tests must assert every property the hook returns (state values, derived values,
and handler functions). Rendering return values into the DOM is not sufficient by itself — the
Harness must include assertions that verify those values and handler effects across the relevant
interactions. If a return value is not directly observable, expose it in the Harness (for example
by rendering it to a `data-testid` element or wiring a handler to a button) so the test can
validate the hook's contract end-to-end.

For JSDoc in TypeScript/TSX harnesses, use field-level params and skip the wrapper object entry:
use `@param activeCommunityId` / `@param onSelect`, not `@param props`.

```tsx
/**
 * Harness for useMyHook.
 *
 * Shows how useMyHook integrates into a real UI:
 * - A text input that updates the search query
 * - A list of filtered results shown while the dropdown is open
 * - A container div that the click-outside listener is attached to
 */
function Harness(props: {
	activeCommunityId: string | undefined; // pre-selected community (undefined = none)
	onSelect: (id: string) => void; // called when the user picks a result
}): ReactElement {
	// Always destructure — React Compiler rejects hook.property access in JSX
	const {
		containerRef, // ref for the outer div — click-outside closes the dropdown
		inputRef, // ref for the <input> — focus is managed externally
		searchQuery, // controlled value of the search field
		isOpen, // true when the dropdown is visible
		filteredCommunities, // list filtered by the current searchQuery
		handleInputFocus, // sets isOpen = true on focus
		handleInputChange, // updates searchQuery on keystroke
		handleSelectCommunity, // picks a community: calls onSelect, closes dropdown
	} = useMyHook(props);

	return (
		<div ref={containerRef} data-testid="container">
			<input
				ref={inputRef}
				data-testid="search-input"
				value={searchQuery}
				onFocus={handleInputFocus}
				onChange={handleInputChange}
			/>
			{isOpen && (
				<ul data-testid="results">
					{filteredCommunities.map((c) => (
						<li
							key={c.community_id}
							data-testid={`result-${c.community_id}`}
							onClick={() => {
								handleSelectCommunity(c.community_id);
							}}
						>
							{c.community_name}
						</li>
					))}
				</ul>
			)}
		</div>
	);
}
```

## See also

- Vitest hook testing (authoring) index