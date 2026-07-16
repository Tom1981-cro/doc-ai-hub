# doc-ai-hub

Private **documentation-automation hub** — the single place from which product
documentation, changelogs, and release materials are generated for **any** of our
repos. Based on [max-doc-ai](https://github.com/maxberko/max-doc-ai) (GPL-v3; see
`LICENSE`).

## Model: run it from here, target any repo

You do **not** install this into each product repo. You run it here and point it
at a repo:

```
# from this repo, in Claude Code:
@claude Create a release for <feature> in repo <owner/repo>
# or an individual skill:
@claude Skill: update-product-doc   Feature: <name>   Category: features
```

The `create-release` skill's pre-flight asks whether to use the current repo or an
**external GitHub repo**, so the same hub serves every product. Generated output
lands in `./output/` (gitignored).

## To use it on another repo

Nothing to add to that repo — just run the workflow here and pick it as the
external target. (Keep credentials for that product in `.env`.)

## Configuration

- `config.yaml` — product + workflow settings. Currently **Nasc + Pylon**; only
  references `${ENV_VARS}`, so it's safe to track.
- `.env` — secrets (gitignored). Copy `.env.example` → `.env` and fill in:
  `ANTHROPIC_API_KEY`, the `PYLON_*` / `COLLECTION_*` IDs, `PRODUCT_URL`,
  `SCREENSHOT_USER` / `SCREENSHOT_PASS`.
- `python3 scripts/setup.py` (interactive) and `python3 scripts/health_check.py`
  validate the setup — see `GETTING_STARTED.md` / `README.md`.

## Status

- ✅ Skills + scripts + config scaffolding in place, configured for Nasc + Pylon.
- ⛳ Pending: real Pylon credentials + Anthropic API key in `.env`, and confirming
  the Nasc app/docs URLs. Until then, code-research + doc-generation steps run;
  screenshot-capture + KB-sync need the live credentials.

---

## Use it across all your repos — Claude Code plugins

This repo is also a **Claude Code plugin marketplace** (`.claude-plugin/marketplace.json`)
exposing two plugins:

- **`docs-tracking`** — `/setup-docs-tracking` bootstraps the self-maintaining docs
  system (ROUTE_REFERENCE + SYSTEM_DOCS + `/hey-claude` + `/sync-route-reference`) in
  whatever repo you run it in. Self-contained — works anywhere after install.
- **`doc-generation`** — the 5 product-doc/release skills. `update-product-doc` and
  `create-changelog` run in any repo with no setup; `capture-screenshots`, `sync-docs`
  and `create-release` run this repo's Python toolchain — set `DOC_AI_HUB` to your
  checkout of this repo and `pip install -r "$DOC_AI_HUB/requirements.txt"` once.

### Install once (available in every repo)

```
/plugin marketplace add Tom1981-cro/doc-ai-hub
/plugin install docs-tracking@doc-ai-hub      # choose "User" scope → all repos
/plugin install doc-generation@doc-ai-hub     # choose "User" scope → all repos
```

Skills are then namespaced: `/docs-tracking:setup-docs-tracking`,
`/doc-generation:update-product-doc`, etc. Run `/setup-docs-tracking` once inside
each repo you want the docs-tracking system in.
