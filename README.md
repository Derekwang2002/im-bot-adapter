# IM Bot Adapter

Thin adapter layer for connecting IM platform bots to the CALL-E Bridge.

The adapter receives platform events, extracts platform identity, normalizes messages into a shared contract, sends them to Bridge, and renders Bridge replies back to the source IM platform. It does not own CALL-E OAuth, authorization state, preview, confirmation, call execution, or webhook business logic.

## Target Platforms

- Discord
- Slack
- Feishu / Lark
- Telegram
- iMessage

## Repository Layout

```text
docs/      Development plans and architecture notes
examples/  Mock services and sanitized platform payloads
pakages/   Package skeletons for contracts and adapters
scripts/   Local developer utilities
test/      Tests while the repository is still small
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
