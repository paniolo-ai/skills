---
source-wiki: sharp-shooter-wiki
source-slug: async-actions-with-loading-states
source-hash: b2d4c61a261e441bd42802fd2f9be03e5eccacb5d8a91735b5f3ac0b7bca27d5
bundled: 2026-07-24
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

## See also

- Zustand (authoring) index