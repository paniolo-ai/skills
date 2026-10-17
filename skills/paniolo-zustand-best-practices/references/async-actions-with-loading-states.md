---
source-slug: async-actions-with-loading-states
source-hash: 826b7a291276c9b2dc300451593cb11e65ce6f05cc952bc601a151269104f9b6
bundled: 2026-10-17
title: Async Actions with Loading States
type: concept
tags:
- authoring
- zustand
- react
- client
- state
updated: 2026-06-18
---

# Async Actions with Loading States

Track `isLoading` and `error` alongside data for every async action:

```typescript
type SongLibraryState = {
	songs: Song[];
	isLoading: boolean;
	error: Error | null;
	fetchSongs: (userId: string) => Promise<void>;
	clearError: () => void;
};

export const useSongLibraryStore = create<SongLibraryState>((set) => ({
	songs: [],
	isLoading: false,
	error: null,

	fetchSongs: async (userId) => {
		set({ isLoading: true, error: null });
		try {
			const songs = await songAPI.list(userId);
			set({ songs, isLoading: false });
		} catch (error) {
			set({
				error: error instanceof Error ? error : new Error("Unknown error"),
				isLoading: false,
			});
		}
	},

	clearError: () => set({ error: null }),
}));
```

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
