# kb-pub

Public publishing target for [`kevcooper/kb`](https://github.com/kevcooper/kb),
served by GitHub Pages from `docs/`.

**This repository is generated.** `docs/` is built by CI in the private `kb`
repo and replaced wholesale on every publish — hand edits there are lost. This
README and anything else at the repo root are left alone.

## Why a separate repo

`kb` is a private knowledge corpus. Pages on a private repo needs a paid plan,
and making `kb` public would expose the corpus alongside the datasets that are
safe to share.

Splitting them makes the privacy boundary **physical rather than logical**: the
corpus does not exist in this repository at all, so no bug in the build script
can leak it. Only third-party-derived datasets — material that was already
public — are ever published here.

## What gets published

Static views of datasets held under `data/` in `kb`, each carrying provenance
in its own `SOURCE.md`. The site serves two audiences:

- a filterable HTML view, built to work on a phone with bad conference wifi
- stable JSON endpoints so the site can be queried directly by an agent:
  `/index.json`, `/llms.txt`, and `/<dataset>/sessions.json`

## Setup

GitHub Pages: **Settings → Pages → Deploy from a branch → `main` / `docs`**.

Publishing is done by `.github/workflows/publish-kb-pub.yml` in `kb`, which
needs a `KB_PUB_TOKEN` secret there — a fine-grained PAT scoped to this
repository alone with Contents: read/write.
