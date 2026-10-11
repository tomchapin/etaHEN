# PS5SX2 payload startup (AI-assisted)

This records a local console configuration prepared on October 11, 2026.
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

## Prepared automatic startup

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

The HEN INI and Payload Manager autoload settings were not changed. Configuration
backup and installation evidence are outside Git in the sibling
`PS5SX2-deploy/autostart/` folder. The two payloads are independent startup
entries; their discovery order is not a promised sequencing mechanism.
Automatic startup after a fresh boot remains to be verified.

## After a full restart

1. Open **Media → WebKit Autoloader** on the PS5. The launcher is a media app.
2. Wait for the exploit to finish, then open Payloads / Payload Manager.
3. Load the same experimental etaHEN payload once. The original 2.5B binary in
   this checkout is not the tested firmware-12.02 payload.
4. Wait for etaHEN startup and the two automatic payloads to finish. Confirm
   the helper notification or its fresh startup log before opening PS5SX2.
5. Open PS5SX2. During the verification boot, do not launch separate copies of
   ShadowMountPlus or the helper manually: that would hide an autostart failure.

If the Media launcher fails, use the tested PC host/DNS/User's Guide route to
run the jailbreak and reach Payloads. Do not run the exploit a second time
just to open the payload manager after a successful jailbreak.

## Verification and rollback

Check `/data/etaHEN/experimental-diagnostics/trace.log` for plugin discovery
and startup readiness, `/data/PS5SXHelper.log` for a new `mountroot-v3` startup,
and `/data/shadowmount/debug.log` for fresh launch/mount handling.
PS5SX2's `/data/PCSX2/logs/boot.log` should show elevation, `JIT probe ok`, and
a frontend frame. Old log contents survive a restart, so check PID and the
new startup block rather than merely finding an old success message.

To disable this configuration, remove only the two `.elf.auto_start` markers
through FTP or turn off those entries in etaHEN's startup menu. Keep the ELF
copies for manual recovery. The separate helper remains available in Payload
Manager as **PS5SX2 Helper (separate)**. Do not enable the failing PS5SX2 AIO
alongside it.
