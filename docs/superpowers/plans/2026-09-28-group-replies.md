# Group reply continuity and remote send plan

Approved by Firaz in chat: remember last explicitly mentioned bot per person per group; another bot switches; a person mention pauses until next bot mention. Settings: follow last bot, fixed bot, mentions only. Show selected responder. Existing permissions and per-send idempotency remain authoritative.

1. Add pure routing policy with tests for person pause, sender isolation, unavailable bots, explicit mentions, retries and modes. Derive remembered choice from durable accepted messages to avoid a second non-atomic write.
2. Extend actor-scoped room preferences, validating all values and fixed bot eligibility at API boundary. Preserve notification/read fields and persisted compatibility.
3. Apply routing at canonical send handler after mention validation, retain original routing on retries. Expose controls and responder identity in ordinary room UI.
4. Reproduce remote stale branch/optimistic row race with reducer tests. Preserve pending user rows during stale remote leaf updates. Publish polled busy state and wake polling after accepted send; do not broaden peer access.
5. Run focused regression, typechecks, isolated conversation fixture and production builds. Preserve signed packaging, independently review infrastructure changes if any. Reinstall only with fresh idle proof and new one-time receipt/rollback; never replay previous installer.
