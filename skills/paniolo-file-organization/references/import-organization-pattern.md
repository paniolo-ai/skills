---
source-slug: import-organization-pattern
source-hash: 7bdd54b3c762c19dd89a71c0d056b83b76f8a8aa6fd4dbcd658faf93628f8a40
bundled: 2026-10-17
title: Import Organization Pattern
type: concept
tags:
- authoring
- file-organization
updated: 2026-06-18
---

# Import Organization Pattern

Detailed reference for import patterns, naming conventions, project structure, and file organization strategies.

# Import Organization Pattern

Standard organization for all TypeScript/React files:

```typescript
// 1. External packages (npm dependencies)
import React, { useState, useEffect } from "react";
import { Effect, pipe } from "effect";
import { createClient } from "@example/db-client";
import { clsx } from "clsx";

// 2. Absolute imports (if aliased in tsconfig)
import { Button } from "@shared/components/Button";
import { Item, ItemError } from "@shared/types/item";
import { formatDuration } from "@shared/utils/formatDuration";

// 3. Internal relative imports (from same package/feature)
import { ItemCard } from "../components/ItemCard";
import { useItemLibrary } from "../hooks/useItemLibrary";
import { ITEM_BATCH_SIZE } from "./constants";
import type { PageProps } from "./types";

// 4. Style imports (if using CSS files - rare with Tailwind)
import styles from "./Page.module.css";
```

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
