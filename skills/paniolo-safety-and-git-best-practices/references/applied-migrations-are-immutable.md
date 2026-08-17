---
source-wiki: sharp-shooter-wiki
source-slug: applied-migrations-are-immutable
source-hash: 63ff609efc30b1fd98ff85aed9507f81e3800474fbc978b5c4f808b62187d25b
bundled: 2026-08-16
title: Applied Migrations Are Immutable
type: concept
tags:
- authoring
- database
- supabase
- safety
updated: 2026-08-15
---

# Applied Migrations Are Immutable

Once a migration has been applied to any shared environment, **its file never changes again**.
Correct it by adding a new migration.

This holds for edits that look harmless. Reworded comments, reformatted SQL, and renamed
constraints all change the file, and migration tooling tracks applied state by filename and content
checksum. A changed file is either a checksum mismatch that blocks the next deploy, or worse, a
change that is quietly never applied because the runner already considers that migration done — so
the environments you cared about keep the old definition while the file claims otherwise.

## The rule

- **Not yet applied anywhere** — editing in place is fine.
- **Applied to staging, production, or a shared branch** — add a new migration.
- **Unsure** — assume applied. Adding a migration that turns out to be redundant costs one file;
  editing one that was already applied costs a broken deploy.

## Recognising the trap

The edit that caused this page was a **comment reworded inside an already-applied migration** —
which is exactly the sort of change a review waves through, because it changes no SQL semantics.
The file is still modified, and that is the part the tooling cares about.

If a migration's description is wrong, fix it where descriptions live: the schema comment via a new
migration, or the project's docs. Do not fix it by editing history.

## Before running anything

Migration commands are frequently environment-suffixed, and the **unsuffixed one is usually
production**:

```bash
<migrate command>:staging   # environment-suffixed: explicit target
<migrate command>            # unsuffixed: often production — check before running
```

Read the script definition rather than inferring the target from the name. Run the staging path,
confirm it is green, and report before promoting.

## See also

- Authoring
