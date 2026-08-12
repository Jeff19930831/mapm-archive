# clash-governance pointer

## Current Takeover

- Authority repo: `D:\Workspace\clash-governance`
- Active Windows profile: the clean `雕云` profile migrated on 2026-08-12.
- Governed behavior: DNS/merge policy, priority rules, custom proxy, and conditional domain-group script are bound to the active profile.
- The previous remote profile is retained only as forensic/rollback reference.
- Full current mapping, risks, verification, and rollback evidence: `D:\Workspace\clash-governance\handoff.md` and `progress.md`.

## Latest Result

- All five enhancements were migrated to the active profile.
- The empty Netflix auto-test-group failure was fixed by conditional group creation.
- Both generated configs passed Mihomo validation and were hot-reloaded.
- The existing dedicated SOCKS5 remained alive and usable; no endpoint or credential update was required.

## Next

- Run every future profile, node, DNS, or routing edit through `D:\Workspace\clash-governance\docs\runbooks\clash-change-control.md`.
- Keep this pointer short; append detailed evidence only in the authority repo.

## Blockers

- None for the Windows authority repo.

## Historical Note

Mac-side 2026-07-03 and 2026-07-23 Clash fixes remain in this pointer's `progress.md` until reconciled into a cross-device governance record.


