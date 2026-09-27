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
- Open **Conversation details → Bot requests**. Both cards show Completed,
  the bot owner and requester, with working **Show request** and **Show result** links.
- Completed cards have no Stop button. Native iOS exposes the same list under
  conversation details; native rendering needs a simulator/device check.

Stop the launcher with Ctrl-C. Its exact fixture server and temporary home
are cleaned up; the printed server log remains as evidence.

Focused regressions:

```sh
pnpm exec vitest run server/shared-request-store.test.ts server/multiplayer-requests.e2e.test.ts server/multiplayer-restart.e2e.test.ts server/multiplayer-links.e2e.test.ts server/request-auth.test.ts companion/test/routes.test.ts src/components/shared-conversation.test.ts src/components/ChatView.controls.test.ts --maxWorkers=2
swift test --package-path ios --filter SharedConversationTests
```

The regressions cover activity replacement without mixing owners or requests,
request reference preservation, completed labels and active work after an
unrelated human message. The isolated request API fixture proves duplicate-send
idempotency, requester/owner cancellation rules, rejection of another member's
stop, queued cancellation without dispatch, cancellation after editing the source,
result references, pagination and loss of reads after leaving.

The restart fixture kills only its owned disposable server. Previously dispatched
work becomes Outcome unknown and is never replayed by retrying the original send.
Work still queued before dispatch may resume. A new deliberate request may run.
Published replies are reconciled before dispatch receipts, covering the crash
window between the room journal append and request completion write.

Cancellation means a stop was requested and no later result will be published;
it cannot undo completed actions. Request metadata contains no provider transcript,
credential, private prompt or approval arguments. Reads require current room
membership; cancellation additionally requires the exact requester or bot owner.
The request ledger fails closed if corrupt, so missing dispatch evidence never
silently becomes permission to repeat work.

Approval resolution, actual remote-host cancellation, concurrency/load limits and
iPhone rendering still need their separate acceptance checks. A passing simulator
build is not native UI acceptance.
