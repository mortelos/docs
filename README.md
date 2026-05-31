# MortelOS Docs

This repository contains the public Markdown source for MortelOS documentation.

The current documentation version is `0`. The branch name is the version name,
so the public URL `/docs/0/{slug}` reads from branch `0`.

## Repository Contract

1. Documentation content lives in `mortelos/docs`.
2. Application code lives outside this repository.
3. The public site reads this repository as a content source.
4. Every page has front matter with a stable `slug`, `title`, `version` and `order`.

## Local Checks

```bash
jq . mortelos/docs/versions.json
jq . mortelos/docs/navigation.json
rg -n "^title:|^slug:|^order:" mortelos/docs
```

