# clash-governance pointer

## Current Takeover

- Authority repo: `D:\Workspace\clash-governance`
- Active Windows profile: the clean `雕云` profile migrated on 2026-08-12.
- Governed behavior: DNS/merge policy, priority rules, custom proxy, conditional domain-group script, and exact WeChat Official Account DIRECT rules are bound to the active profile.
- The previous remote profile is retained only as forensic/rollback reference.
- Full current mapping, risks, verification, and rollback evidence: `D:\Workspace\clash-governance\handoff.md` and `progress.md`.

## Latest Result

- Added exact DIRECT rules for the WeChat API, operator backend, and developer portal.
- Both generated configs passed Mihomo validation; runtime inspection returned three target Domain -> DIRECT rules.
- Direct egress was verified and WeWrite created the requested draft successfully.
- Persistent WeWrite AppID/AppSecret fields were cleared after publishing; the disclosed AppSecret still needs rotation before reuse.

## Next

- Rotate the disclosed WeChat AppSecret before the next publish and recheck the DIRECT public IP against the official-account allowlist.
- Use D:\Workspace\clash-governance\docs\runbooks\wechat-official-account-direct.md for publishing preflight and clash-change-control.md for live routing edits.
- Keep this pointer short; append detailed evidence only in the authority repo.

## Blockers

- User/backend action remains: rotate the disclosed WeChat AppSecret.
- The direct public IP can change after an ISP reconnect and invalidate the allowlist.

## Historical Note

Mac-side 2026-07-03 and 2026-07-23 Clash fixes remain in this pointer's `progress.md` until reconciled into a cross-device governance record.


