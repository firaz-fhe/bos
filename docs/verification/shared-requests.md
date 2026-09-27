# Shared request answers

Run from the repository with Node 24 or newer:

```sh
node --experimental-strip-types scripts/verify-shared-requests.ts
```

The launcher creates a disposable home, synthetic Alex and Maya identities,
and two bots using the fake provider. It creates a group containing only the
two people, sends one request to each bot, and requires two responses with
request references before printing the preview URL. No real credentials or
live user data are used.

Open the printed preview in the browser and select **Launch crew**.

- Pepper's answer must quote the checklist request.
- Willow's answer must quote the onboarding request.
- Each answer must show its bot and owner.
- Clicking Willow's quote must highlight the onboarding request.
- Completed activity must not leave the conversation looking busy.
- The group must still contain only Alex and Maya.

Stop the launcher with Ctrl-C. Its exact fixture server and temporary home
are cleaned up; the printed server log remains as evidence.

Focused regressions:

```sh
pnpm exec vitest run src/components/shared-conversation.test.ts src/components/ChatView.controls.test.ts --maxWorkers=2
swift test --package-path ios --filter SharedConversationTests
```

The regressions cover activity replacement without mixing owners or requests,
request reference preservation, completed labels and active work after an
unrelated human message. They do not prove queued-request cancellation,
approval state routing, real remote-host delivery or iPhone rendering. Those
remain separate acceptance requirements.
