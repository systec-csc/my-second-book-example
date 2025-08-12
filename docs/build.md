# Checking out Yocto project and build images

After doing the "normal" checkout using `git clone`, please run the following
command to checkout and initialize all git submodules (e.g. `sources/meta-*`)

```sh
git submodule update --init --recursive --force --checkout
```

## Build default image and write to SD card

Build the image:

```sh
cd build/
. conf/setenv
bitbake sysworxx-image-default
```

To get a list of available block devices run `lsblk`. Find the correct block
device for SD card and then redirect the output of `lsblk` to the device as
shown below.

```sh
# replace "sdX" with the block device which should be used
xzcat build/deploy-ti/images/sysworxx/sysworxx-image-default-sysworxx.rootfs.wic.xz > /dev/sdX
```

## Build and use the eMMC installer

This only applies for sysWORXX devices which have on-board eMMC.

The eMMC installer is a self-extracting archive which can be installed when
booted from SD cards. This will format and partition the eMMC and writes the
default image to it.

```sh
cd build/
. conf/setenv
bitbake sysworxx-image-default
./emmc-installer/build.sh
```

The commands above will create the file `sysworxx-image-default-emmc-installer-*.sh`.
To install it follow the steps below:

- Boot from SD Card
  - sysWORXX CTR-600/800 devices: DIP-6=Off
  - sysWORXX Pi: Boot jumper not connected
- Copy and install

```sh
# On PC: copy from PC zu sysworxx device (example)
scp sysworxx-image-default-emmc-installer-* root@sysworxx:/tmp
# On sysworxx:
chmod +x /tmp/sysworxx-image-default-emmc-installer-*
/tmp/sysworxx-image-default-emmc-installer-*
poweroff
```

- Unplug power supply, enable eMMC booting and power on again.
  - sysWORXX CTR-600/800 devices: DIP-6=On
  - sysWORXX Pi: Connect boot jumper (`X501`)

## Build and Install RAUC bundle

```sh
cd build/
. conf/setenv
bitbake sysworxx-bundle-default
```

The bundle `build/deploy-ti/images/sysworxx/sysworxx-bundle-default-sysworxx.raucb` can
then be copied to a running device and be installed with `rauc install`. For
example:

```sh
# on PC:
scp build/deploy-ti/images/sysworxx/sysworxx-bundle-default-sysworxx.raucb root@device:/tmp
# on sysWORXX device:
rauc install /tmp/sysworxx-bundle-default-sysworxx.raucb
reboot
```

The device will then reboot and switch to the other boot slot.

## Build SDK

```sh
cd build/
. conf/setenv
bitbake sysworxx-image-default -c populate_sdk
```

The SDK installer script can then be found in:

```
./build/deploy-ti/sdk/sysworxx-glibc-x86_64-sysworxx-image-default-aarch64-sysworxx-toolchain-5.0.4.sh
```

## Browser HMI image

For HMI usage for sysWORXX Pi another image is available:

- Yocto image: `bitbake sysworxx-image-browser-hmi`
- RAUC bundle: `bitbake sysworxx-bundle-browser-hmi`

Resulting files can be installed / updated analog to
`sysworxx-image/bundle-default`.

To enable the `browser-hmi.service` run:

```sh
systemctl enable --now browser-hmi
```

The service has some configuration options for different usage scenarios. Edit
the service with:

```sh
EDITOR=nano systemctl edit browser-hmi.service
# `EDITOR=nano` changes the default editor to `nano`. Users which prefer using
# `vim` can ignore this part.
# In case the terminal emulator over UART produces strange output for TUI
# applications one could try to prepend `TERM=xterm` to fix this.
```

## Browser-HMI example: Show External Web-Site on normal browser mode

Show an external Web-Site:

```sh
# systemctl edit browser-hmi.service
[Service]
Environment=HMI_URL="https://example.com"
Environment=HMI_ARG3=""
```

## Browser-HMI example: Show Local Node-RED Dashboard

Enable node-red with `systemctl enable --now node-red.service`. Then create a
flow and add a Dashboard to control/monitor some state.

See: <https://flows.nodered.org/node/node-red-dashboard>

```sh
# systemctl edit browser-hmi.service
[Unit]
Requires=NetworkManager-wait-online.service node-red.service
After=NetworkManager-wait-online.service node-red.service

[Service]
# node-red does not seem to implement SD-notify support, therefore give it some time to startup
ExecStartPre=sleep 10
Environment=HMI_URL="http://127.0.0.1:1880/ui"
```

## Keyboard input

By default keyboard inputs works with the (English) QWERTY layout. To setup a
different keyboard set the following environment variables.

- XKB_DEFAULT_RULES
- XKB_DEFAULT_MODEL
- XKB_DEFAULT_LAYOUT
- XKB_DEFAULT_VARIANT
- XKB_DEFAULT_OPTIONS

For example using a German keyboard with the `nodeadkeys` variant use:

```sh
# systemctl edit browser-hmi.service
[Service]
Environment=HMI_ARG3=""
Environment=XKB_DEFAULT_LAYOUT="de"
Environment=XKB_DEFAULT_VARIANT="nodeadkeys"
```

To list available options use the `localectl` command with the various `--list-*`
options.

See also [Documentation: Cage Configuration](https://github.com/cage-kiosk/cage/wiki/Configuration)

## Modifying Linux kernel source

```sh
devtool modify linux-ti-staging
devtool finish linux-ti-staging ../sources/meta-of-your-choice
```

- `modify` will checkout the kernel sources to the workspace directory. Patches
  of recipes will be applied to the source as git commits.
  - changes to the kernel configuration via fragment files is not supported
- `finish` will format all git commits to patches and copy them to the specified
  output directory.
- After this it may be necessary to check the recipe file's content, since the
  formatting is sometimes a bit awkward.

## Modifying Linux kernel configuration

```sh
bitbake linux-ti-staging -c menuconfig
bitbake linux-ti-staging -c diffconfig
# and `mv` the generated config fragment to the target layer
```

Hint: This can also be done while the kernel is in _modifying_ state. (see
section above)

## Build a specific device-tree from Kernel source

Enter development shell for Linux kernel and build a specific device tree:

```sh
bitbake linux-ti-staging -c devshell
make defconfig
make ti/k3-am623-systec-ctr800-rev0.dtb
```

## Force a specific device tree

Stop in u-boot shell and perform one of the following commands to boot with the
specified device tree.

```sh
setenv findfdt setenv fdtfile ti/k3-am623-systec-ctr-prodtest.dtb; boot
setenv findfdt setenv fdtfile ti/k3-am623-systec-ctr600-rev0.dtb; boot
setenv findfdt setenv fdtfile ti/k3-am623-systec-ctr800-rev0.dtb; boot
setenv findfdt setenv fdtfile ti/k3-am625-systec-pi-rev0.dtb; boot
```
