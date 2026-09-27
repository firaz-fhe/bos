# Shared people and bot mentions

Run the focused regressions:

```sh
pnpm exec vitest run shared/shared-mentions.test.ts server/shared-room-trust.test.ts server/multiplayer-human-chat.e2e.test.ts server/multiplayer-mentions.e2e.test.ts src/components/shared-conversation.test.ts src/components/ChatView.controls.test.ts --maxWorkers=2
swift test --package-path ios --filter SharedConversationTests
```

The API tests launch disposable servers and fake providers. They prove:

- A person and bot named Maya have distinct qualified aliases. Bare ambiguous
  names, quoted examples and ordinary text invoke neither.
- A person mention records the human recipient and creates no bot request.
- Only current room members can be human recipients; only authorized bots can
  be bot recipients. Explicit bot selection controls which inferred bot runs.
- A retry after renaming a bot returns the original accepted message and keeps
  one request. Reusing that send id with changed content fails.
- Direct bot conversations retain their existing automatic reply behavior.

For actual desktop UI, launch `scripts/verify-shared-requests.ts` as described
in [shared requests](shared-requests.md). Its synthetic roster now includes
both a human Maya and Alex's bot Maya. In Launch crew, type `@Maya`, select
**Maya · person**, append a message and send. The composer clears; the stored
message has Maya's person id in `humanMentions`, an empty `botTargets` array,
and the request count remains two (the fixture's original requests).

The native `-shared-chat-preview` fixture includes the same name collision.
Type `@Maya`: choices show Maya · person / Person, Maya · Alex / Alex's bot,
and Willow / Maya's bot (owner search). Selecting the person inserts the
qualified alias and closes the suggestions. This native fixture intercepts
all traffic and does not send messages to a real teammate.

Both picker workflows and the desktop accepted human send were visually
verified on 28 September 2026. The native simulator build and core tests pass.
Real APNs delivery and the coordinated physical-device rollout remain separate
release checks; this evidence does not claim those are complete.
