# bos bot fork design

date: 2026-09-20
status: approved for planning

## outcome

bos bot is a focused, bot-first desktop application derived from the Apache-2.0 open-source edition of OpenMausBot. It should feel as direct as Grok Bot while allowing each bot to use Claude or Codex through the official local CLI and the user's existing subscription. The current BOS superapp remains a separate advanced operator cockpit.

## v1 boundary

v1 includes:

- a sidebar roster of bots and their conversations
- create, edit, duplicate, pin, hide, and delete bot flows
- per-bot name, avatar, instructions, working folder, engine, model, effort, and permission mode
- Claude and Codex subscription engines through installed and authenticated official CLIs
- streamed messages, tool activity, approval cards, attachments, interruption, resume, and conversation history
- local computer panel where supported, behind explicit opt-in and existing permission controls
- voice input and local macOS speech output where OpenMausBot already supports them without a paid service
- user-configured MCP servers, with the local AIOS knowledge MCP offered as a first-party preset
- BOS branding, icons, mascot, colors, typography, copy, package identifiers, and application name
- local-first storage and a signed macOS build

v1 excludes:

- hosted accounts, billing, public multi-user deployment, and organization administration
- mobile applications
- cloud computers and Box integration
- Composio, Pipedream, or a public connector marketplace
- routines, public webhooks, and unattended cloud execution
- OpenMausBot enterprise code
- merging BOS superapp pane, terminal, browser, editor, or dashboard systems into this app

## source and licensing

Create a new repository from OpenMausBot's open-source tree. Remove `enterprise/` before development. Preserve the Apache-2.0 license, copyright notices, dependency notices, and provenance required by bundled native components. Replace product trademarks and artwork, but do not imply affiliation with OpenMausBot, xAI, or Grok Bot.

Keep upstream history or an explicit upstream remote so security and protocol fixes can be reviewed and selectively integrated. BOS-specific work should remain in small commits that separate branding, feature removal, AIOS integration, and security changes.

## architecture

Retain OpenMausBot's two-process boundary:

1. Electron and React render the desktop interface.
2. A loopback-only Node harness owns agent processes, model discovery, transcripts, approvals, MCP configuration, and the normalized event stream.

The frontend sends typed HTTP commands and folds a single SSE stream into UI state. Provider-specific logic stays behind the driver contract. Claude uses its official CLI streaming protocol. Codex uses the official app-server JSON-RPC protocol. The UI never parses either provider's native wire format.

Add a BOS configuration layer for product defaults, brand assets, bundled MCP presets, and feature flags. Avoid scattering BOS conditionals through upstream modules. Features removed from v1 should be disabled through bounded configuration or clean module removal, not hidden with CSS.

## bot and conversation model

A bot owns its identity and defaults. A conversation owns its transcript and provider resume cursor. Changing the selected model affects the next turn and does not rewrite earlier messages. If native resume fails, the app must explain whether it can safely recover by replaying bounded context or must start a fresh conversation.

Each bot stores:

- identity and avatar
- system instructions
- engine and model selection
- reasoning effort
- permission mode
- working directory
- enabled MCP servers
- voice settings

Secrets and provider credentials must not be copied into bot records or renderer-visible state.

## subscription model access

Claude and Codex are available only when their official CLIs are installed and authenticated. BOS bot detects them, reports the executable and version, reads the supported model catalog where the CLI exposes it, and clearly distinguishes unavailable, unauthenticated, quota-limited, and protocol-incompatible states.

The app must not scrape web sessions, proxy consumer OAuth tokens, or imitate provider clients. API-key and custom engines remain possible through OpenMausBot's existing extension points, but they are secondary to official CLI subscription access in v1.

## aios knowledge integration

Ship an optional preset for the local AIOS MCP server. Enabling it adds the server through the normal MCP registry; it does not bypass the server's read policy. The preset must use a configurable Node executable and server path rather than embedding Firaz's home-directory paths in distributable code.

The renderer shows configured and connected as separate states. A successful process spawn is not proof that knowledge calls work. Verification requires tool discovery plus one allowed lookup and one policy-refused lookup.

## brand application

Use the supplied kit at `aios-firaz/outputs/2026-09-20-bos-brand-kit` as the visual source of truth. Integrate:

- `build/bos.icns` for macOS packaging
- the appropriate app-icon raster set for secondary surfaces
- the BOS mascot and mark for bot defaults and empty states
- light and dark lockups for onboarding and about surfaces
- `tokens/bos-tokens.css` as the basis for semantic UI tokens

Preserve Grok Bot's strong information hierarchy without copying proprietary assets or source: narrow identity rail, bot list, focused conversation, contextual settings sheet, and one dominant composer.

## safety and privacy

Before enabling computer control or full-access execution:

- bind the harness only to loopback by default
- require explicit user action for remote exposure
- verify renderer isolation, preload boundaries, CSP, navigation policy, and external-link handling
- keep secrets out of argv, logs, transcripts, crash reports, and renderer state
- store secrets in the macOS Keychain where feasible; do not accept plaintext persistence as the final shipped design
- default bots to the least permissive useful mode
- preserve one-time approval semantics and never silently upgrade them into durable approval
- display when a bot is controlling the computer and prevent concurrent controllers
- pin and verify any downloaded native automation artifacts
- disable telemetry until its schema and consent surface are reviewed

## failure handling

The app should turn failures into explicit product states:

- missing CLI: show installation guidance without automatically installing software
- expired authentication: show the provider's supported login command
- rate limit: preserve the draft and conversation, show the reset information when available
- agent crash: preserve received events and offer a bounded retry
- lost resume cursor: explain recovery options before replaying context
- MCP failure: identify the specific server without breaking the base conversation
- computer-use failure: stop control, release the session lock, and retain an audit trail
- migration failure: keep the prior data readable and do not partially overwrite it

## verification

The fork is ready for personal v1 use only after:

- license and third-party notice review passes
- unit and integration tests from upstream pass after feature removal
- Claude and Codex each complete a real subscription-backed conversation
- both engines stream tool activity and handle allow/deny approval correctly
- switching model, effort, and permission mode affects only subsequent turns
- conversations survive app restart and resume correctly
- AIOS MCP passes discovery, allowed-read, and refused-read checks
- local computer control is opt-in, visible, interruptible, and single-controller
- no provider token or MCP secret appears in renderer state, logs, argv, or exported transcript
- the packaged macOS application launches with BOS identity and correct signing metadata

## delivery sequence

1. establish the fork, provenance, license files, and upstream remote
2. remove enterprise and out-of-scope cloud modules
3. apply BOS product identity and design tokens
4. reduce onboarding to Claude/Codex subscription detection and bot creation
5. wire the AIOS MCP preset
6. harden secrets, Electron boundaries, loopback transport, and computer-use controls
7. run provider, persistence, approval, MCP, packaging, and visual verification

## deferred decisions

Public distribution, commercial hosting, mobile clients, cloud computers, routines, connector marketplaces, and team administration require separate designs. They must not expand v1 implicitly.
