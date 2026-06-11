# IM Bot Adapter Development Plan

## 1. Goal and Scope

Build a thin IM Bot Adapter layer that connects mainstream IM platforms to the Bridge layer. The adapter is responsible for platform event ingestion, identity extraction, message normalization, Bridge communication, and rendering Bridge replies back to the source platform.

The adapter must not own CALL-E business logic. OAuth binding, authorization state, preview, confirmation, call execution, and long-running webhook updates belong in Bridge / CALL-E services.

## 2. Architecture Boundary

```text
IM Platform Bot
  -> Bot Adapter
  -> Bridge API / Event Bus
  -> CALL-E auth + SDK/WebHook layer
```

Core contracts:

- `InboundIMMessage`: normalized inbound message sent from adapters to Bridge.
- `BridgeReply`: platform-neutral reply returned by Bridge.
- `platformUserKey`: stable user key, for example `discord:user:123456789`.
- `conversationId`: reply path only; it must not be used as the authorization key.

Platform-specific differences must stay inside adapter packages: event format, signature verification, message send API, interactive UI support, retry behavior, and rate limits.

## 3. Proposed Repository Structure

```text
docs/
  architecture.md
  adapter-contract.md
  platform-notes/

pakages/
  contracts/
  bridge-client/
  shared/
  discord-bot/
  slack-bot/
  feishu-bot/
  telegram-bot/
  imessage-adapter/

examples/
  bridge-mock/
  payloads/

test/
  contract/
  adapters/
```

Note: the current repository uses `pakages/`. Keep it until the project decides to rename it to `packages/`; if renamed, update imports, documentation, scripts, and CI in one change.

## 4. Delivery Milestones

### M0: Project Foundation

- Add TypeScript workspace configuration.
- Add package manager scripts for build, test, lint, and local development.
- Add `.env.example` with variable names only.
- Add shared logging and configuration conventions.

Exit criteria:

- `npm run build`, `npm test`, and `npm run lint` are available.
- Empty package skeletons compile.

### M1: Shared Contracts and Bridge Client

- Define `InboundIMMessage`, `BridgeReply`, platform enum/types, and action schema.
- Implement `platformUserKey` helpers.
- Implement Bridge HTTP client with timeout, retry, and typed errors.
- Add mock Bridge server for local adapter testing.

Exit criteria:

- Contract tests validate required fields and invalid payloads.
- Bridge client tests cover success, timeout, retry, and error responses.

### M2: Discord Adapter P0

- Receive mention and slash command events.
- Verify Discord interaction signatures.
- Normalize user, channel/thread, message, timestamp, and text.
- Render text, auth links, confirm/cancel actions, and error replies.
- Prevent bot message loops and deduplicate retries.

Exit criteria:

- Discord payload fixtures pass normalization tests.
- Local adapter can call mock Bridge and render a reply.

### M3: Slack Adapter P0

- Support Events API, slash commands, and Block Kit actions.
- Verify Slack request signatures.
- Normalize team, user, channel, thread, message, timestamp, and text.
- Respect Slack's short acknowledgement window; defer Bridge work when needed.
- Render Block Kit cards and fallback text.

Exit criteria:

- Slack fixtures pass normalization and signature tests.
- Interactive actions produce expected `BridgeReply` callbacks.

### M4: P1 Adapter Skeletons

- Add Feishu/Lark, Telegram, and iMessage skeleton packages.
- Document platform-specific auth, webhook, and identity risks.
- Implement minimal text-only flow where practical.

Exit criteria:

- Skeletons compile.
- Platform notes document required credentials, event types, and known limitations.

### M5: Hardening and Release Readiness

- Add structured logs, request IDs, and correlation IDs.
- Add platform-level rate limit and retry handling.
- Add deployment notes and operational runbook.
- Add CI for build, lint, and tests.

Exit criteria:

- CI passes on a clean checkout.
- README documents local setup and supported platforms.

## 5. Platform Bot Setup Plan

This project must track the creation and configuration of each IM bot application, not only adapter code. Treat bot setup as a release prerequisite for each platform.

### Shared Setup Requirements

- Create the platform bot/app in the platform developer console.
- Record required credentials in `.env.example` by name only.
- Configure webhook or event callback URLs for local and deployed environments.
- Enable only the minimum scopes needed for message receive, reply send, and interactive actions.
- Document local tunneling requirements, for example `ngrok` or Cloudflare Tunnel.
- Add a sanitized payload fixture after each event type is verified.

### Platform Checklist

