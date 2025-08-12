# RTI Watchdog

By default the first RTI watchdog is used as primary watchdog. The timeout is
configured to 30 seconds.

U-Boot initializes and configures the watchdog. It also passes the heartbeat (aka
timeout) setting to the Kernel via cmdline.

The Linux Kernel takes over the watchdog after `initramfs` has completed, since
the Watchdog driver (`rti_wdt`) is compiled to a Kernel Module.

`systemd` is configured to take over triggering of the watchdog when starting.

## Test watchdog

The following command will crash the kernel and after 15-30s the device will
reset.

```sh
echo c > /proc/sysrq-trigger
```

Or kill the init manager (`systemd`).

```sh
# kill `init` PID 1 (Most signals will not work here. SIGSEGV does work.)
kill -SEGV 1
```

## Configure Software Watchdog for systemd services

The following `systemd` service will cause a "hard" reboot since the service will
never service the configured software watchdog.

```sh
# /etc/systemd/system/fail.service
[Unit]
Description=Fail watchdog

[Service]
ExecStart=/bin/bash -c 'sleep 99999999'
Restart=always
StartLimitInterval=3min
StartLimitBurst=3
StartLimitAction=reboot-force
WatchdogSec=3

[Install]
WantedBy=multi-user.target
```

In case the reboot would get stuck the hardware watchdog should still lead to
reset.

See also:

- <https://www.man7.org/linux/man-pages/man5/systemd-system.conf.5.html#HARDWARE_WATCHDOG>
- <https://man7.org/linux/man-pages/man3/sd_notify.3.html>

## Other watchdog peripherals

sysWORXX AM62x devices may have more watchdogs available which can be used
directly from applications. To get a list of available watchdogs use the
following command.

```sh
ls -la /dev/watchdog*
```

To test the functionality and cause a reset run the following command.

```sh
echo 1 > /dev/watchdog3
```

See also: <https://www.kernel.org/doc/html/v5.9/watchdog/watchdog-api.html>
