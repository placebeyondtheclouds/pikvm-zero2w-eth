# DIY IP KVM, pikvm on RPI zero 2W with ethernet

## description

Based on https://github.com/pikvm/pikvm

Use case:

- to control a server with old BMC firmware that has broken remote control feature
- Wake-on-LAN other devices

Main differences between my build and the [official build for Raspberry Pi Zero 2 W](https://docs.pikvm.org/v2/#required-parts):

    - My hardware setup has ENC28J60 chip based ethernet adapter, connected to GPIO through SPI protocol (four-wire serial bus)
    - DS1307 chip RTC clock module (optional)
    - Didn't utilize ATX power control
    - Powered by the keyboard/mouse interface cable plugged into the USB socket of the rpi (can be changed to external power by cutting +5V wire in this cable and adding another cable plugged into PWR socket of the rpi)

SPI reference: https://www.analog.com/en/resources/analog-dialogue/articles/introduction-to-spi-interface.html
Overlays reference: https://raw.githubusercontent.com/raspberrypi/firmware/master/boot/overlays/README

## parts list

- a generic plastic case for PLC, 145x90x40 2 元
- HDMI to CSI2 adapter, TC35874XBG chip https://www.waveshare.com/wiki/HDMI_to_CSI_Adapter 129 元
- ribbon cable, 22pin to 15 pin, 20cm 8 元
- ENC28J60 chip based ethernet adapter (10Mbps) https://www.waveshare.com/wiki/ENC28J60_Ethernet_Board 14 元
- rp0 aluminium case 24 元
- USB mini to USB A cable
- 32Gb A1 U1 C10 microsd card 60 元
- SSD1315 chip based OLED display https://www.waveshare.com/wiki/0.96inch_OLED_Module 28 元
- Raspberry Pi zero 2 W - https://www.waveshare.com/wiki/Raspberry_Pi_Zero_2_W 165 元
- DS1307 RTC board 20 元 (do not recommend, better use DS3231 chip based board)
- terminated colored wires 2 元

## software

https://files.pikvm.org/images/v2-hdmi-zero2w-latest.img.xz

## wiring

- RPI GPIO pinout: https://pinout.xyz/pinout/spi
- ENC28J60 board is connected to SPI0 bus (SPI0 pins are GPIO 7, 8, 9, 10, 11)

| ENC28J60 | RPI GPIO             |
| -------- | -------------------- |
| GND      | GND (Pin 20)         |
| VCC      | 3.3V (Pin 17)        |
| CS       | GPIO8 CE0 (Pin 24)   |
| SCK      | GPIO11 SCLK (Pin 23) |
| SI       | GPIO10 MOSI (Pin 19) |
| SO       | GPIO9 MISO (Pin 21)  |
| INT      | GPIO 25 (Pin 22)     |

- DS1307 chip RTC module is connected to I2C bus, my particular module is occupying GPIO socket at 1-10 pins with I2C bus pins passthrough

- connect OLED module to I2C. By default the display is set to SPI mode. To set the display to I2C mode, remove resistor R1 and set the address with RO (default is 0x3D). OLED module I2C bus wiring:

| OLED | RPI GPIO     | DS1307 |
| ---- | ------------ | ------ |
| VCC  | 3.3V (pin 1) | 3V3    |
| DIN  | SDA (pin 3)  | SDA    |
| CLK  | SCL (pin 5)  | SCL    |
| GND  | GND (pin 6)  | GND    |

- OLED module could also be connected to SPI1 (SPI1 pins are GPIO 16, 17, 18, 19, 20, 21)

| OLED | RPI GPIO                   |
| ---- | -------------------------- |
| VCC  | 3.3V (pin 1)               |
| DIN  | SPI1 MOSI GPIO 20 (pin 38) |
| CS   | SPI1 CE0 GPIO 18 (pin 12)  |
| RES  | RES GPIO 27 (pin 13)       |
| GND  | GND (pin 6)                |
| CLK  | SPI1 SCLK GPIO 21 (pin 40) |
| DC   | GPIO 26 (pin 37)           |

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

## setting up

- flash the image, then remount and edit the file `pikvm.txt` on the boot partition
- nano pikvm.txt
  - ```
      ETH_ADDR=192.168.1.150/24
      ETH_DNS=192.168.1.1
      ETH_GW=192.168.1.1
      WIFI_ESSID='tempwifi'
      WIFI_PASSWD='9eu8xdexm08rfh0w9erf9ewf09wexr'
      WIFI_REGDOM=CH
      WIFI_ADDR=192.168.18.150/24
      WIFI_DNS=192.168.18.1
      WIFI_GW=192.168.18.1
    ```
- boot
- http://192.168.18.150 user `admin`, password `admin`
- `ssh root@192.168.18.150` user `root`, password `root`
- change passwords:
  - `rw`
    `passwd root`  
    `kvmd-htpasswd set admin`  
    `ro`
- update:

  - `rw`
    `pacman -Syy`
    `pacman -S pikvm-os-updater iperf3 chrony`
    `pikvm-update`
    `ro`

- set up the ethernet module:

  - `rw`
  - `nano /boot/config.txt`
    - ```
        dtparam=spi=on
        dtoverlay=enc28j60,int_pin=25,speed=25000000
      ```
  - `ro`
  - `sudo /sbin/reboot`
  - `lsmod`
  - `dmesg | grep Ethernet`
  - switch to full duplex script:

    - `rw`
    - `nano /etc/systemd/system/enc28j60-full-duplex.service`

      - ```
        [Unit]
        Description=ENC28J60 Full Duplex
        After=network.target

        [Service]
        Type=oneshot
        ExecStart=/usr/bin/ethtool -s eth0 speed 10 duplex full autoneg off

        [Install]
        WantedBy=multi-user.target
        ```

    - `systemctl enable enc28j60-full-duplex.service`
    - `systemctl start enc28j60-full-duplex.service`
    - `ro`

- set up the OLED display for use with SPI (enable SPI1):

  - `rw`
  - `nano /boot/config.txt`
    - ```
        dtoverlay=spi1-3cs,
      ```
  - `systemctl enable --now kvmd-oled kvmd-oled-reboot kvmd-oled-shutdown`
  - `ro`
  - `sudo /sbin/reboot`
  - `lsmod`
  - `dmesg | grep spi`

- set up the OLED display for use with I2C (desolder R1 and connect to I2C bus):

  - `rw`
  - `nano /boot/config.txt`

    - ```
        dtparam=i2c_arm=on
      ```

  - `nano /etc/modules-load.d/kvmd.conf`

    - ```
        i2c-dev
      ```

  - `systemctl enable --now kvmd-oled kvmd-oled-reboot kvmd-oled-shutdown`
  - `ro`
  - `sudo /sbin/reboot`

- set up rtc (optional):

  - `rw`
  - `nano /boot/config.txt`
    - ```
        dtoverlay=i2c-rtc,ds1307
      ```
  - `systemctl restart chronyd`
  - `date`
  - `hwclock --show`
  - `hwclock -w`
  - `crontab -e`

    - ```
      @reboot hwclock -r
      ```

  - `timedatectl set-timezone Asia/Shanghai`
  - `ro`

- change ip:

  - `rw`
  - nano /etc/systemd/network/eth0.network

    - ```
        [Match]
        Name=eth0


        [Network]
        Address=192.168.19.150/24
        Gateway=192.168.19.1
        DNS=192.168.19.1

      ```

  - `systemctl enable systemd-networkd.service`
  - `systemctl enable systemd-resolved.service`
  - `systemctl restart systemd-networkd.service`
  - `systemctl restart systemd-resolved.service`
  - or temporarily:
    - ip a show
    - ip link set eth0 up
    - ip a add 192.168.19.150/24 broadcast + dev eth0
      - ip a del 192.168.19.150/24 dev eth0
    - ip r show
    - ip r add 0.0.0.0/0 via 192.168.19.1 dev eth0
  - `ro`
  - `ethtool eth0`

- change wifi parameters:

  - `rw`
  - `nano /etc/systemd/network/wlan0.network`

    - ```
        [Match]
        Name=wlan0

        [Network]
        Address=192.168.18.150/24
        DNS=192.168.18.1
        DNSSEC=no

        [Route]
        Gateway=192.168.18.1
        # https://github.com/pikvm/pikvm/issues/583
        Metric=50
      ```

    - `wpa_passphrase 'MyNetwork' 'P@assw0rd' > /etc/wpa_supplicant/wpa_supplicant-wlan0.conf`
    - `chmod 640 /etc/wpa_supplicant/wpa_supplicant-wlan0.conf`
    - `systemctl enable wpa_supplicant@wlan0.service`
    - `systemctl restart wpa_supplicant@wlan0.service`
    - `ro`

- configure the KVM software

  - `rw`
  - `nano /etc/kvmd/override.yaml`
    An indent of 4 spaces is used. Comments starts with the `#` symbol. more: https://technotim.live/posts/pikvm-at-scale/#pikvm-config
    - ```
      kvmd:
          gpio:
              drivers:
                  wol_server1:
                      type: wol
                      mac: 00:00:00:00:00:00
                  wol_server2:
                      type: wol
                      mac: 00:00:00:00:00:00
                  reboot:
                      type: cmd
                      cmd: [/usr/bin/sudo, reboot]
                  restart_service:
                      type: cmd
                      cmd: [/usr/bin/sudo, systemctl, restart, kvmd]
              scheme:
                  pikvm_led:
                      pin: 0
                      mode: input
                  wol_server1:
                      driver: wol_server1
                      pin: 0
                      mode: output
                      switch: false
                  wol_server2:
                      driver: wol_server2
                      pin: 0
                      mode: output
                      switch: false
                  reboot_button:
                      driver: reboot
                      pin: 0
                      mode: output
                      switch: false
                  restart_service_button:
                      driver: restart_service
                      pin: 0
                      mode: output
                      switch: false
              view:
                  table:
                      - ["#server3", ]
                      - ["#server2", "wol_server1 | WoL"]
                      - ["#server1", "wol_server2 | WoL"]
                      - ["#PiKVM", "pikvm_led|green", "restart_service_button|confirm|Service", "reboot_button|confirm|Reboot"]
      ```
