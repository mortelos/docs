# MortelOS Docs

This repository contains the public Markdown source for MortelOS documentation.

The current documentation version is `0`. Version `0` is written on `main`: the
public site maps `/docs/0/{slug}` to `main` until version 1 ships and version `0`
is frozen on its own branch.

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

