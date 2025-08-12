
# Using Peripherals

This section describes, how to access the GPIO in the Linux user space,
_without_ using the `sysworxx-io` library.

Available inputs and outputs may vary from device to device. To get some insight
into Pins and their use the following command.

```sh
gpioinfo | less
```

## Digital Outputs

Some devices provide digital outputs via PWM. When setting the duty cycle of PWM
output to 100% this is equivalent to setting the output active.

## Digital Outputs - sysWORXX CTR-600

```sh
gpioset -t0 DO_0=1; sleep 0.2; gpioset -t0 DO_0=0
gpioset -t0 DO_1=1; sleep 0.2; gpioset -t0 DO_1=0
gpioset -t0 LED_RUN=1; sleep 0.2; gpioset -t0 LED_RUN=0
gpioset -t0 LED_ERR=1; sleep 0.2; gpioset -t0 LED_ERR=0

# DO_2_PWM
echo 0 > /sys/class/pwm/pwmchip0/export
echo 400000000 > /sys/class/pwm/pwmchip0/pwm0/period
echo 300000000 > /sys/class/pwm/pwmchip0/pwm0/duty_cycle
echo 1 > /sys/class/pwm/pwmchip0/pwm0/enable

# DO_3_PWM
echo 0 > /sys/class/pwm/pwmchip2/export
echo 400000000 > /sys/class/pwm/pwmchip2/pwm0/period
echo 100000000 > /sys/class/pwm/pwmchip2/pwm0/duty_cycle
echo 1 > /sys/class/pwm/pwmchip2/pwm0/enable
```

## Digital Outputs - sysWORXX CTR-800

```sh
gpioset -t0 DO_0=1; sleep 0.2; gpioset -t0 DO_0=0
gpioset -t0 DO_1=1; sleep 0.2; gpioset -t0 DO_1=0
gpioset -t0 DO_2=1; sleep 0.2; gpioset -t0 DO_2=0
gpioset -t0 DO_3=1; sleep 0.2; gpioset -t0 DO_3=0
gpioset -t0 DO_4=1; sleep 0.2; gpioset -t0 DO_4=0
gpioset -t0 DO_5=1; sleep 0.2; gpioset -t0 DO_5=0
gpioset -t0 DO_6=1; sleep 0.2; gpioset -t0 DO_6=0
gpioset -t0 DO_7=1; sleep 0.2; gpioset -t0 DO_7=0
gpioset -t0 DO_8=1; sleep 0.2; gpioset -t0 DO_8=0
gpioset -t0 DO_9=1; sleep 0.2; gpioset -t0 DO_9=0
gpioset -t0 DO_10=1; sleep 0.2; gpioset -t0 DO_10=0
gpioset -t0 DO_11=1; sleep 0.2; gpioset -t0 DO_11=0
gpioset -t0 DO_12=1; sleep 0.2; gpioset -t0 DO_12=0
gpioset -t0 DO_13=1; sleep 0.2; gpioset -t0 DO_13=0

# DO_14_PWM
echo 0 > /sys/class/pwm/pwmchip0/export
echo 400000000 > /sys/class/pwm/pwmchip0/pwm0/period
echo 300000000 > /sys/class/pwm/pwmchip0/pwm0/duty_cycle
echo 1 > /sys/class/pwm/pwmchip0/pwm0/enable

# DO_15_PWM
echo 0 > /sys/class/pwm/pwmchip2/export
echo 400000000 > /sys/class/pwm/pwmchip2/pwm0/period
echo 100000000 > /sys/class/pwm/pwmchip2/pwm0/duty_cycle
echo 1 > /sys/class/pwm/pwmchip2/pwm0/enable

gpioset -t0 RELAY0_EN=1; sleep 1; gpioset -t0 RELAY0_EN=0
gpioset -t0 RELAY1_EN=1; sleep 1; gpioset -t0 RELAY1_EN=0
gpioset -t0 LED_RUN=1; sleep 0.2; gpioset -t0 LED_RUN=0
gpioset -t0 LED_ERR=1; sleep 0.2; gpioset -t0 LED_ERR=0
```

## sysWORXX Pi Outputs

