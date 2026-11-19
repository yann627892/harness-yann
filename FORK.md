# This fork

**Harness Yann** (`yann627892/harness-yann`) is an unofficial fork of Harness Maxxing
(`makushingu/harness-maxxing`) plus three features kept on top of upstream. It follows upstream `stable` closely:
every update is a merge, not a rebase, so `maxxing update` on the server gets upstream's versions **and** these
features. `NOTICE.md` states the filiation and what the Fair Source license allows; the repository, its releases
and any listing must not use the names "Harness Maxxing" or "Maxxing" beyond saying the work is based on them.

Branches: `stable` and `gateway-providers` are the same commit (the server updates from `stable`). Never push to
`origin` (403): push to `fork`.

## What the fork adds

| Feature | Entry points | Code |
|---|---|---|
| Several LLM gateways at once (OpenCode Zen, OpenCode Go, OpenRouter, custom) beside a token or an API key; each session picks its credential with its model | Settings -> Claude -> Gateway; model picker tags; `provider` on sessions; `POST /api/claude/auth` | `server/maxxing/gateway.py`, `api/claude.py`, `claude_auth.py`, `sessions/spawn.py` (`claude_env`), `sessions/runner.py`, `sessions/service.py` (`_check_provider`, `public`), `web/src/features/claude/ClaudeConnect.tsx`, `web/src/pages/sessions/Widgets.tsx` |
| Follow-up suggestions above the composer (small model, one call per idle turn, cache + per-session lock) | `GET /api/sessions/{id}/suggestions`; Settings -> Claude -> Follow-up suggestions (`config.suggestions`, default `true`) | `server/maxxing/sessions/suggest.py`, `api/sessions.py`, `web/src/pages/sessions/Composer.tsx`, `Column.tsx` |
| Run a command on a server from the Infra page | `POST /api/infra/exec`; **Run a command** | `server/maxxing/api/infra.py`, `web/src/features/infra/RunCommand.tsx`, `InfraPage.tsx` |

State: `config.json` `gateways = {id: {provider, base_url, models}}` + `secrets.json`
`gateway_key_<id>`; `config.json` `suggestions`. An older install's single gateway (`config.gateway`,
secrets `anthropic_base_url` / `anthropic_auth_token`) is read as one entry and rewritten on the next
connection — that migration must keep working.

## Invariants a merge can break silently

- **`provider` travels with the session**: accepted by create/patch, returned by `public()`, carried into
  `runner.argv` / `spawn.claude_env`. `"gateway"` (no id) is the historical alias for the default gateway.
- **Gateway turns get `ANTHROPIC_BASE_URL` + `ANTHROPIC_AUTH_TOKEN` + the `ANTHROPIC_DEFAULT_*_MODEL` pins;
  Anthropic turns must never see them** (`spawn._drop_env_gateway`). `ANTHROPIC_BASE_URL` and
  `ANTHROPIC_AUTH_TOKEN` are reserved names in `spawn`'s env allow-list.
- **Effort is upstream's now** (`claude --effort`, `spawn.EFFORTS`, `EffortRow` in `ModelPicker`). Do not
  reintroduce the fork's old `spawn.effort_env` / `CLAUDE_CODE_EFFORT_LEVEL` path.
- **Suggestions stay a lazy i18n group** (`composer`/`settings`), never `core`: `test_web_first_load` caps
  `/sessions` at 20 000 bytes in French. `files` is not preloaded on `/sessions`; that exception is listed in
  `tests/test_web_i18n_groups.py` (`KNOWN`).
- **The upstream counter is informational only**: `upstream_behind` counts original-project commits not merged into
  the fork; `behind` still means this fork's updates and keeps the Update button. Keys stay in the `system` group.
- **`web/dist` is committed**: any `web/` change needs `npm run build` and the rebuilt `dist` in the same commit.
- **`README.md` is the fork's landing page**: its title is `# Harness Yann` and a block between
  `<!-- FORK: start -->` and `<!-- FORK: end -->`, right under the badges, states that this is an unofficial fork
  of Harness Maxxing and what it adds (the license forbids using their names for the fork beyond that). Upstream
  edits rarely touch those lines; when they do, keep them.
- **`POST /api/infra/exec` is documented in `docs/dev/API.md`**: upstream's contract test walks every route.
- The fork's features are announced in `CHANGELOG.md` under `### Added by this fork (not upstream)` inside the
  current version's section — the first `## ` heading must stay exactly the installed version (`test_docs.py`).

## Merge checklist

```bash
cd /home/yann/harnessyann/harness-maxxing
git fetch origin stable
git switch -c merge-upstream-<version> stable
git merge origin/stable            # resolve, see below
```

Conflicts almost always land in: `sessions/service.py`, `sessions/runner.py`, `sessions/spawn.py`,
`api/sessions.py`, `api/claude.py`, `web/src/pages/sessions/{Widgets,Composer,Column}.tsx`,
`web/src/lib/types.ts`, `web/src/lib/queries/*`, `web/src/i18n/{en,fr,groups}.ts`, `docs/dev/{API,INTERNALS}.md`.
Keep upstream's structure and re-add only the fork's plumbing (`provider`, `suggestions`, `infra/exec`).

Then:

```bash
cd web && npm run build && cd ..            # commits web/dist
cd server && uv run pytest -q tests/test_gateway.py tests/test_sessions.py tests/test_api_contract.py \
  tests/test_docs.py tests/test_webbuild.py tests/test_web_i18n_groups.py tests/test_web_first_load.py \
  tests/test_web_design_rules.py
uv run pytest -q                            # full suite; the environment kills long runs, split by file
```

Known flaky, deselect rather than chase: `tests/test_handoff_order.py::test_the_hold_expires_when_the_summary_never_comes`,
`test_dashboard.py::test_day_timeline`, `test_profile_photo.py`.

```bash
git switch stable && git merge --ff-only merge-upstream-<version>
git branch -f gateway-providers stable
git push fork stable gateway-providers      # never `origin`
```

On the server: `sudo -i` alone, then `maxxing update`, then `Ctrl+Shift+R` in the browser.
