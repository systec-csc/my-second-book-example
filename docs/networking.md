# Networking

Network Manager is used to configure network interfaces.

Use `nmcli` to show the status of all network connections.

## Ethernet

By default Ethernet interfaces are configured for DHCP.

Example for setting up static IPv4 network connection:

```sh
nmcli con add type ethernet con-name "static-ip" ifname eth1
nmcli con mod static-ip ipv4.addresses 192.168.3.100/24 gw4 192.168.3.1
nmcli con mod static-ip ipv4.method manual
nmcli con mod static-ip connection.autoconnect yes
nmcli con up static-ip ifname eth1
```

## WiFi and Bluetooth

## WiFi

```sh
nmcli device wifi list
nmcli device wifi connect SSID_or_BSSID password password
# WiFi should now be connected
nmcli device show wlan0
nmcli connection show
```

## Set wireless regulatory domain

The regulatory domain is set via a kernel module parameter for `brcmfmac`.

```txt
#/etc/modprobe.d/brcmfmac_regd.conf
options brcmfmac regdomain="ETSI"
```

## Bluetooth

```sh
bluetoothctl
power on
discoverable on
# connect with tablet or smartphone
```

## Links for WiFi/Bluetooth driver/firmware

- <https://lairdcp.github.io/guides/linux_docs/1.0/lwb-sona-ifx/sig_lwb_sona_ifx_series_radio_linux_yocto.html>
- <https://github.com/LairdCP/LWB5plus-Tutorials/blob/main/A2DP-test-LWB5p-dongle-iMX8M-Plus-EVK.md>
- <https://github.com/LairdCP/LWB5plus-Tutorials/blob/main/LWBplusDongle-imx8-yocto.md>
