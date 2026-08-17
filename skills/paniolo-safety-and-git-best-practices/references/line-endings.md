---
source-wiki: sharp-shooter-wiki
source-slug: line-endings
source-hash: 246d83ad2b6662f3c0becfaf43e158dd757fda90980679b89e6f3d4da5ccae92
bundled: 2026-08-16
title: Line Endings
type: concept
tags:
- authoring
- git
- formatting
- cross-platform
- windows
updated: 2026-08-15
---

# Line Endings

Every repo uses **LF** in committed content. Declare it once in
`.gitattributes`, at the repo root:

```text
* text=auto eol=lf
```

That is the whole policy. The rest of this page is about the failure it does
*not* prevent.

## `.gitattributes` normalizes on commit, not on write

`eol=lf` converts CRLF to LF when git stores a blob. It does nothing to the
file sitting in your working tree. So a tool that writes CRLF — a Windows
editor, a PowerShell redirect, an agent using a Windows text API — leaves a
CRLF file on disk that:

- `git status` reports as clean, because the normalized blob is unchanged;
- `git diff` shows as no change, for the same reason;
- every other tool reads as CRLF, because they read the file, not the blob.

The bug therefore only appears in tools that compare file bytes: hash checks,
golden-file tests, generated-output diffs, digest stamps.

## Symptom: a byte comparison fails on a "clean" tree

If a digest, golden, or generated-output check fails and the diff viewer shows
identical text, count the CR bytes before looking anywhere else:

```bash
tr -cd '\r' < path/to/file | wc -c    # expect 0
```

A non-zero count on a file `git status` calls clean is the answer. Normalize it:

```bash
python -c "p='path/to/file'; d=open(p,'rb').read().replace(b'\r\n',b'\n'); open(p,'wb').write(d)"
```

## Generators normalize their own output

Any tool that writes a file which is later compared byte-for-byte must emit LF
regardless of host platform, and end the file with exactly one `\n`. Do not
rely on the platform default and do not rely on `.gitattributes` to clean up
afterwards — by the time git sees it, the comparison has already failed.

The same applies to the trailing newline. A file with none, or with two, is a
byte difference that no rendered view will show you.

## Configure it, do not infer it

When a tool needs to know a project's line-ending policy, read the declaration
the project already makes — `.gitattributes` first, then `.editorconfig` —
rather than guessing from the current file's contents. Sampling existing files
gets the wrong answer exactly when it matters most: in a repo that has already
drifted.

## See also

- git-commit-format-and-style — commit message conventions.
- rules — hook and CI enforcement.
