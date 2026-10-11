# PS5SX2 payload startup (AI-assisted)

This records a local console configuration verified after a full restart on
October 11, 2026.
The `tomchapin/etaHEN` checkout is a fork of the original etaHEN project.
The console instead runs GronedWaffel's experimental unified port, reporting
`ex-diag-20261005.unified-dev8` on firmware 12.02. This configuration does not
replace that running HEN with the binary in this repository.

## Why the app failed after a restart

WebKit Autoloader runs the exploit and starts the payload loader/manager.
It does not imply that etaHEN, ShadowMountPlus or the PS5SX2 helper have loaded.
Without ShadowMountPlus, PS5SX2 can fail to launch. With ShadowMountPlus but
without the separate helper, the tested app launched into a black screen and
logged `no etaHEN/OnionHEN/Helper answering`.

Loading the verified `PS5SXHelper.elf` from PS5SX2 release 2.00 restored the
2.02 app. Its new PID 121 received root privileges after 17 ms, passed the JIT
probe and drew the first interface frame after 660 ms. The user confirmed the
app opened again.

## Verified automatic startup

The running port's bootstrapper discovers `.elf` files with a sibling
`.auto_start` marker and launches them after its startup readiness check.
See its [bootstrapper](https://github.com/GronedWaffel/etahen-11.00-13.60/blob/main/Source%20Code/bootstrapper/source/main.cpp)
and [ELF validation/launch code](https://github.com/GronedWaffel/etahen-11.00-13.60/blob/main/Source%20Code/include/port_plugin_runtime.h).
The original project's bootstrapper also supports `.elf.auto_start` markers;
the exact directories and readiness handling differ between versions.

These files were placed on the console:

```text
/data/etaHEN/payloads/ShadowMountPlus_1.7-beta5fix1.elf
/data/etaHEN/payloads/ShadowMountPlus_1.7-beta5fix1.elf.auto_start
/data/etaHEN/payloads/PS5SXHelper.elf
/data/etaHEN/payloads/PS5SXHelper.elf.auto_start
```

The markers are empty files. Both ELF copies passed the port's header/segment
validation and SHA-256 FTP readback. No existing payload was replaced.

| Payload | SHA-256 |
| --- | --- |
| ShadowMountPlus 1.7-beta5fix1 | `e85fd63b705498488f39c09c9aea6f9fa22e0df2e932865de49a8cedf25172d5` |
| PS5SXHelper 2.00 / mountroot-v3 | `737d95a8a1545447c4166b094005704465e420f79f54e882e30e7f9e4cde6210` |

The HEN INI was not changed. Payload Manager was subsequently configured to
start the matching experimental HEN as described below. Configuration backup
and installation evidence are outside Git in the sibling
`PS5SX2-deploy/autostart/` folder. The two payloads are independent startup
entries; their discovery order is not a promised sequencing mechanism.
Automatic startup was verified on a fresh boot, as recorded below.

### Choose the matching HEN, then automate it

A restart test exposed a second problem: Payload Manager's default repository
offered `etaHEN_2.5B.bin`, and launch history showed that file was sent twice.
The HEN services did not start. A successful loader handoff is not a successful
HEN startup, and that original payload is not the experimental build used here.

The [unified r1 release](https://github.com/GronedWaffel/etahen-11.00-13.60/releases/tag/v2.5B-unified.1)
identifies `etaHEN-11.00-13.60.elf` as the unchanged, tested `unified-dev8` binary.
It was downloaded, checked against the published SHA-256
`46c161473b27d12f1fa5e2d1d10466ce587db841ed9e8d71a219424d8fcf637f`,
scanned by Defender with no threats found, and uploaded through Payload Manager
to `/data/pldmgr/payloads/etaHEN/etaHEN-11.00-13.60.elf`.
After the fresh boot, FTP readback verified all 36,746,616 bytes against the
same SHA-256.

The Payload Manager API saved these settings for its next start:

```text
AUTOLOAD_ENABLED=1
AUTOLOAD_DELAY=10
```

`/data/pldmgr/autoload.txt` contains just `etaHEN-11.00-13.60.elf`.
etaHEN then starts the two marker-enabled payloads itself. This avoids a separate
Payload Manager sequence racing the HEN's initialization. Other manager options
were preserved. The configuration API did not start another HEN while the
settings were saved; its autoload status remained inactive until the restart.

## After a full restart

1. Open **Media → WebKit Autoloader** on the PS5. The launcher is a media app.
2. Wait for the exploit and Payload Manager's 10-second autoload countdown.
3. Payload Manager sends `etaHEN-11.00-13.60.elf` automatically. The original
   2.5B binary in this checkout is not the tested firmware-12.02 payload.
4. Wait for etaHEN startup and the two automatic payloads to finish. Confirm
   the helper notification or its fresh startup log before opening PS5SX2.
5. Open PS5SX2. The startup chain handles etaHEN, ShadowMountPlus and the helper;
   do not launch additional copies manually when they are already running.

If the Media launcher fails, use the tested PC host/DNS/User's Guide route to
run the jailbreak and reach Payloads. Do not run the exploit a second time
just to open the payload manager after a successful jailbreak.

## Verification and rollback

The fresh-boot test passed without manually loading another payload:

- Payload Manager history contained only `etaHEN-11.00-13.60.elf`; autoload
  reported one payload completed, with the saved 10-second delay.
- The new HEN bootstrap PID 88 reported build `unified-dev8`, observed startup
  readiness and discovered two marker-enabled plugins.
- ShadowMountPlus PID 96 synchronized the library; helper PID 97 installed its
  mount hook and announced readiness about 82 seconds into console uptime.
- PS5SX2 PID 98 received root 17 ms after its request, passed the JIT probe,
  brought up its frontend in 617 ms and drew the first frame after 653 ms.
- The companion API reported `mode: menu`, and the user confirmed success.

The captured logs and API results are saved outside Git at
`PS5SX2-deploy/autostart/cold-boot-success/`. This verifies one full-restart
startup on the recorded firmware and versions.

Check `/data/etaHEN/experimental-diagnostics/trace.log` for plugin discovery
and startup readiness, `/data/PS5SXHelper.log` for a new `mountroot-v3` startup,
and `/data/shadowmount/debug.log` for fresh launch/mount handling.
PS5SX2's `/data/PCSX2/logs/boot.log` should show elevation, `JIT probe ok`, and
a frontend frame. Old log contents survive a restart, so check PID and the
new startup block rather than merely finding an old success message.

To disable HEN autoload, turn off Payload Manager's Autoload setting or restore
its backed-up config. To disable the HEN's two child payloads, remove only the
two `.elf.auto_start` markers
through FTP or turn off those entries in etaHEN's startup menu. Keep the ELF
copies for manual recovery. The separate helper remains available in Payload
Manager as **PS5SX2 Helper (separate)**. Do not enable the failing PS5SX2 AIO
alongside it.
