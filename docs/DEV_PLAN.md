# IM Bot Delivery Plan

## 1. Goal and Scope

The current phase is to create and configure mainstream IM platform bots. Work should focus on developer console setup, bot/app permissions, webhook or event callback configuration, local smoke testing, and payload fixture capture.

Adapter implementation is temporarily out of scope. Shared contracts, Bridge client code, message normalization, reply rendering, CALL-E OAuth, preview, confirmation, call execution, and webhook business logic belong to a later Adapter / Bridge phase.

## 2. Architecture Boundary

```text
IM Platform Bot
  -> webhook / event callback smoke endpoint
  -> sanitized payload fixture
  -> later Adapter / Bridge phase
```

Current deliverables:

- Created platform bot/app.
- Required permissions and scopes recorded.
- Webhook/event callback connected to a local or deployed smoke endpoint.
- Minimal send/receive or interaction behavior verified.
- Sanitized event fixtures saved for future adapter development.

Do not build platform adapter packages yet unless needed for a minimal smoke endpoint.

## 3. Proposed Repository Structure

```text
docs/
  platform-notes/
    discord.md
    slack.md
    feishu.md
    telegram.md
    imessage.md

examples/
  payloads/

scripts/
  smoke-webhook/

test/
  smoke/
```

The existing `pakages/` skeleton is not part of the active scope. Keep it untouched until the Adapter phase starts.

## 4. Delivery Milestones

### M0: Project Foundation

- Add `.env.example` with bot credential variable names only.
- Add `docs/platform-notes/` templates for each platform.
- Add a fixture redaction checklist.
- Add a minimal webhook smoke endpoint only if needed for event verification.

Exit criteria:

- Setup notes exist for every target platform.
- No real token, signing secret, webhook URL, phone number, or user email is committed.

### M1: Shared Bot Setup Artifacts

- Define standard platform note structure: app creation, scopes, webhook URL, local tunnel, test account, known risks.
- Define fixture naming and redaction rules.
- Document local tunneling choices such as `ngrok` or Cloudflare Tunnel.
- Document manual smoke test steps.

Exit criteria:

- A contributor can create a new platform note without guessing required sections.
- Fixture examples are sanitized and reproducible.

### M2: Discord Bot P0

- Create Discord application and bot user.
- Enable required intents and interactions.
- Configure slash commands and local callback URL.
- Invite bot to a test server with minimum permissions.
- Capture sanitized mention, slash command, and button payloads.

Exit criteria:

- Mention and slash command reach the smoke endpoint.
- A simple interaction response can be returned.
- Bot-authored messages are observed and documented for future filtering.

### M3: Slack Bot P0

- Create Slack app.
- Enable Events API, slash commands, interactivity, and required bot scopes.
- Install app to a test workspace.
- Configure event and action callback URLs.
- Capture sanitized event, slash command, and Block Kit action payloads.

Exit criteria:

- Slack URL verification passes.
- Events and actions reach the smoke endpoint.
- Slack response timing constraints are documented.

### M4: Feishu / Lark and Telegram Bots P1

- Create Feishu/Lark internal app and configure message events / card callbacks.
- Create Telegram bot with BotFather and configure webhook.
- Capture sanitized message and interaction payloads for both platforms.
- Document permission, event subscription, privacy mode, and deployment constraints.

Exit criteria:

- Feishu/Lark message events reach the smoke endpoint.
- Telegram text updates reach the smoke endpoint.
- Required production approval or public URL constraints are documented.

### M5: iMessage Feasibility P2

- Choose a feasible iMessage automation approach.
- Document host OS, account, phone/handle identity, and operational limitations.
- Verify basic receive/send if the target environment allows it.

Exit criteria:

- iMessage is either verified with a minimal flow or explicitly marked blocked with reasons.

### M6: Adapter Handoff

- Summarize payload shapes, identity fields, reply capabilities, retry behavior, and rate limits.
- Create backlog items for adapter contracts and implementation.
- Keep adapter code out of this phase unless approved separately.

Exit criteria:

- Future adapter work can start from documented platform setup and sanitized fixtures.

## 5. Platform Bot Setup Plan

This project currently prioritizes the creation and configuration of each IM bot application. Adapter code is deferred.

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
- Smoke test proving the platform bot can send events to the configured endpoint.

## 6. Testing Plan

### Test Layers

- Manual smoke tests: prove each platform bot can send events to the configured endpoint.
- Fixture checks: confirm captured payloads are sanitized and stored under `examples/payloads/`.
- Configuration checks: verify required scopes, event subscriptions, callback URLs, and local tunnel settings are documented.
- Security checks: confirm no real token, signing secret, webhook URL, phone number, or user email is committed.
- Deferred tests: adapter unit, contract, Bridge client, and reply rendering tests belong to the later Adapter phase.

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

- Bot/app can be created by following the platform note.
- Required scopes and callback settings are documented.
- Event delivery reaches the smoke endpoint.
- Interactive callback delivery is verified where the platform supports it.
- Fixture redaction is complete before committing.
- Known blockers are recorded in the platform note.

### Suggested Commands

Once tooling exists, standardize these scripts:

```bash
npm run smoke:webhook
npm run fixtures:check
npm run lint
```

All milestone PRs should include manual smoke test evidence and fixture names.

## 7. Implementation Rules

- Do not implement full Adapter behavior in this phase.
- Do not store CALL-E OAuth tokens or platform bot secrets in the repository.
- Do not put CALL-E call execution logic in bot setup scripts.
- Keep platform notes specific enough that another contributor can recreate the bot setup.
- Capture fixtures only after redacting tokens, user identifiers, emails, phone numbers, and webhook URLs.
- Record platform limits such as callback timeouts, public URL requirements, app review needs, and rate limits.

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

### 2026-06-11 - Re-scope Plan to Bot Creation

- Changed: reframed the plan and README around platform bot creation and configuration; moved adapter implementation, Bridge client, and contract work into a deferred phase.
- Tested: verified the source Feishu document reflects the bot-first scope at revision 35.
- Notes: update was made after clarifying that the primary work is bot creation, not adapter development.

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
