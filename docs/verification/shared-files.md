# Shared files and source navigation

Start a disposable fake-provider workspace:

```sh
node --experimental-strip-types scripts/verify-shared-files.ts
```

Use the printed preview URL. The fixture has two human members, two published
text files and 205 intervening messages. It never connects to the user's data.

1. Open Launch crew and Conversation details. Shared files lists both files,
   newest first, with author, date, size, download and Show message controls.
2. Choose Show message for launch-brief.txt. The original source is outside the
   initial 200-message page. The controller loads contiguous earlier history,
   closes details and lands on the original attachment message on the first tap.
3. Jump to latest, reopen details and download a file. Bytes are requested only
   when opened; the file index itself contains metadata and source IDs.
4. Check the same index on iPhone under Conversation details > Shared files.
   Open a file in the native preview and use Show message to return to its source.
   Native compilation and core client tests do not substitute for this UI check.

The 2026-09-27 desktop visual check verified steps 1–3 in Chrome against the real
renderer and isolated server. The downloaded launch-brief.txt matched the original
49-byte fixture contents. It caught and corrected a synthetic scroll event
that re-enabled bottom-follow before source navigation completed. Native iPhone52 visual inspection verified the Shared files empty state after
reconnection. A populated native file preview and source jump remain pending.

Contract checks:

```sh
pnpm exec vitest run server/shared-room-repository.test.ts server/multiplayer-links.e2e.test.ts server/request-auth.test.ts companion/test/routes.test.ts src/components/SharedConversationDetails.test.ts
cd ios && swift test --filter ShareClientTests
```

These cover whole-message pagination, durable history, removed-message exclusion,
invalid cursors, membership revocation, staged-upload exclusion and an authenticated
linked workspace. The linked fixture exercises bot discovery, preferences, attachment
publication, file listing, message edits/removal and denied access after room deletion.
Unknown forwarding routes remain denied. The sidecar permits only authenticated GET
for the file index. Private bot attachments are not part of this index.

Stop the foreground fixture with Ctrl-C. It removes only its disposable data.
