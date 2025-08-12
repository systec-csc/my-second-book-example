# Boot media and partitioning

| Size      | Device    | mountpoint    | usage                                         |
|-----------|-----------|---------------|-----------------------------------------------|
| 4 MiB     | mmcblk#p1 | /boot/vendor  | U-Boot bootloader binaries \[\^1\],           |
|           |           |               | Serial number, license keys, calibration data |
| 4 MiB     | mmcblk#p2 | /boot/u-boot  | U-Boot environment                            |
| \-        | mmcblk#p3 | \-            | Extended MBR partition                        |
| 2.5 GiB   | mmcblk#p5 | /             | root-fs slot `A` (read-only)                  |
| 2.5 GiB   | mmcblk#p6 | /             | root-fs slot `B` (read-only)                  |
| Remaining | mmcblk#p7 | /home \[\^2\] | user data in /home and overlays for           |
|           |           |               | with read-write access.                       |

Where `#` represents the boot media:

- `mmcblk0`: eMMC
- `mmcblk1`: SD card

## eMMC provisioning

This is only needed for new eMMC - this should already be done on purchased
hardware.

In u-boot run:

```sh
# This needs to be done only once for a new eMMC
# activate boot partitions for booting from them
mmc partconf 0 1 1 1
mmc bootbus 0 2 0 0

# https://e2e.ti.com/support/processors-group/processors/f/processors-forum/1168342/faq-am62x-how-to-check-and-configure-emmc-flash-rst_n-signal-to-support-warm_reset-from-emmc-booting-on-am62x-sk-e2
mmc rst-function 0 1

# Give device a power-on-reset. Some eMMC need this for unknown reasons.
```

See also: [Flash Linux to eMMC](https://dev.ti.com/tirex/explore/node?node=A__AdNWBqCVds4ZSqU9osT1tQ__AM62-ACADEMY__uiYMDcq__LATEST)
