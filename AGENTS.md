# Repository Guidelines

## Project Structure & Module Organization

This repository is the starting point for the IM Bot Adapter project:

- `README.md` contains the project title and should grow with setup notes.
- `docs/` is for architecture notes, adapter contracts, and platform-specific design documents.
- `pakages/contracts/` holds the shared contract package skeleton. Preserve the path for now; rename to `packages/` only in one coordinated change.
- `examples/` is for mock bridge services and sample platform payloads.
- `scripts/` is for local developer utilities.
- `test/` is for tests until package-level tests are introduced.

Expected modules include shared contracts, a Bridge client, common utilities, and adapters for Discord, Slack, Feishu/Lark, Telegram, and iMessage.

## Build, Test, and Development Commands

No package manager configuration exists yet. After adding `package.json`, prefer standard npm scripts:

- `npm install` installs dependencies.
- `npm run build` compiles TypeScript packages.
- `npm test` runs the full test suite.
- `npm run lint` checks formatting and code quality.
- `npm run dev:<platform>` runs a local adapter, for example `npm run dev:discord`.

Document any new command in `README.md` when adding it.

## Coding Style & Naming Conventions

Use TypeScript. Prefer two-space indentation, explicit exported types, and small platform-specific modules. Name files with kebab-case, for example `render-reply.ts`, `verify-signature.ts`, and `normalize-user-key.ts`. Use PascalCase for types such as `InboundIMMessage` and `BridgeReply`, and camelCase for functions and variables.

Keep platform differences inside adapter packages. Shared protocol types should live in the contracts package and be imported by adapters and Bridge client code.

## Testing Guidelines

Add tests alongside the behavior they validate or under `test/` while the repository is small. Use descriptive names such as `discord-adapter.test.ts` and `bridge-reply.test.ts`. Prioritize contract tests for message normalization, reply rendering fallbacks, idempotency, and signature verification.

Run `npm test` before submitting changes once test tooling is added.

## Commit & Pull Request Guidelines

This checkout has no Git history, so no existing commit convention can be inferred. Use concise imperative commit messages such as `add bridge reply contract` or `implement slack event normalization`.

Pull requests should include a short summary, test results, linked issue or task when available, and screenshots or payload examples for user-visible adapter behavior.

## Security & Configuration Tips

Never commit bot tokens, signing secrets, OAuth credentials, or webhook URLs. Add required variables to `.env.example` only as names, not real values. Validate incoming webhook signatures before normalizing events.
