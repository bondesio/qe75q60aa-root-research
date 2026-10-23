# Samsung QE75Q60AAUXXC root research

A model-specific research package documenting the verified root path on one
Samsung QE75Q60AAUXXC retail television and carrying the source changes needed
to adapt the upstream Samsung TV root tools to that device.

This is a research record, not a universal installer. The tested device was
reported by Samsung's API as `21_NIKEL_UHD`, running Tizen 6.0 on ARMv7 with
software `T-NKLADEUC-2310, BT-S`. Other sizes, regional variants, and firmware
releases have not been verified by this work.

## Findings

- Initial command execution used SDB package-name command injection,
  identified publicly as SVE-2025-50109, and ran as `sdk` (UID 901).
- UID 0 came from the existing `archive-root` capability and signed-archive
  launcher route. This work did not discover a new kernel exploit.
- The temporary route was verified on 2026-10-20 with an authenticated UID 0
  session. The root agent reported UID/EUID/GID/EGID 0, the expected full
  capability mask, and Samsung's `User` SMACK label. Before/after hashes of
  `/etc/passwd` and `/usr/share/sdbd/public.pem` matched, and staging cleanup
  succeeded.
- A boot-launched agent was later installed in the existing Gaming Hub
  service-app slot. It reacquired root by connecting to local SDB and invoking
  the same chain. Three unattended boot validations were recorded on distinct
  boot IDs; this establishes repeatability for those tests only.
- The boot-agent deployment replaces the Gaming Hub service implementation.
  It is a user-space modification on writable app storage, not a firmware,
  kernel, or bootloader modification.

See [the technical report](docs/technical-report.md) for the chain, adaptations,
validation limits, provenance, and safety notes. [Patch notes](docs/patches.md)
explain the separate base revisions and build/deployment boundary.

## Source and attribution

The host root tool is based on
[`chris-ritsen/samsung-tv-root`](https://github.com/chris-ritsen/samsung-tv-root),
Unlicense, base revision `4ad2c4d3b0d0e0da4b5b7ebe4a787e94596f416b`.
The boot-agent layer is based on
[`levkropp/samsung-tv-root-untethered`](https://github.com/levkropp/samsung-tv-root-untethered),
a fork of that project, Unlicense, base revision
`2b0629381b3eaf5a78f84332d630d6d4b10681e7`.

The two files in `patches/` contain only the reviewed Q60A-specific source
changes relative to those bases. The relay's hard-coded private source-IP
restriction was intentionally omitted; configure a suitable access policy for
your own network before exposing any management listener. The patches do not
include generated payloads, certificates, private keys, relay credentials,
Samsung firmware or SDK framework binaries.

The SDB command-injection weakness is publicly documented as SVE-2025-50109
by [Bishop Fox](https://bishopfox.com/blog/samsung-tizen-os-version-through-9-0).
This overlay is released under the Unlicense; see [LICENSE](LICENSE). The
upstream projects retain their own licensing and attribution. This repository
is not affiliated with Samsung or the upstream authors.
