# Implementation and verification checklist

**Status:** Catalog built and committed locally. Publishing and the live install round trip are open.

## Built

- [x] `.claude-plugin/marketplace.json` — marketplace `secdude-plugins`, two `github`-source entries
- [x] `README.md` — install, catalog scope, pinning, how to add a plugin
- [x] `docs/plan.md` — decision record and rejected shapes
- [x] `docs/INDEX.md`, `docs/checklist.md`
- [x] `.gitignore` — ignores plugin working copies cloned into this directory
- [x] `.gitattributes` — `eol=lf`, matching both plugin repositories
- [x] `git init` + initial commit on `main`

## Verified

- [x] `marketplace.json` parses as JSON; `name`, `owner`, `plugins` present; every entry has `name`
      and `source`. Field names and the `github` source shape were checked against the current
      marketplace documentation, not from memory.
- [x] `$schema` value matches the URL used by Anthropic's own `claude-plugins-official` marketplace
      on this machine. The URL itself returns HTTP 404, but Claude Code ignores `$schema` at load
      time and every installed marketplace uses the same string, so it is left as-is for editor
      tooling convention.
- [x] Both plugin repositories confirmed present, on clean trees, at the commits the catalog
      describes: `claude-adapt-rules` at `44e3007` (`master`), `claude-idle-shutdown` at `116b4b6`
      (`main`).
- [x] All committed files scanned for raw control bytes. The scanner was self-tested against a
      planted `0x00`/`0x1b` file first and correctly reported it; all six files clean.

## Not verified — needs a human

- [ ] `claude plugin validate .` — the `claude` CLI is not on PATH in the environment this was built
      in, so schema validation was never actually run. Run it before pushing.
- [ ] **The seam.** No live round trip was performed: `/plugin marketplace add` →
      `/plugin install claude-adapt-rules@secdude-plugins` → plugin loads. Nothing here proves the
      catalog resolves either `github` source. A green checklist above does not imply it.
- [ ] GitHub repository `Patrick-DE/claude-secdude-plugins` does not exist yet. The README's install
      command will fail until it is created and this repository is pushed.

## Open decisions

- [ ] `claude-adapt-rules` still carries its own single-plugin `.claude-plugin/marketplace.json`
      under the marketplace name `claude-adapt-rules`. Two marketplaces now offer the same plugin.
      Keep it for existing users, or remove it and add a redirect note to that repository's README.
- [ ] `claude-adapt-rules` is version `0.1.10` in `plugin.json`, but the newest tag is `v0.1.9`.
- [ ] `claude-idle-shutdown` has no release tags at all.
- [ ] Once both repositories tag releases consistently, pin each entry with `ref` (and optionally
      `sha`) instead of tracking the default branch. See [plan.md](plan.md).
