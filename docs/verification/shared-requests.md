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
  conversation details.

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

## Native offline fixture

Build the Debug iOS app and install it only on an owned disposable simulator.
Launch with `-shared-chat-preview`, `-companion.prefs.islandIntro never`,
`-companion.onboarding.welcomeSeen YES` and `-companion.onboarding.notificationsSeen YES`.
This debug-only URL protocol intercepts every API call; the background host event
stream is disabled. It uses synthetic Alex/Maya identities and no real credentials.

Open Launch crew. The fixture has 220 messages, a bundled image attached to message
1, two eligible bots and three request cards: completed, another person's working
request and your queued request. Check:

- Typing @ shows Pepper/Alex and Willow/Maya; selection retains the draft.
- Shared files opens launch-board.png in native preview. Show message loads and
  navigates to the older original message, preserving the draft.
- Bot requests shows owner/requester and correct state. Only your queued request
  has Stop. Stopping changes it to Stopped with the before-start explanation.
- Show result dismisses details and returns to Pepper's result. A latest-message
  button must not linger when the conversation is already at its end.
- Test the software keyboard, bottom-follow when mention chips change height,
  and intentional scrollback. Record gesture tests as unverified if simulator
  automation cannot deliver a usable scroll gesture.

Native preview, older file source navigation, request controls, result navigation,
mention selection and keyboard avoidance were visually checked on the disposable
simulator on 27 September. Deliberate drag scrollback still needs verification.
Approval resolution, actual remote-host cancellation and concurrency/load limits
remain separate checks. A passing simulator build is not device acceptance.

## Restoring backups

`server/workspace-backup.test.ts` and `server/shared-request-store.test.ts` check
that restoration retains shared history and completed results, cancels old queued
work, and marks previously dispatched work Outcome unknown. Per-room sequence
barriers suppress legacy recovery, including the message-before-ledger crash
window. Only new deliberate requests may run after a restore.


## Shared progress and remote cancellation

The room receives fixed working/step/approval status text. Provider tool names,
commands, paths and approval arguments are not copied into shared progress.
`shared/shared-request.test.ts` checks that boundary. The isolated request API
fixture also checks that a private tool command does not enter shared history.

`server/remote-bot-bridge.test.ts` uses a fake linked home to prove cancellation
sends an interrupt only to the exact shared thread, leaving the owner's private
thread untouched. Waiting-for-owner progress names the owner without exposing
the approval tool/arguments or automatically resolving it.

The request API fixture runs overlapping requests to the same bot in two rooms:
the second room stays queued, a cancel addressed through the wrong room is
rejected, and the correctly cancelled queued request never dispatches. These are
isolated protocol checks; the final two-host device acceptance remains required.

## Shared bot appearance

Run `node --experimental-strip-types scripts/verify-shared-mascots.ts` for a
separate fake-provider server and ordinary App preview. Open the printed
`previewUrl`, then the **Mascot acceptance** conversation. Pepper must show an
orange bear and Willow a pink cat in their working rows, even after the human
message. POST to the printed `finishUrl` (fixture only) to release both replies;
their author avatars must keep those bodies. The room header remains human.
Ctrl-C closes the preview and disposable server. `--smoke` checks the roster
and preview response, then closes; it does not assert browser rendering.

`ChatView.controls.test.ts` also renders the actual chat component to verify
body paths/colors, simultaneous speakers, uploaded bot images, and the person
photo independently of the room's placeholder profile.
