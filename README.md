# secdude-plugins

A Claude Code plugin marketplace. One catalog, three plugins.

| Plugin | What it does | Runtime | Source |
| --- | --- | --- | --- |
| `claude-adapt-rules` | Mines your sessions for the corrections you had to repeat, then distils them into evidence-backed rules — per-repo automatically, global only with your approval. | Python | [Patrick-DE/claude-adapt-rules](https://github.com/Patrick-DE/claude-adapt-rules) |
| `claude-idle-shutdown` | Arms a watchdog that powers the machine off once every Claude session on it has gone idle. | Node | [Patrick-DE/claude-idle-shutdown](https://github.com/Patrick-DE/claude-idle-shutdown) |
| `claude-refusal-detector` | Detects LLM safety triggers and pinpoints minimal refusal text. | Python | [Patrick-DE/claude-refusal-detector](https://github.com/Patrick-DE/claude-refusal-detector) |

## Install

Add the marketplace once:

```bash
/plugin marketplace add Patrick-DE/claude-secdude-plugins
```

Then install whichever plugins you want:

```bash
/plugin install claude-adapt-rules@secdude-plugins
```

```bash
/plugin install claude-idle-shutdown@secdude-plugins
```

```bash
/plugin install claude-refusal-detector@secdude-plugins
```

Pull catalog changes later with `/plugin marketplace update secdude-plugins`.

## What lives here

Only the catalog. This repository contains `.claude-plugin/marketplace.json` and documentation —
no plugin code. Each plugin stays in its own repository with its own tests, versions, and release
tags; the catalog points at them with `github` sources. Cloning this repo gets you the index, not
the plugins.

That split is deliberate: the three plugins have different toolchains (Python vs. Node) and different
release cadences, and none should be forced to move when another ships.

`claude-adapt-rules` also ships its own single-plugin marketplace, named `claude-adapt-rules`. It
still works and is not going away — if you added it already, nothing to do. This catalog is the one
that carries all three plugins.

## Versions and pinning

Neither entry pins a `ref` or `sha`, so each plugin is fetched from its repository's default branch
(`master` for `claude-adapt-rules`, `main` for `claude-idle-shutdown`). Users receive a new version
when the plugin's own `plugin.json` version changes.

To pin a plugin to a release instead, add a `ref` to its `source`:

```json
"source": {
  "source": "github",
  "repo": "Patrick-DE/claude-adapt-rules",
  "ref": "v0.1.10"
}
```

Pinning is worth doing once both repositories tag every release consistently. See
[docs/checklist.md](docs/checklist.md).

## Adding a plugin

1. Push the plugin to its own repository with a valid `.claude-plugin/plugin.json`.
2. Add an entry to the `plugins` array in `.claude-plugin/marketplace.json` — `name` and `source`
   are the only required fields; the rest is metadata shown in the plugin browser.
3. Validate, commit, push. Users pick it up on their next `/plugin marketplace update`.

Renaming or removing an entry later? Use the marketplace-level `renames` map so existing installs
migrate instead of breaking.

## Local development

Point Claude Code at this working copy instead of GitHub to test catalog changes before pushing:

```bash
/plugin marketplace add C:\Users\patri\sources\repos\claude-secdude-plugins
```

Validate the manifest:

```bash
claude plugin validate .
```

## Docs

- [docs/INDEX.md](docs/INDEX.md) — documentation index
- [docs/plan.md](docs/plan.md) — why the catalog-only shape was chosen
- [docs/checklist.md](docs/checklist.md) — what is done and what is still open

## License

MIT
