---
name: paniolo-internationalization-system
description: |
  i18n patterns for this project — useLocale/useLanguage/useCurrentLang hooks, URL-based language routing, adding translation keys, SupportedLanguageType. Use when adding UI text, building localized links, or working with language switching.
license: MIT
metadata:
  version: 0.1.0
tags:
- i18n
- react
references:
- 'wiki: sharp-shooter-wiki:i18n → references/i18n.md'
- 'wiki: sharp-shooter-wiki:typescript-i18n-creating-language-links → references/typescript-i18n-creating-language-links.md'
---

**Requires:** file-read, terminal (linting/testing). No network access needed.

**Reference:** [i18n](references/i18n.md)
— full architecture, hook internals, routing system, and configuration.

## Use When

Use this skill when:

- Adding/changing localized UI copy or language-aware routes/links.
- Editing language hooks, supported-language handling, or translation keys.

Execution workflow:

1. Use the appropriate language hook for context (`useLocale` default for UI).
2. Keep language in URL/path handling consistent with existing routing patterns.
3. Update translation keys across all supported language files.
4. Validate localized behavior with targeted tests/checks, then run `npm run lint`.

**Output:** summarize hook/key/path changes and impacted locales; note any fallback or
missing-translation behavior changes.

The project uses `react-i18next` with URL-path-based routing
(`/en/songs`, `/es/songs`, `/zh/songs`).
Supported languages: `"en" | "es" | "zh"` — from `@/shared/language/supported-languages`.

## Which Hook to Use

| Hook               | Returns       | Use when                                                                                            |
| ------------------ | ------------- | --------------------------------------------------------------------------------------------------- |
| `useLocale()`      | `{ lang, t }` | Component needs **both** language and translation function                                          |
| `useLanguage()`    | `lang`        | Component needs language only (e.g., building locale-aware URLs)                                    |
| `useCurrentLang()` | `lang`        | Need lang from URL pathname without component reactivity (or in tests with `{ pathname }` override) |

**Default: `useLocale()`** — the most ergonomic for UI components.

```typescript
import useLocale from "@/react/lib/language/locale/useLocale";

function MySongCard(): ReactElement {
  const { t, lang } = useLocale();
  return (
    <div>
      <p>{t("songs.by_artist")}</p>
      <a href={`/${lang}/songs`}>{t("nav.songs")}</a>
    </div>
  );
}
```

[i18n](references/i18n.md)

## Adding Translation Keys

Add the key to **all three files** in `react/src/translations/` (`en.json`, `es.json`, `zh.json`):

```json
{ "songs": { "by_artist": "By Artist" } }
```

Use dot notation in `t()`: `t("songs.by_artist")`.

[i18n](references/i18n.md)

## Building Language-Aware Links

Always prefix routes with `/${lang}/` — never hardcode `/en/` or `/`:

```typescript
// ✅ GOOD
const { lang } = useLocale();
<Link to={`/${lang}/community/${communityId}`}>...</Link>

// ❌ BAD
<Link to={`/en/community/${communityId}`}>...</Link>
```

To switch language on the same page, use `getPathWithoutLang`:

```typescript
const pathWithoutLang = getPathWithoutLang(location.pathname);
navigate(`/${newLang}${pathWithoutLang}`);
```

[typescript-i18n-creating-language-links](references/typescript-i18n-creating-language-links.md)

## Reading Language Outside a Component

Use `getCurrentLangFromPath` in non-component contexts or tests — falls back to
`defaultLanguage` for unrecognised paths.

[typescript-i18n-creating-language-links](references/typescript-i18n-creating-language-links.md)

## Language Provider

`LanguageProvider` syncs the i18next runtime with the URL. Do not call `i18n.changeLanguage()`
directly — let the provider handle it.

[i18n](references/i18n.md)

## Do Not

- ❌ Hardcode UI strings — always use `t("key")`
- ❌ Build links without the language prefix
- ❌ Import `i18n` directly in components — use `useLocale()` / `useTranslation()`
- ❌ Add keys to only one translation file — all three must be updated together

## References

- Supported languages: `@/shared/language/supported-languages`
- Full reference: [i18n](references/i18n.md)
- React conventions: [react-best-practices/SKILL.md](../paniolo-react-best-practices/SKILL.md)

## Skill Handoffs

- UI component implementation → also load `react-best-practices`.
- Key/path naming decisions → also load `naming-best-practices`.