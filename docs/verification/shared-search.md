# Shared conversation search

Use the disposable server and renderer fixture:

```sh
node --experimental-strip-types scripts/verify-shared-requests.ts
```

It creates synthetic people, fake-provider bots, 205 intervening messages and
an older `Original café + plan` message. No real credentials or live data are used.

1. Open Launch crew, then Find in conversation or Cmd/Ctrl+F. The search field
   must receive focus. It searches retained history, not just mounted messages.
2. Search `café +`. Open the one result. The older original message must be loaded,
   highlighted and visible with neighbouring history and a return-to-latest control.
3. Reopen search. Select Bot results; only the two request-linked bot replies
   appear, labelled with their bot and owner. People, files, links and author
   filters narrow the same room history. Empty results explain how to recover.
4. Stop with Ctrl-C to clean up the owned server and temporary home.

For iOS, build Debug and launch `-shared-chat-preview` on an owned disposable
simulator, as described in [shared requests](shared-requests.md). Open Launch crew
and its details, then Search conversation. Verify Bot results shows Pepper · Alex;
Files shows Maya's original launch board. Selecting that result loads message 1
from beyond the initial 200-message window. Its text and attachment must be fully
visible below the header. Do not install physical devices for this check.

Focused checks:

```sh
pnpm exec vitest run server/shared-room-repository.test.ts server/multiplayer-links.e2e.test.ts server/request-auth.test.ts companion/test/routes.test.ts src/components/ChatView.controls.test.ts --maxWorkers=1
swift test --package-path ios --filter ClientTests
```

Repository tests exercise retained history beyond the initial page, Unicode
normalization, literal punctuation, source-sequence pagination, each content
filter, exact author IDs, updated text, removed messages and membership revocation.
The linked-home test forwards encoded Unicode queries through an authenticated
fixture and rejects unknown routes/filters and deleted-room access. Native client
tests cover encoded plus signs and result/cursor decoding.

On 27 September 2026 the desktop and native simulator walkthroughs passed;
142 targeted JS tests and 55 native client tests passed. Both type checks and the
Debug simulator build passed. This is staged acceptance, not installed-device or
public distribution acceptance.
