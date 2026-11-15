# Notice

**Harness Yann** is an unofficial fork of [Harness Maxxing](https://github.com/makushingu/harness-maxxing),
based on upstream `2.0.0-alpha.151` (commit `dd4c7a473`). It is **not** Harness Maxxing, is not maintained by
the Harness Maxxing authors, and its name is not a claim on theirs. The application itself keeps the name
Harness Maxxing, as the software it is.

`LICENSE` is the Harness Maxxing Fair Source License 1.0, **unchanged**, copyright notice included, as its
section 4 requires. This fork is modified, and the changes are listed below. Under that license the fork may be
used, copied, modified and redistributed free of charge; it may not be sold, rented or licensed, included in a
paid product or a paid service, or offered as a hosted or managed service for a fee, and the names
"Harness Maxxing", "Maxxing" and their logos may not be used for the fork beyond stating truthfully that it is
based on Harness Maxxing.

## Changes

Three features are kept on top of upstream, merged with every upstream release (see `FORK.md` for the files,
the invariants and the merge checklist).

**Several LLM gateways at once.** An Anthropic-compatible endpoint can be connected beside a subscription token
or an API key instead of replacing it: OpenCode Zen, OpenCode Go, OpenRouter, or a custom gateway, one of each.
Each session picks its credential with its model, the picker lists every side's models tagged with the gateway's
name, Settings -> Claude tests and removes each connection on its own, and the default provider (new sessions,
session titles, Team summaries) is one of them. An install that already had the single gateway is migrated on
first start.

- `server/maxxing/gateway.py` (new): presets, base-URL validation, connection state (`config.json` `gateways`,
  `secrets.json` `gateway_key_<id>`), the turn environment (`ANTHROPIC_BASE_URL`, `ANTHROPIC_AUTH_TOKEN`,
  `ANTHROPIC_API_KEY=""`, the `ANTHROPIC_DEFAULT_{OPUS,SONNET,HAIKU}_MODEL` role pins), the model ids for the
  picker and for background calls.
- `server/maxxing/api/claude.py`: `POST /api/claude/auth` accepts gateways; `GET /api/claude/status` exposes
  them; the test call uses the gateway's cheapest model; the `suggestions` setting.
- `server/maxxing/sessions/{spawn,runner,service}.py`: the `provider` of a session travels from create/patch to
  `public()` and to the `claude` process environment; an Anthropic turn never inherits a gateway set in the
  service environment.
- `web/src/…`: the **Gateway** tab of Settings -> Claude, the gateway tags in the model picker, types and
  translations (English, French).

**Follow-up suggestions above the message box.** `GET /api/sessions/{id}/suggestions` returns two or three short
messages written by the smallest model from the end of the conversation, cached per `(session, turns, log size)`
and locked per session; the UI shows them above the composer and one click sends the message. They can be turned
off in Settings -> Claude (`config.json` `suggestions`, default `true`).

- `server/maxxing/sessions/suggest.py` (new), `server/maxxing/api/sessions.py`, `web/src/pages/sessions/…`.

**Run a command on a server from the Infra page.** `POST /api/infra/exec` runs a command on a listed server and
returns its exit code, output and duration; `web/src/features/infra/RunCommand.tsx` is the dialog.

Documentation, tests and the built UI (`web/dist`, committed as upstream does) are updated with them:
`docs/dev/API.md`, `docs/dev/INTERNALS.md`, `docs/user/04-sessions.md`, `docs/user/11-settings-and-keys.md`,
`server/tests/test_gateway.py`, `server/tests/test_sessions.py`, `CHANGELOG.md`, `FORK.md`.

## Installing this fork

```sh
REPO=yann627892/harness-yann BRANCH=stable sh install.sh
```

`maxxing update` then follows this fork's `stable` branch, which is upstream's `stable` plus the changes above.
