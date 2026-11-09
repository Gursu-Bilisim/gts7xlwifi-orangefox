## Pre-release test notes

Tested by the maintainer before publishing. Update this file for every release.

| Item | Value |
|---|---|
| Device | Samsung Galaxy Tab S7+ Wi-Fi, SM-T970 (gts7xlwifi) |
| ROM | LineageOS 24.0, `24.0-20261029-nightly` (Android 17), microG, FolkPatch root in boot |
| Security patch (ROM boot header) | 2026-10 |
| ROM kernel | 4.19.325-cip136-st20-perf-gad129ac77e9d, clang 22.0.2 (the recovery carries the same kernel) |
| /data | f2fs, FBE v2 (`aes-256-xts:aes-256-cts:v2+inlinecrypt_optimized`), metadata encryption |
| Lock screen | pattern |
| Recovery reports | Android 99.87.36, patch 2127-12-31 (placeholders, see README) |
| Tested image | SHA-256 `01bcb8018be7eccf9a44f2552e94416139af002dafec36a66137a3d63445c67d` |

Installed with `samloader flash --partition RECOVERY <img> --no-reboot` in Download mode; the
recovery was then started with `adb reboot recovery` from Android.

| Test | Result | Notes |
|---|---|---|
| Decrypt /data with the pattern | OK | first attempt; `User 0 Decrypted Successfully` |
| Android after a decrypting session | OK | `sys.user.0.ce_available=true`; `/data/misc_ce/0/checkin` keeps `checkin_data_file:s0:c512,c768` and its inode; no `Relabelling` or `deleting for recreation` in logcat |
| Console overlay (^ button) | OK | crashed the recovery before this build, see patches/README.md |
| Brightness slider (console and Settings) | OK | |
| Files menu | OK | |
| ADB | OK | |
| MTP | OK | Windows Explorer, files copied |
| Reboot to Download mode | OK | from the OrangeFox reboot options |
| microSD (exFAT, 1 TB) | OK | backups written to it |
| Backup of Boot, EFS and Persist to microSD | OK | 120 MB in 7 s, `efs_image.emmc.win` 20 MB, `persist_image.emmc.win` 32 MB, with `.sha2` files |
| USB-OTG card reader behind a USB-C hub (exFAT) | OK | mounted at `/usb_otg`, shown as USB storage in the Files menu |
| `recovery-inspect.ps1` | OK | all checks |
| `patch-check.ps1` against the ROM boot.img | OK | `RESULT: match` (placeholder values, measured on this ROM) |

This build differs from the previous test build (`a955e88e…f594798`) only in `libminuitwrp.so`
(the console fix). Backup, USB-OTG, microSD and the Android checks above were run with that build;
the device files, vold and the recovery binary did not change.

Known message: after the pattern is entered, the console shows `mount -t ext4 -o ro
/dev/block/by-name/persist /mnt/vendor/persist process ended with ERROR: 1` in red. persist is
already mounted there; the message has no effect (see patches/README.md).

Not tested: restoring a backup, Format Data, a Data backup, installing a ROM or an update from
OrangeFox, flashing the `.tar` with Odin, installing the OrangeFox `.zip`, FAT32 and NTFS storage,
PIN or password lock screens, other LineageOS builds, SM-T975 and SM-T976B.
