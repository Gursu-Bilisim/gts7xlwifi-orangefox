# Changes outside the device tree

The releases of this repository are built from OrangeFox Recovery `fox_12.1` with two patches
applied outside `device/samsung/gts7xlwifi`. Base revisions are listed in `BASE_REVISION`.

## bootable_recovery.diff

Applies to OrangeFox `bootable/recovery`, commit `cbcc4f713dbef98d76d6d2cfd019b1ebfd9418e9`.
TWRP and OrangeFox are licensed under the GNU General Public License, version 3 or later, and so
are these changes (see `COPYING`).
The same source tree also builds
[Gursu-Bilisim/tucana-orangefox](https://github.com/Gursu-Bilisim/tucana-orangefox),
[Gursu-Bilisim/bramble-orangefox](https://github.com/Gursu-Bilisim/bramble-orangefox) and
[Gursu-Bilisim/jackpotlte-orangefox](https://github.com/Gursu-Bilisim/jackpotlte-orangefox), so the
patch contains the changes made for those devices as well.

Made for gts7xlwifi:

- `minuitwrp/graphics.cpp`: `gr_blit()` clamps the drawn area to the size of the source surface.
  With `TW_ROTATION := 270` the console overlay crashed the recovery (`SIGSEGV`,
  `SEGV_ACCERR`). The backtrace ended in `android::rect_generic`
  (`libpixelflinger/scanline.cpp`) called from `gr_blit()`, reading just past a 4-byte-per-pixel
  buffer: `gr_blit()` drew the requested width and height even when they exceeded the image, and
  pixelflinger does not clamp texture reads. With rotation the image is first copied into a buffer
  of exactly its size, and buffers above 64 KB end at a guard page, so the read faulted.
  Unrotated devices only lose pixels that were read from outside the image.
- `minuitwrp/graphics.cpp`, `minuitwrp/truetype.cpp`: the rotated copies of images and text are
  allocated with two spare, zeroed rows. Added while the crash was investigated; the clamp above
  is the fix, the padding stays as a margin. Not used without `TW_ROTATION`.

Made for the other devices and active here too:

- `partitionmanager.cpp`: when the ROM's fstab is copied to `/etc/additional.fstab`,
  `/dev/block/bootdevice/by-name/` paths are rewritten to `/dev/block/by-name/`, so that `/data`
  is resolved to its block device and FBE is detected.
- `partitionmanager.cpp`: before metadata decryption, `persist` is mounted read-only at
  `/mnt/vendor/persist` and `modem` read-only at `/vendor/firmware_mnt` (made for tucana). On
  gts7xlwifi `init.recovery.qcom.rc` has already mounted `persist` there, so the first command
  prints `mount -t ext4 -o ro /dev/block/by-name/persist /mnt/vendor/persist process ended with
  ERROR: 1` in red after the pattern is entered. The message has no effect; a check for an
  existing mount is planned for the next update of the shared patch.
- `gui/patternpassword.cpp`, `gui/objects.hpp`: `GUIPatternPassword::IsInRegion`. A dot accepts
  touches within three dot radii of its centre, which reaches past the pattern's placement box
  for the corner dots. A touch that started there was outside the box, so the page ignored the
  whole gesture. The pattern now also claims touches that hit a dot.

Made for the other devices, without effect here:

- `minuitwrp/resources.cpp`, `minuitwrp/include/minuitwrp/minui.h`, `gui/nanosvg.cpp`,
  `gui/resources.cpp`: on devices built with `TARGET_RECOVERY_PIXEL_FORMAT := ABGR_8888`
  (`res_swap_red_blue()` returns 1), SVG output, tints and shapes swap red and blue (made for
  jackpotlte). gts7xlwifi uses `RGBX_8888`, where the function returns 0; the `recovery` binary
  still needs the new `libminuitwrp.so`.
- `orangefox.mk`, `data.cpp`, `gui/pages.cpp`: build variable `OF_SCREEN_BOTTOM_INSET` (pixels,
  default 0) for screens that curve at the bottom edge (made for tucana). Not set here.
- Theme (`pages/templates/templates.xml`, `resources/vars.xml`): a 144x144 px touch area around
  the "show password" eye icon on the decrypt page (made for tucana).
- Theme (`gui/theme/portrait_hdpi/themes/navbar.xml`): gesture navigation is off by default and
  the screen height no longer adds 96 px for a hidden navigation bar (made for tucana).

## system_vold.diff

Applies to TeamWin `android_system_vold`, branch `android-12.1`, commit
`a164ba05c5fef288059774a776b2e6e1119957cf`, checked out at `system/vold`. vold is licensed under
the Apache License 2.0, and so are these changes.

- `Decrypt.cpp`: the gatekeeper is looked up as the `mdfpp` instance first, then as `default`.
  On this tablet LineageOS 24 verifies the lock screen credential in the QSEE MDFPP gatekeeper
  (`keymaster_tee nwd_gatekeeper verify ... ret = 0` in the Android log). In the recovery the
  AOSP `android.hardware.gatekeeper@1.0-service` loads `gatekeeper.mdfpp.so` and registers as
  `IGatekeeper/mdfpp`; the `default` instance (QTI service) takes the SPU path and returned
  `VERIFICATION_FAILED` (-30) for the correct pattern. Devices without an `mdfpp` instance keep
  the old behaviour (`tryGetService` does not wait).
- `FsCrypt.cpp`: no recursive `restorecon` over `system_ce`, `vendor_ce` and `misc_ce` after the
  CE key is installed. The recovery's Android 12 `libselinux` does not know the per-user MLS level
  that Android 14 and later give to `misc_ce/<user>/checkin` and `sdksandbox`, and relabelled them
  to `s0`. On the next boot `vold_prepare_subdirs` could not relabel `checkin` (`relabelfrom`
  denied): LineageOS 24 deletes and recreates the directory, LineageOS 23.2 leaves the CE storage
  unprepared. Android labels these paths itself when the user is unlocked.

## Applying

```bash
cd <OrangeFox source tree>/bootable/recovery
git checkout cbcc4f713dbef98d76d6d2cfd019b1ebfd9418e9
git apply <this repository>/patches/bootable_recovery.diff

cd <OrangeFox source tree>/system/vold
git checkout a164ba05c5fef288059774a776b2e6e1119957cf
git apply <this repository>/patches/system_vold.diff
```

`navbar.xml` uses CRLF line endings upstream; the diff keeps them, so apply it without
converting line endings.
