# QE75Q60AAUXXC root result and method

## Tested target

- Model: Samsung QE75Q60AAUXXC (Q60A, 75-inch)
- Platform identifier: `21_NIKEL_UHD`
- OS: Tizen 6.0, ARMv7
- Reported software string during validation: `T-NKLADEUC-2310, BT-S`
- Test dates: 2026-10-20 to 2026-10-21

This is evidence for that physical set and software state. It does not establish
compatibility with all Q60A devices, regional variants, newer firmware, or other
Tizen televisions.

## Root chain

The successful path combined two distinct steps:

1. **SDB command execution:** Developer Mode's SDB service accepted a crafted
   package name that reached a shell command context. The resulting process was UID 901, not root. This is the publicly disclosed command-injection
   issue [SVE-2025-50109](https://bishopfox.com/blog/samsung-tizen-os-version-through-9-0).
   The path required Developer Mode and the configured developer host.
2. **Privilege escalation:** the archive-root implementation used the TV's
   capability-bearing Samsung .NET runtime, a private mount namespace, and the
   signed-archive launcher (`sdbd-tarlauncher`) to obtain an authenticated
   UID-0 agent. This capability/namespace/archive-launcher route is described
   in the [upstream archive-root documentation](https://github.com/chris-ritsen/samsung-tv-root/blob/master/README.md#archive-root-route).
   No kernel or GPU exploit was used.

The Q60A-specific host-side adaptations in the patches include runtime
enumeration for Samsung's .NET host, target-compatible environment setup,
SMACK-aware staging of archive/signature inputs, and a passwd view that retains
both the synthetic root account and the pre-escalation service identity. The latter
was required because the launcher resolves the calling service identity before
executing the archive command.

## Temporary root validation

The session reported UID/EUID/GID/EGID 0 and capability mask
`0000003fffffffff`, with SMACK label `User`. The root tool compared hashes of
`/etc/passwd` and `/usr/share/sdbd/public.pem` before and after the session and
reported both unchanged. It also reported per-run staging cleanup. A separate
soak held an authenticated root session for more than five minutes; a normal
CLI run subsequently completed with exit status 0.

Root was initially volatile: it ended with the session and did not survive a
reboot.

## Boot-time reacquisition

A headless .NET service agent was installed in the existing
`com.samsung.tv.ghservice` service-app slot. The app was retained as a hidden
service app and configured to start at boot. Once running, it connected to
SDB on loopback, triggered the same SDB injection locally, and launched the
archive-root payload. A token-authenticated relay then allowed the management
host to use the resulting root agent while Developer Mode's configured host
remained loopback.

The boot agent replaced Gaming Hub's background service implementation. The
change was confined to writable application storage and package metadata; the
kernel, firmware, boot image and bootloader were not changed. Backups and a
targeted rollback path were prepared before installation.

Three unattended reboot validations were recorded on distinct boot IDs. The
agent completed its local SDB handshake, acquired new UID-0 proof after each
boot, and returned authenticated management access. This is repeatability over
those tested boots, not a guarantee across firmware updates or every power
state.

SSH was later added as a management option under the existing root runner. It
is not required to understand the initial exploit chain or the boot-time
reacquisition method.

## Attribution and novelty

Neither the SDB injection nor archive-root was discovered here. SVE-2025-50109
was publicly disclosed by Bishop Fox. The archive-root route and boot-time
self-root design were already described in the [upstream project](https://github.com/chris-ritsen/samsung-tv-root)
and the [Q60B fork report](https://github.com/chris-ritsen/samsung-tv-root/issues/1). This work's result is a verified adaptation to this exact QE75Q60A
model/software state, plus fixes needed for its .NET and SMACK behavior.

No public result for this exact model identifier was found during the
preparation of this report. That is not proof this was the first private or
public success; the result should be described as a newly documented exact-model
validation, not a claim of being first ever.

## Reproduction and safety limits

The patch files are source deltas, not a turnkey installer. They must be applied
to the exact upstream bases listed in the main README and built with the
matching TV runtime/framework. Tizen/Samsung binaries and SDK framework files
are not redistributed. Before any reproduction, review current firmware
compatibility, use an isolated trusted network, restrict management access,
and make a recovery plan. Replacing the Gaming Hub service affects that
feature. Firmware updates can replace the modified application or change the
chain. A UID-0 process with SMACK label `User` is not proof that all Samsung
security boundaries are bypassed.
