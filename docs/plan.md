# Plan: one marketplace, two plugins

**Status:** Shipped (catalog v1). Publishing to GitHub is still open — see [checklist.md](checklist.md).

## Problem

`claude-adapt-rules` and `claude-idle-shutdown` were two independent repositories. `claude-adapt-rules`
carried its own single-plugin `marketplace.json`; `claude-idle-shutdown` had no marketplace at all and
could only be installed from a local path. Anyone wanting both had to know about both repositories and
register two marketplaces.

Goal: one marketplace users add once, listing both plugins.

## Decision: catalog-only marketplace

This repository contains `.claude-plugin/marketplace.json` and documentation. No plugin code. Each
plugin is fetched from its own repository with a `github` source.

```
claude-secdude-plugins/
├── .claude-plugin/
│   └── marketplace.json      # the entire product
├── docs/
└── README.md
```

### Why

- **Different toolchains.** `claude-adapt-rules` is a Python package with pytest and a `pyproject.toml`;
  `claude-idle-shutdown` is Node with `node --test`. A monorepo forces one CI to carry both, and every
  contributor to either to install both.
- **Independent release cadence.** Each plugin keeps its own version, tags, and history. Shipping a fix
  in one does not touch the other.
- **`claude-adapt-rules` is multi-host.** It ships `.claude-plugin/`, `.codex-plugin/`, and
  `gemini-extension.json` manifests. Folding it into a Claude-specific marketplace repo would misfile it.
- **Cheap to extend.** A third plugin is one JSON object, not a repository migration.

### Cost accepted

Three repositories instead of one, and the catalog can drift from reality if a plugin repository is
renamed or deleted. The catalog is small enough that this is a cheaper problem than merged CI.

## Shapes rejected

| Shape | Why not |
| --- | --- |
| Monorepo with history preserved (subtree merge) | Merges Python and Node toolchains into one CI, and collapses two independent release tag series into one. |
| Monorepo with fresh history | Same coupling, and discards commit history from both repositories. |
| Keep `claude-adapt-rules`' own single-plugin marketplace as the shared one | Its marketplace name is `claude-adapt-rules`. Users installing `claude-idle-shutdown@claude-adapt-rules` reads as a mistake, and the plugin repo would own a catalog listing a plugin it has nothing to do with. |

## Sources are unpinned, deliberately

Neither entry sets `ref` or `sha`, so each plugin comes from its repository's default branch
(`master` for `claude-adapt-rules`, `main` for `claude-idle-shutdown`). Version resolution then falls
through to each plugin's own `plugin.json`.

Pinning to release tags is the better end state, but it needs both repositories to tag every release.
`claude-adapt-rules` is at `0.1.10` in `plugin.json` with tags only up to `v0.1.9`;
`claude-idle-shutdown` has no tags at all. Pinning today would either point at a tag that does not
exist or freeze a plugin at a stale commit. Tag first, then pin.

## Extending

1. Push the plugin to its own repository with a valid `.claude-plugin/plugin.json`.
2. Add an entry to `plugins` in `.claude-plugin/marketplace.json`. Only `name` and `source` are
   required; everything else is metadata for the plugin browser.
3. Run `claude plugin validate .`, commit, push.

Renames and removals go through the marketplace-level `renames` map so existing installs migrate
instead of breaking.
