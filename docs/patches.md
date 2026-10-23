# Applying the source deltas

The two patches target different repositories and must be applied separately.
They are source-only overlays; they do not contain Samsung firmware, the
captured framework assemblies, generated signatures, or a ready-to-install TV
package.

## Temporary root tooling

Base: `chris-ritsen/samsung-tv-root`, revision
`4ad2c4d3b0d0e0da4b5b7ebe4a787e94596f416b`.

The patch adapts the host-driven archive-root route used for the initial
verified session. It includes the runtime-directory fallback, target launcher
environment, SMACK-aware archive/signature staging and the observed command
monitor budget. The source itself remains the upstream project; read its
license and instructions before building.

## Boot-time reacquisition

Base: `levkropp/samsung-tv-root-untethered`, revision
`2b0629381b3eaf5a78f84332d630d6d4b10681e7`.

The patch records the Q60A boot-agent changes: startup evidence, checked
staging, SDK UID 901 in the synthetic passwd view and target launcher
environment. It deliberately omits the deployment-specific private source-IP
allowlist from the relay. Use a fresh token and a network policy that restricts
the relay to your management host.

## Build and deployment boundary

The verified deployment was built against framework assemblies extracted from
the target TV; those Samsung assemblies are not included or redistributed here.
On the TV, the agent remained a hidden service app with on-boot enabled. The
launcher-owned staging directory and its exact inputs required SDK UID 901
ownership and the expected shared SMACK label; ordinary Unix modes alone were
not sufficient. A generic install script is not included because the verified
ownership/label setup was performed manually and the wrong app/manifest state
can disable the original Gaming Hub background service.

Before deploying, inspect the exact target firmware, take a restorable backup
of the original app and package metadata, restrict all management listeners,
and establish a recovery path. The published evidence covers the named test
set and recorded reboots only.
