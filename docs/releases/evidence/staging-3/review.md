# Independent source review

Native reviewer `/root/review_reply_changes` inspected the actual free runtime, setup route, coordinator creation and recovery changes.

Findings corrected and independently rechecked:
- Revalidate the initiating admin session after upstream key verification and before persistence. A delayed-auth revocation fixture proves 401, unchanged credentials and released provider lock.
- Persist a new coordinator's profile and role together rather than identifying interrupted setup by name/soul. Save failure restores memory; retry and reload at capacity preserve one coordinator.

Reviewer reran 12 tests across the two failure-boundary suites and reported both findings cleared, with no new important issue in scope. No real provider account, inference or installed artifact was reviewed by that source check.

A separate integration check exercised the real isolated HTTP API for write-only credentials, delayed-body update serialization, unaffected sibling provider process, unavailable paid catalog and refused sends during reconnect.
