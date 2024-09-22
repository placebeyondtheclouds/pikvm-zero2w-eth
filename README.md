# DIY IP KVM, pikvm on RPI zero 2W with ethernet

## description

Based on https://github.com/pikvm/pikvm.

Use case:

- to control a server with old BMC firmware that has broken remote control feature
- Wake-on-LAN other devices

Main differences between my build and the [official build for Raspberry Pi Zero 2 W](https://docs.pikvm.org/v2/#required-parts):

    - My hardware setup has ENC28J60 chip based ethernet adapter, connected to GPIO through SPI protocol (four-wire serial bus)
    - DS1307 chip RTC clock module (optional)
    - Didn't utilize ATX power control
    - Powered by the keyboard/mouse interface cable plugged into the USB socket of the rpi (can be changed to external power by cutting +5V wire in this cable and adding another cable plugged into PWR socket of the rpi)

RPI GPIO pinout: https://pinout.xyz/pinout/spi

## parts list

- a generic plastic case for PLC, 145x90x40
- HDMI to CSI2 adapter, TC35874XBG chip https://www.waveshare.com/wiki/HDMI_to_CSI_Adapter
- ribbon cable (wide to narrow)
- ENC28J60 chip based ethernet adapter (10Mbps) https://www.waveshare.com/wiki/ENC28J60_Ethernet_Board
- rp0 aluminium case
- USB mini to USB A cable
- 32Gb tf card
- SSD1315 chip based OLED display https://www.waveshare.com/wiki/0.96inch_OLED_Module
- Raspberry Pi zero 2 W - https://www.waveshare.com/wiki/Raspberry_Pi_Zero_2_W

## software

https://files.pikvm.org/images/v2-hdmi-zero2w-latest.img.xz

## wiring

- ENC28J60 board is connected to SPI0 bus (SPI0 pins are GPIO 7, 8, 9, 10, 11)

| ENC28J60 | RPI GPIO         |
| -------- | ---------------- |
| GND      | GND (Pin 20)     |
| VCC      | 3.3V (Pin 17)    |
| CS       | CE0 (Pin 24)     |
| SCK      | SCLK (Pin 23)    |
| SI       | MOSI (Pin 19)    |
| SO       | MISO (Pin 21)    |
| INT      | GPIO 25 (Pin 22) |

- DS1307 chip RTC module is connected to I2C bus, my particular module is occupying GPIO 1-10 pins with I2C bus pins passthrough

- OLED module is connected to SPI1 (SPI1 pins are GPIO 16, 17, 18, 19, 20, 21)

| OLED | RPI GPIO                   |
| ---- | -------------------------- |
| VCC  | 3.3V (pin 1)               |
| DIN  | SPI1 MOSI GPIO 20 (pin 38) |
| CS   | SPI1 CE0 GPIO 18 (pin 12)  |
| RES  | RES GPIO 27 (pin 13)       |
| GND  | GND (pin 6)                |
| CLK  | SPI1 SCLK GPIO 21 (pin 40) |
| DS   | GPIO 25 (pin 22)           |

Wire colour GPIO Pin # Name
White 16 RST reset
Orange 17 CS – Chip select
Yellow 18 CLK - Clock
Blue 19 MOSI / DIN
Green 20 DC – Data/Command
Red 3.3 Volts VCC
Black GND GND

oled (GND/G) --- Pi ( Pin 6 Gnd)
oled (Vin/+) --- Pi (Pin 1 3.3v)
oled (MOSI/SI) --- Pi (Pin 19) GPIO 10 (MOSI)
oled (SCK/CL) --- Pi (Pin 23) GPIO 11 (SCLK))
oled (DC/DC) --- Pi (Pin 16) GPIO 23
oled (Reset/R) --- Pi (Pin 18) GPIO 24
oled (OLEDCS/OC) --- Pin 24 (GPIO 8 CE0)

By default the display is set to SPI mode. To set the display to I2C mode, remove resistor R1 and set the address with RO (default is 0x3D). OLED module I2C bus wiring:

| OLED | RPI GPIO     | DS1307 |
| ---- | ------------ | ------ |
| VCC  | 3.3V (pin 1) | 3V3    |
| DIN  | SDA (pin 3)  | SDA    |
| CLK  | SCL (pin 5)  | SCL    |
| GND  | GND (pin 6)  | GND    |

## setting up

enable SPI1 in /boot/config.txt
`dtoverlay=spi1-3cs`
