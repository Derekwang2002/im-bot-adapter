# IM Bot Adapter

Bot-first workspace for creating and configuring IM platform bots that will later connect to the CALL-E Bridge.

The current focus is platform bot creation: developer console setup, permissions, webhook/event callback configuration, local smoke testing, and sanitized payload fixtures. Adapter implementation, Bridge client code, shared contracts, OAuth binding, and CALL-E business logic are deferred.

## Target Platforms

- Discord
- Slack
- Feishu / Lark
- Telegram
- iMessage

## Repository Layout

```text
docs/      Development plans, platform notes, and setup records
examples/  Sanitized platform payloads and later mock services
pakages/   Existing package skeletons; adapter work is deferred
scripts/   Local developer utilities
test/      Smoke checks while the repository is still small
```

Note: `pakages/` is the current directory name. If it is renamed to `packages/`, update docs, imports, scripts, and CI in the same change.

## Development

Tooling has not been added yet. The intended command set is:

```bash
npm install
npm run build
npm test
npm run lint
```

See [docs/DEV_PLAN.md](docs/DEV_PLAN.md) for the implementation roadmap, testing plan, and changelog process.