| Platform | Setup Tasks | Required Validation |
|-|-|-|
| Discord | Create Discord application and bot user; enable interactions; configure slash commands; invite bot with message and command permissions. | Mention and slash command reach the adapter; buttons render; bot messages are ignored. |
| Slack | Create Slack app; enable Events API, slash commands, interactivity, and required bot scopes; install to workspace. | URL verification passes; event and action signatures verify; adapter responds within Slack timing limits. |
| Feishu / Lark | Create internal app; enable message events and card callbacks; configure event subscription and app permissions. | Message event reaches webhook; open_id, chat_id, and message_id normalize correctly; interactive card callback works. |
| Telegram | Create bot with BotFather; configure webhook; decide privacy mode for groups; define inline keyboard behavior. | Text updates reach webhook; user_id and chat_id normalize correctly; inline keyboard callback works. |
| iMessage | Select implementation approach and deployment host; define phone/handle identity fallback; document operational limits. | Plain text receive/send works in the target environment; fallback identity is stable enough for Bridge binding. |

### Setup Deliverables

- `docs/platform-notes/<platform>.md` with app creation steps, scopes, webhook settings, and risks.
- `examples/payloads/<platform>-*.json` sanitized fixtures.
- `.env.example` entries for required variables.
- Adapter smoke test against `examples/bridge-mock`.

## 6. Testing Plan

### Test Layers

- Unit tests: pure functions such as normalization, key generation, reply rendering, and signature verification.
- Contract tests: validate `InboundIMMessage` and `BridgeReply` schema compatibility.
- Adapter tests: feed real platform payload fixtures into adapters and assert normalized output.
- Bridge client tests: use mocked HTTP responses for success, failure, timeout, and retry behavior.
- Integration tests: run an adapter against `examples/bridge-mock` and verify outbound platform renderer calls.

### Fixture Strategy

Store sanitized payloads under `examples/payloads/`, grouped by platform:

```text
examples/payloads/
  discord-message.json
  discord-interaction.json
  slack-event.json
  slack-action.json
  feishu-message.json
  telegram-update.json
  bridge-reply-auth-link.json
  bridge-reply-confirm.json
```

Fixtures must not contain real tokens, user emails, phone numbers, webhook URLs, or workspace identifiers.

### Required Coverage Areas

- Identity extraction binds authorization to the sender, not the channel or group.
- Bot-authored events are ignored.
- Retry events are idempotent.
- Unsupported UI actions degrade to text.
- Signature verification rejects invalid or stale requests.
- Bridge errors render user-readable platform replies.

### Suggested Commands

Once tooling exists, standardize these scripts:

```bash
npm run build
npm test
npm run test:contract
npm run test:adapters
npm run lint
```

All milestone PRs should include the relevant command output in the PR description.

## 7. Implementation Rules

- Keep adapters thin: `platform event -> InboundIMMessage` and `BridgeReply -> platform message`.
- Do not store CALL-E OAuth tokens in adapter packages.
- Do not put CALL-E call execution logic in platform adapters.
- Validate webhook signatures before parsing or trusting payloads.
- Use `messageId` or platform interaction IDs for idempotency.
- Use `correlationId` across adapter logs and Bridge calls.
- Provide text fallback for every interactive reply type.

## 8. Changelog Process

Every code or documentation change must update the changelog below in the same PR or commit. Add newest entries at the top.

Format:

```markdown
### YYYY-MM-DD - Short Title

- Changed: concise description of what changed.
- Tested: commands or checks performed.
- Notes: migration notes, follow-ups, or known gaps.
```

If no test command exists yet, write `Tested: not run; tooling not added yet`.

## 9. Changelog

### 2026-06-11 - Add Platform Bot Setup Plan

- Changed: added explicit platform bot creation and configuration plan covering Discord, Slack, Feishu/Lark, Telegram, and iMessage; updated `.gitignore` to ignore generated Lark authorization QR images.
- Tested: verified the section exists in the Feishu document at revision 15.
- Notes: the same section was also added to the source Feishu project document.

### 2026-06-11 - Prepare Repository for Git

- Changed: added `.gitignore` for local authorization QR artifacts and `.gitkeep` files so empty project directories are preserved in Git.
- Tested: not run; repository metadata change only.
- Notes: GitHub remote still depends on available authentication or a target repository URL.

### 2026-06-11 - Update README

- Changed: expanded README with project purpose, target platforms, repository layout, intended commands, and development plan link.
- Tested: not run; documentation-only change.
- Notes: package tooling is still not present.

### 2026-06-11 - Add Development Plan

- Changed: created the initial development plan, milestone breakdown, testing strategy, implementation rules, and changelog process.
- Tested: not run; documentation-only change.
- Notes: project tooling and package structure are still skeletal.
