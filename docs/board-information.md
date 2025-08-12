# Board Information EEPROM

sysWORXX devices have an on-board EEPROM with vendor information stored as an
partial u-boot environment.

These data are used during bootup to set proper MAC addresses and select the
correct device tree for the device.

!!! warning
    This section is only for reference of programmed data these data should and
    cannot be changed by the customer.

## Example EEPROM data

```sh
setenv eeprom_layout 1
setenv fdt_prefix k3-am623-systec-ctr800
setenv hw_iface_rev 0
env set -f ethaddr  22:22:22:22:22:30
env set -f eth1addr 22:22:22:22:22:32
setenv som_order_no 111111
setenv som_bom_rev  1
setenv som_pcb_num  4123
setenv som_serial_number 123123
setenv dev_order_no 222222
setenv dev_bom_rev 1
setenv dev_pcb_num 4234
setenv dev_serial_number 234234
```

Not all of the above data are mandatory for all sysWORXX devices.

The device tree name will be derived in the following scheme `${fdt_prefix}-rev${hw_iface_rev}.dtb`.
If either `fdt_prefix` OR `hw_iface_rev` are not set the `k3-am623-systec-fallback.dptb` will be used.
