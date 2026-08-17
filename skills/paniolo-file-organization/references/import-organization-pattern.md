---
source-wiki: sharp-shooter-wiki
source-slug: import-organization-pattern
source-hash: 6cf2c30f6b05a4bb093cae9617c6a01fe534d66262905a4fb9f3295e93d9f409
bundled: 2026-08-01
title: Import Organization Pattern
type: concept
tags:
- authoring
- file-organization
updated: 2026-06-18
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

## See also

- File organization (authoring) index
