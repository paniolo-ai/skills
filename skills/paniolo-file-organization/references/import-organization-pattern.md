---
source-wiki: sharp-shooter-wiki
source-slug: import-organization-pattern
source-hash: 75d911e7cd998c972fe6333122e887771cef0b38d4c7b60f346461f7f695af6c
bundled: 2026-07-20
title: Import Organization Pattern
type: concept
tags:
- authoring
- file-organization
updated: 2026-06-18
sources:
- raw/harness-eng/songshare-ai-system/skill-file-organization.md
---

# Import Organization Pattern

Detailed reference for import patterns, naming conventions, project structure, and file organization
strategies.

# Import Organization Pattern

Standard organization for all TypeScript/React files:

```typescript
// 1. External packages (npm dependencies)
import React, { useState, useEffect } from "react";
import { Effect, pipe } from "effect";
import { createClient } from "@supabase/supabase-js";
import { clsx } from "clsx";

// 2. Absolute imports (if aliased in tsconfig)
import { Button } from "@shared/components/Button";
import { Song, SongError } from "@shared/types/song";
import { formatDuration } from "@shared/utils/formatDuration";

// 3. Internal relative imports (from same package/feature)
import { SongCard } from "../components/SongCard";
import { useSongLibrary } from "../hooks/useSongLibrary";
import { SONG*BATCH*SIZE } from "./constants";
import type { PageProps } from "./types";

// 4. Style imports (if using CSS files - rare with Tailwind)
import styles from "./Page.module.css";
```

## See also

- File organization (authoring) index