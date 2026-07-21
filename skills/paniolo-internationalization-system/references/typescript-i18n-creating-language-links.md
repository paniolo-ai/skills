---
source-wiki: sharp-shooter-wiki
source-slug: typescript-i18n-creating-language-links
source-hash: 6abe6bff27170c9d8cc44276dbd078a629ffee2f6ef44025272226166ecaf1f0
bundled: 2026-07-20
title: Creating Language Links
type: concept
tags:
- authoring
- i18n
- react
- client
- routing
updated: 2026-06-18
sources:
- raw/harness-eng/doc-fragment-extraction/doc-content-organization.md
---

# Creating Language Links

```typescript
import { useLanguage } from '@/react/src/language/useLanguage';

function LanguageAwareLink() {
  const { getLocalizedPath } = useLanguage();

  return (
    <Link to={getLocalizedPath('/song')}>
      {/* Automatically resolves to /zh/song, /es/song, etc. */}
      View Songs
    </Link>
  );
}
```

## See also

- Internationalization (i18n) (authoring) index