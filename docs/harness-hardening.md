# Harness hardening — audit fixes of 2026-08-09

**Status:** Shipped 2026-08-09. All three repos committed, pushed, tagged
(`claude-adapt-rules` v0.1.11, `claude-idle-shutdown` v0.1.0), catalog pinned to those refs,
ty debt fixed and gated in CI. 2026-08-09 evening: the first-ever Linux CI run caught a real
pre-existing bug — `Path("C:\\...").name` on POSIX returns the whole string, so Windows-style
cwds (which sit next to WSL-style ones in a single transcript store) produced no project name
and inject went silent. Fixed host-independently via `PureWindowsPath` in **v0.1.12**; catalog
pins moved there. Last human step: update the installed plugin to 0.1.12, then delete the
remaining `hooks` block from `~/.claude/settings.json` (runbook step 5).

Audit goal: reduce agent-introduced bugs. Diagnosis: the harness was instruction-heavy but
enforcement-light — four competing planning frameworks, five review paths, three memory layers,
zero deterministic verification. The fixes convert prose gates into mechanical ones and write the
tie-breaks down once.

## What shipped

| Item | Change | Verified |
| --- | --- | --- |
| Hook double-fire (C1) | `~/.claude/settings.json` no longer wires dev copies of `inject.py`/`capture.py` — the installed `claude-adapt-rules` plugin (0.1.10) provides them. Dev `guard.py` stays (not shipped in 0.1.10). Plugin 0.1.11 prepped in the dev repo: ships guard as PreToolUse, version bumped in `plugin.json` + `pyproject.toml`. | settings.json parses; one hook set per event |
| PATH-dependent hooks (C2) | Remaining hook uses `C:\Python314\python.exe` instead of bare `python`. Plugin hooks keep portable `python` (other machines). | — |
| Statusline (C3) | **Accepted as-is.** Copying `caveman-statusline.ps1` out of the marketplace clone would freeze it out of upstream security fixes (it carries symlink/escape-injection hardening). Failure mode if the marketplace vanishes is a silently absent statusline — benign. | — |
| Lint debt | `claude-adapt-rules` had 60 ruff findings (tooling configured but never executed — no uv/ruff on this machine until now). 29 auto-fixed, 31 E501s hand-wrapped with byte-identical strings. | `ruff check` clean; 176 pytest green before and after |
| Hookify rules (E1) | Per-repo advisory gates: python/node edit-verification + stop-checklists in both plugin repos, marketplace-validation rule in the catalog. `.claude/*.local.md` gitignored in all three. | files load next tool use |
| File protections (E5) | Global `permissions.deny` in `~/.claude/settings.json`: Edit/Write blocked on `.env`, `.env.local`, `.env.production`, and all lockfiles (`package-lock.json`, `pnpm-lock.yaml`, `yarn.lock`, `uv.lock`, `Cargo.lock`). `.env.example` stays editable. | settings.json parses |
| Pre-commit gates (E2) | `.githooks/pre-commit` + `core.hooksPath` in all three repos: ruff+pytest / `npm test` / `json.tool` (+`claude plugin validate` when CLI present). Enforces R-0024 mechanically. | all three executed green on this machine |
| CI (E3) | GitHub Actions: `ci.yml` in both plugin repos (ubuntu+windows matrix; ruff+pytest via uv / `npm test` on Node 22), `validate.yml` in the catalog. | runs on first push — see runbook |
| CLAUDE.md precedence (O1–O3, M2) | New top section "Precedence & process spine": superpowers = the one process spine, rtt-\* = executors, one review gate, memory division of labor, deterministic-gates-beat-prose, no caveman-compress on instruction files. Backup at `~/.claude/CLAUDE.md.bak-2026-08-09`. | — |

## Known-red / open items

- **ty: FIXED 2026-08-09** — the 18 diagnostics were 3 sites (`archive.py` None-default,
  `harness.py` double-`get` narrowing, `ledger.py` `from_dict` splat). Root-cause fixes
  (`default_factory`, local-variable narrowing, explicit dict copy), full suite green, ty now
  gated in `ci.yml` alongside ruff and pytest.
- **`claude plugin validate` in catalog CI is unverified** (CLI wasn't on PATH in the working
  shells here). If the Action fails on auth, drop that step; the JSON-syntax gate stands.
- **feature-dev plugin stays enabled** — disabling it in settings.json was blocked by the
  permission classifier. The spine section neutralizes the conflict; to remove it entirely, flip
  `"feature-dev@claude-plugins-official": false` yourself.
- **MongoDB MCP: user decided 2026-08-09 to leave it desktop-global**
  (`%APPDATA%\Claude\claude_desktop_config.json`). Revisit only if an unrelated project ever
  touches the cluster; scoping recipe: remove there, add per-project `.mcp.json`.
- **CLAUDE.md learned-rules block untouched** (tool-owned by claude-adapt-rules). Candidates for
  retirement via its own review flow, since Verification Discipline covers them:
  R-0004, R-0024, R-0025, R-0028.

## Release runbook (tags before pins — pinning to a nonexistent tag breaks installs)

1. Review + commit the changes in each repo (suggested commits at the end of the session summary).
2. `claude-adapt-rules`: `git tag v0.1.11 && git push origin master --tags`
3. `claude-idle-shutdown`: `git tag v0.1.0 && git push origin main --tags`
4. Then pin this catalog — in `.claude-plugin/marketplace.json`, add to each source:
   `"ref": "v0.1.11"` / `"ref": "v0.1.0"`. Commit, push.
5. Update the installed plugin to 0.1.11, then delete the remaining `hooks` block from
   `~/.claude/settings.json` — guard then comes from the plugin. **Until you do this, do not
   update the plugin past 0.1.10 with the settings hook still present, or guard fires twice.**
6. Each future release: bump `plugin.json` + `pyproject.toml` together (the stop-checklist
   reminds you), tag, push tags, move the catalog ref.

## Checklist

- [x] settings.json: double-fire removed, absolute python, deny-list added
- [x] plugin 0.1.11 prepped (guard hook, version sync)
- [x] ruff debt cleared, 176 tests green
- [x] 6 hookify rules, 3 gitignores
- [x] 3 pre-commit gates wired + executed green
- [x] 3 CI workflows written
- [x] CLAUDE.md spine section, backup taken
- [x] MongoDB MCP located, scoping documented
- [x] Commit + push all three repos (adapt-rules `5525d3b`, idle-shutdown `56dcbe3`)
- [x] Tags v0.1.11 / v0.1.0 pushed, catalog `ref` pins added
- [x] ty diagnostics fixed (3 sites), ty gated in CI
- [x] MongoDB: decision recorded — stays desktop-global
- [x] POSIX cwd-parsing bug (found by first CI run) fixed in v0.1.12, regression-tested
- [ ] Human: update installed plugin to 0.1.12, then delete `hooks` block from
      `~/.claude/settings.json` (guard fires twice until then — do it in that order)
- [ ] Optional: disable feature-dev (`"feature-dev@claude-plugins-official": false`)
- [ ] Optional: retire R-0004/R-0024/R-0025/R-0028 via claude-adapt-rules' own review flow