sysWORXX Pi by default only has RGB LED outputs. Via device tree overlays (DTBO)
more can be configured. See [](#device-tree-overlay-dtbo-setup).

```sh
gpioset -t0 LED_RD=1 LED_GN=0 LED_BL=0 # red
gpioset -t0 LED_RD=0 LED_GN=1 LED_BL=0 # green
gpioset -t0 LED_RD=0 LED_GN=0 LED_BL=1 # blue
gpioset -t0 LED_RD=0 LED_GN=0 LED_BL=0 # off
```

## Digital Inputs

Most digital inputs are mapped to a `gpio-keys` device which means they are
available as regular input device. State changes of inputs can be observed with
`evtest`.

However this does not make sense for all kinds of inputs. Some are still
available via GPIO character interface.

```sh
gpioinfo | less
```

## Digital Inputs - sysWORXX CTR-600/800

```sh
# DI0 .. DI15 == KEY_F1 .. KEY_F16
# RUN switch == KEY_1
evtest /dev/input/by-path/platform-gpio_input-event

# /BOOT and /CONFIG
gpioget "/BOOT" "/CONFIG"
```

## Digital Inputs - sysWORXX Pi

sysWORXX Pi by default only has no inputs. Via device tree overlays (DTB)
more can be configured. See `dtbo-setup`.

## A/B Encoder / Counter

On CTR-600 and CTR-800 one A/B Encoder / Counter is available. This functionality
is available in parallel to _Digital Input_ functionality.

Connection:

- CTR-600: `DI_2 / DI_3`
- CTR-800: `DI_14 / DI_15`

```sh
cd /sys/bus/counter/devices/counter0/count0

# A/B Encoder
echo 0 > enable
echo "quadrature x4" > function
echo 1 > enable
while true; do cat count; sleep 0.2; done

# Counter (counts pulses on first DI channel, second DI channel switches direction)
echo 0 > enable
echo "pulse-direction" > function
echo 1 > enable
while true; do cat count; sleep 0.2; done
```

## Serial interfaces

- `ttyS4` is `SERIAL0` on CTR-800
- `ttyS5` is `SERIAL1` on CTR-800
- `ttyS6` is `SERIAL2` on CTR-800

## RS-232

Setup:

```sh
# on PC
stty -F /dev/ttyUSB0 115200 -echo raw
date > /dev/ttyUSB0
cat /dev/ttyUSB0
# on sysWORXX device
stty -F /dev/ttyS4 115200 -echo raw
cat /dev/ttyS4
date > /dev/ttyS4
```

## RS-232 with hardware flow control

```sh
# on PC
stty -F /dev/ttyUSB0 115200 -echo raw crtscts
date > /dev/ttyUSB0
# on sysWORXX device
stty -F /dev/ttyS6 115200 -echo raw crtscts
cat > /dev/ttyS6
# expected behavior:
# - sending data will block until other side has `cat` running
# - if `cat` is already running the send command will not block
```

## RS-485 with DEN GPIO mapped to RTS

This requires Driver Enable (DEN) mapped to RTS,
e.g. `rts-gpios = <&main_gpio0 63 GPIO_ACTIVE_HIGH>;`

```sh
# on sysWORXX device
/usr/bin/rs485 /dev/ttyS6
stty -F /dev/ttyS6 115200 -echo raw
cat /dev/ttyS6
date > /dev/ttyS6
# on PC
stty -F /dev/ttyUSB0 115200 -echo raw
date > /dev/ttyUSB0
cat > /dev/ttyUSB0
```

## CAN Bus

Example setup of CAN Bus interface:

```sh
ip link set can0 type can bitrate 500000
ip link set can0 up
```

Use `cansend` / `candump` to send and receive CAN messages.

## sysWORXX Pi 40 Pin Header

All pins of the 40 Pin Header are unused by default. To allow adding
functionality to these pins without replacing the whole root file system
device tree overlays can be used.

|               Function | Pin | Pin | Default                     |
|-----------------------:|:---:|:---:|:----------------------------|
|                    3V3 |  1  |  2  | 5V0                         |
|    GPIO1_23 (I2C3 SCA) |  3  |  4  | 5V0                         |
|    GPIO1_22 (I2C3 SCL) |  5  |  6  | GND                         |
|                GPIO0_0 |  7  |  8  | GPIO0_72 (UART4_TX)         |
|                    GND |  9  | 10  | GPIO0_71 (UART4_RX)         |
|                GPIO0_1 | 11  | 12  | GPIO0_35 (MCASP1_ACLKX)     |
|                GPIO0_2 | 13  | 14  | GND                         |
|                GPIO0_3 | 15  | 16  | MCU_GPIO0_13 (MCU_MCAN0_TX) |
|                    3V3 | 17  | 18  | MCU_GPIO0_14 (MCU_MCAN0_RX) |
|      GPIO0_9 (SPI1_D0) | 19  | 20  | GND                         |
|     GPIO0_10 (SPI1_D1) | 21  | 22  | GPIO0_40                    |
|     GPIO0_8 (SPI1_CLK) | 23  | 24  | GPIO0_7 (SPI1_CS0)          |
|                    GND | 25  | 26  | GPIO0_13 (SPI1_CS1)         |
|    I2C2_SDA (GPIO0_44) | 27  | 28  | I2C2_SCL (GPIO0_43)         |
|    GPIO1_24 (MCAN0_TX) | 29  | 30  | GND                         |
|    GPIO1_25 (MCAN0_RX) | 31  | 32  | GPIO1_9 (PWM1)              |
|        GPIO1_28 (PWM2) | 33  | 34  | GND                         |
| GPIO0_37 (MCASP1_AFSX) | 35  | 36  | GPIO0_41                    |
|                GPIO0_4 | 37  | 38  | GPIO0_34 (MCASP1_AXR0)      |
|                    GND | 39  | 40  | GPIO0_33 (MCASP1_AXR1)      |

Functions in parentheses are usable as alternative.

## Device Tree Overlay (DTBO) setup

The script `dtbo-setup` is provided to simplify DTBO configuration. Examples:

```sh
dtbo-setup --help
dtbo-setup ls
dtbo-setup set k3-am625-systec-pi-sysworxx-io-default.dtbo
dtbo-setup get
```

Applying device tree overlays is performed by the _u-boot_ bootloader. The DTBO
configuration is saved in _u-boot environment_. Therefore a `reboot` is needed to
apply the new device tree overlay setting.

The environment lives in the second partition (`mmcblk#p2`) on the respective boot device.
This partition is mounted and the environment is available under:
`/boot/u-boot/uboot.env`

### DTBO: GPIO via 40 Pin Header

Enable with: `dtbo-setup set k3-am625-systec-pi-sysworxx-io-default.dtbo`

This will enable:

- SPI1 with CS0
- UART4
- I2C3
- GPIOs as described below

Outputs:

```sh
gpioset -t0 DO_0=1; sleep 0.2; gpioset -t0 DO_0=0  # GPIO0_1
gpioset -t0 DO_1=1; sleep 0.2; gpioset -t0 DO_1=0  # GPIO0_35
gpioset -t0 DO_2=1; sleep 0.2; gpioset -t0 DO_2=0  # GPIO0_3
gpioset -t0 DO_3=1; sleep 0.2; gpioset -t0 DO_3=0  # MCU_GPIO0_13
gpioset -t0 DO_4=1; sleep 0.2; gpioset -t0 DO_4=0  # GPIO0_40
gpioset -t0 DO_5=1; sleep 0.2; gpioset -t0 DO_5=0  # GPIO0_41
gpioset -t0 DO_6=1; sleep 0.2; gpioset -t0 DO_6=0  # GPIO0_4
```

Inputs:

```sh
# DI_0 == GPIO0_0  == BTN_TRIGGER_HAPPY1
# DI_1 == GPIO0_2  == BTN_TRIGGER_HAPPY2
# DI_2 == MCU_GPIO0_14  == BTN_TRIGGER_HAPPY3
# DI_3 == GPIO0_13 == BTN_TRIGGER_HAPPY4
# DI_4 == GPIO0_44 == BTN_TRIGGER_HAPPY5
# DI_5 == GPIO0_43 == BTN_TRIGGER_HAPPY6
# DI_6 == GPIO1_24 == BTN_TRIGGER_HAPPY7
# DI_7 == GPIO1_25 == BTN_TRIGGER_HAPPY8
# DI_8 == GPIO1_9  == BTN_TRIGGER_HAPPY9
# DI_9 == GPIO1_28 == BTN_TRIGGER_HAPPY10
# DI_10 == GPIO0_37 == BTN_TRIGGER_HAPPY11
# DI_11 == GPIO0_34 == BTN_TRIGGER_HAPPY12
# DI_12 == GPIO0_33 == BTN_TRIGGER_HAPPY13
evtest /dev/input/by-path/platform-sysworxx-0-io-inputs-event-joystick
```

### DTBO: sysWORXX Pi HAT – Smart Metering

- RS-485/Modbus RTU: `/dev/ttyS4`
- M-Bus: `/dev/ttyS6`
- S0 input: `/dev/input/by-path/platform-sysworxx-1-smart-metering-hat-inputs-event`

### DTBO: sysWORXX Pi HAT – Industrial Communication

- NetX SPI interface: `/dev/spidev1.0`
- NetX GPIO character device: `industrial-communication-hat-gp`
- CAN-Bus SocketCAN interface: `can0`
- CAN LED: `/sys/class/leds/mcan0_act`

