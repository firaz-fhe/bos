# Free-model setup renderer smoke

Use the documented disposable [chat UI harness](chat-ui.md), not an installed app or live server:

```sh
node --experimental-strip-types scripts/control-omb.ts ui launch --tool-calls '[]'
```

In a second terminal, pass the exact printed handle:

```sh
node --experimental-strip-types scripts/verify-free-model-ui.ts /tmp/openmausbot-verify-data-XXXXXX/ui.json
```

The recipe runs the real Settings, onboarding and model-picker components. It installs a browser-side fixture boundary before interacting with the connection form: the first setup request returns a synthetic 502, later requests return a synthetic connected free-model inventory. All external `fetch` calls are blocked. The synthetic key never reaches the server setup endpoint or OpenRouter. This isolates rendering and state transitions; real persistence, authorization and provider-update locking are covered separately by `server/free-model-setup.e2e.test.ts`.

The assertions verify password input with autocomplete off, a retryable failure, successful inventory refresh, key clearing after connection in Settings and onboarding, a Cloud rail containing the free provider, and no synthetic key in localStorage, chat, API config output or visible text. Screenshots capture failure, connected Settings, cloud picker and connected onboarding.

Evidence is written to `.omb-scratch/verify-evidence/free-model-ui/`:

- `settings-retry.png`
- `settings-connected.png`
- `picker-cloud.png`
- `onboarding-connected.png`
- `result.json` with assertions and the explicit synthetic boundary

Run on 28 September 2026: assertions passed; screenshot review confirmed masked input, visible error/retry, connected inventory and onboarding. Browser console had no error entries. Initial inspection found a short-viewport picker layout issue. After the compact reconnect fix, the final rerun passed an added DOM clipping assertion: the full model row was visible at the harness viewport, and the updated screenshot was visually checked.

Stop the launcher with Ctrl-C after capturing evidence. It shuts down only its owned fixture/browser and removes its disposable data. Keep its printed server log path with the result. No provider authentication, live inference, installation or deployment is established by this renderer smoke.
