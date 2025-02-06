# Custom DIY IP KVM, pikvm on Raspberry Pi Zero 2W with ethernet

## Description

Based on https://github.com/pikvm/pikvm

## Use cases:

- When a secure, truly open-source solution (unlike [some closed-source KVM products](https://github.com/sipeed/NanoKVM/issues?q=is%3Aissue%20state%3Aopen%20%20security) [are](https://www.youtube.com/watch?v=plJGZQ35Q6I)) is needed
- A jump device to Wake-on-LAN other devices
- Building with off-the-shelf components as a safeguard against supply chain attacks

## Main differences

between my build and the [official DIY V2 build instructions for Raspberry Pi Zero 2 W](https://docs.pikvm.org/v2/#required-parts):

- My hardware setup has ENC28J60 chip based ethernet adapter, connected to GPIO through SPI protocol (four-wire serial bus)
- Full duplex mode for the ethernet adapter.
- I2C OLED display
- Didn't utilize ATX power control
- Powered by the keyboard/mouse interface cable plugged into the USB socket of the rpi (can be changed to external power by cutting +5V wire in this cable and adding another cable plugged into PWR socket of the rpi)
- RTC module

## Build pictures

| ![untitled-1.jpg](pictures/small/untitled-1.jpg) | ![untitled-2.jpg](pictures/small/untitled-2.jpg) | ![untitled-3.jpg](pictures/small/untitled-3.jpg) |
| :----------------------------------------------: | :----------------------------------------------: | :----------------------------------------------: |

| ![untitled-4.jpg](pictures/small/untitled-4.jpg) | ![untitled-5.jpg](pictures/small/untitled-5.jpg) | ![untitled-6.jpg](pictures/small/untitled-6.jpg) |
| :----------------------------------------------: | :----------------------------------------------: | :----------------------------------------------: |

| ![untitled-7.jpg](pictures/small/untitled-7.jpg) | ![untitled-8.jpg](pictures/small/untitled-8.jpg) | ![untitled-9.jpg](pictures/small/untitled-9.jpg) |
| :----------------------------------------------: | :----------------------------------------------: | :----------------------------------------------: |

| ![untitled-10.jpg](pictures/small/untitled-10.jpg) | ![untitled-11.jpg](pictures/small/untitled-11.jpg) | ![untitled-13.jpg](pictures/small/untitled-13.jpg) |
| :------------------------------------------------: | :------------------------------------------------: | :------------------------------------------------: |

| ![untitled-14.jpg](pictures/small/untitled-13.jpg) | ![untitled-15.jpg](pictures/small/untitled-14.jpg) | ![untitled-16.jpg](pictures/small/untitled-17.jpg) |
| :------------------------------------------------: | :------------------------------------------------: | :------------------------------------------------: |

| ![untitled-17.jpg](pictures/small/untitled-18.jpg) | ![untitled-18.jpg](pictures/small/untitled-18.jpg) | ![untitled-19.jpg](pictures/small/untitled-19.jpg) |
| :------------------------------------------------: | :------------------------------------------------: | :------------------------------------------------: |

| ![untitled-20.jpg](pictures/small/untitled-20.jpg) | ![untitled-21.jpg](pictures/small/untitled-21.jpg) | ![Screenshot from 2024-09-27 22-17-45.png](pictures/Screenshot.png) |
| :------------------------------------------------: | :------------------------------------------------: | :-----------------------------------------------------------------: |

## Parts list

- a generic plastic case for PLC, 145x90x40 2 元
- HDMI to CSI2 adapter, TC35874XBG chip https://www.waveshare.com/wiki/HDMI_to_CSI_Adapter 129 元
- ribbon cable, 22pin to 15 pin, 20cm 8 元
- ENC28J60 chip based ethernet adapter (10Mbps) https://www.waveshare.com/wiki/ENC28J60_Ethernet_Board 14 元
- rp0 aluminum heatsink case 24 元
- USB mini to USB A cable
- 32Gb A1 U1 C10 microsd card 60 元
- SSD1315 chip based OLED display https://www.waveshare.com/wiki/0.96inch_OLED_Module 28 元
- Raspberry Pi zero 2 W - https://www.waveshare.com/wiki/Raspberry_Pi_Zero_2_W 165 元
- DS1307 RTC board 20 元 (optional, do not recommend, better use DS3231 chip based board)
- terminated colored wires 2 元
- zip ties
- HDMI cable

## Software

download https://files.pikvm.org/images/v2-hdmi-zero2w-latest.img.xz

or [compile from source](https://docs.pikvm.org/building_os/)

## Wiring

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

- DS1307 chip RTC module is connected to I2C bus (default address 0x68). If decide to use the module, then the screen wires need to be soldered to the back of the board.

- connect OLED module to I2C. By default the display is set to SPI mode. To set the display to I2C mode, remove the resistor R1 and change the address (the default is 0x3D) to 0x3C by shorting the resistor R2. OLED module I2C bus wiring:

| OLED board | RPI GPIO     |
| ---------- | ------------ |
| VCC        | 3.3V (pin 1) |
| DIN        | SDA (pin 3)  |
| CLK        | SCL (pin 5)  |
| GND        | GND (pin 6)  |

## Setting up

- Flash the image, then remount and edit the file `pikvm.txt` on the boot partition
- `nano pikvm.txt`
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
- Boot
- http://192.168.18.150 user `admin`, password `admin`
- `ssh root@192.168.18.150` user `root`, password `root`
- Change the passwords:
  - `rw`
  - `passwd root`
  - `kvmd-htpasswd set admin`
  - `ro`
- Update:

  Make sure there is Internet connectivity

  - `rw`
  - `pacman -Syy`
  - `pacman -S pikvm-os-updater iperf3 chrony i2c-tools`
  - `pikvm-update`
  - `ro`
  - on error `failed to synchronize any databases`:
    - `rm -f /var/lib/pacman/db.lck`

- Disable IPv6:

  - `rw`
  - `echo ipv6.disable_ipv6=1 >> /boot/cmdline.txt`
  - `ro`

- Set up the ethernet module:

  - `rw`
  - `nano /boot/config.txt`
    - ```
        dtparam=spi=on
        dtoverlay=enc28j60,int_pin=25,speed=20000000
      ```
  - `ro`
  - `sudo /sbin/reboot`
  - `lsmod`
  - `dmesg | grep Ethernet`
  - switch to full duplex:

    - `nano /etc/systemd/system/enc28j60-full-duplex.service`

      - ```
            [Unit]
            Description=ENC28J60 Full Duplex
            After=multi-user.target

            [Service]
            Type=simple
            Restart=always
            ExecStartPre=/bin/sleep 60
            ExecStart=ifconfig eth0 down && ethtool -s eth0 speed 10 duplex full autoneg off && systemctl restart systemd-networkd.service && systemctl restart systemd-networkd.service && ifconfig eth0 up

            [Install]
            WantedBy=multi-user.target
        ```

    - `chmod 644 /etc/systemd/system/enc28j60-full-duplex.service`
    - `systemctl daemon-reload`
    - `systemctl enable enc28j60-full-duplex.service`
    - `systemctl start enc28j60-full-duplex.service`
    - `ro`

- Test the stability of the ethernet connection:

  - on the pikvm:

    - `iperf3 -s 192.168.1.150`

  - on the host machine:
    - `iperf3 -c  192.168.1.150 -P 2 -t 30`

- Set up the OLED display for use with I2C:

  - `rw`
  - `nano /boot/config.txt`

    - ```
        dtparam=i2c_arm=on
      ```

  - `nano /etc/modules-load.d/raspberrypi.conf`

    - ```
        i2c-dev
      ```

  - `systemctl enable --now kvmd-oled kvmd-oled-reboot kvmd-oled-shutdown`
  - `ro`
  - `sudo /sbin/reboot`
  - `i2cdetect -y 1` should display 3c at the address 0x3C

- (optional) Set up rtc:

  - `rw`
  - `nano /boot/config.txt`
    - ```
        dtoverlay=i2c-rtc,ds1307
      ```
  - `sudo /sbin/reboot`
  - `i2cdetect -y 1` should display UU at the address 0x68
  - `timedatectl set-timezone Asia/Shanghai`
  - `systemctl enable chronyd`
  - `systemctl start chronyd`
  - `date`
  - `hwclock --show`
  - `hwclock --systohc`, do it once to write the current time to the RTC
  - add read from the hardware clock at boot:

    - `nano /etc/systemd/system/hwclock-sync.service`

      - ```
        [Unit]
        Description=Sync hwclock from system clock
        After=systemd-modules-load.service
        Before=systemd-timesyncd.service

        [Service]
        Type=oneshot
        ExecStart=/usr/bin/hwclock --hctosys
        RemainAfterExit=yes

        [Install]
        WantedBy=multi-user.target
        ```

    - `chmod 644 /etc/systemd/system/hwclock-sync.service`
    - `systemctl daemon-reload`
    - `systemctl enable hwclock-sync.service`

  - `ro`

- (optional) Change ethernet ip:

  - `rw`
  - `nano /etc/systemd/network/eth0.network`

    - ```
        [Match]
        Name=eth0


        [Network]
        Address=192.168.1.150/24
        Gateway=192.168.1.1
        DNS=192.168.1.1

      ```

  - `systemctl enable systemd-networkd.service`
  - `systemctl enable systemd-resolved.service`
  - `systemctl restart systemd-networkd.service`
  - `systemctl restart systemd-resolved.service`
  - or temporarily:
    - ip a show
    - ip link set eth0 up
    - ip a add 192.168.1.150/24 broadcast + dev eth0
      - ip a del 192.168.1.150/24 dev eth0
    - ip r show
    - ip r add 0.0.0.0/0 via 192.168.1.1 dev eth0
  - `ro`
  - `ethtool eth0`

- (optional) Change wifi ip and parameters (https://docs.pikvm.org/wifi/#setting-up-wi-fi-manually):

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

    - `wpa_passphrase 'tempwifi' '9eu8xdexm08rfh0w9erf9ewf09wexr' > /etc/wpa_supplicant/wpa_supplicant-wlan0.conf`
    - `chmod 640 /etc/wpa_supplicant/wpa_supplicant-wlan0.conf`
    - `systemctl enable wpa_supplicant@wlan0.service`
    - `systemctl start wpa_supplicant@wlan0.service`
    - `ro`

- Disable wifi:

  - `rw`
  - `echo > /etc/wpa_supplicant/wpa_supplicant-wlan0.conf`
  - `systemctl disable wpa_supplicant@wlan0.service`
  - `systemctl stop wpa_supplicant@wlan0.service`
  - `ro`

- Disable talking to the outside STUN servers:

  - `rw`
  - `systemctl disable --now kvmd-janus`
  - `systemctl enable --now kvmd-janus-static`
  - `ro`

- Configure the KVM software

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
  - `ro`

## Results

- It's working as it supposed to
- Power consumption 2.5W when streaming and half of that when idle
- The adapter doesn't support auto-negotiation, so full duplex is not supposed to work with auto-negotiation on the other end (requres setting full duplex mode manually on both ends), **but it is working** with every usb-ethernet adapter I tested as well as with tl-sf1008m 100M switch. Default half-duplex mode connection is unstable and hangs on traffic.
- Ethernet stability:
  - ```
    [ ID] Interval           Transfer     Bitrate         Retr
    [  5]   0.00-30.00  sec  20.5 MBytes  5.73 Mbits/sec    2             sender
    [  5]   0.00-30.01  sec  18.9 MBytes  5.28 Mbits/sec                  receiver
    [  7]   0.00-30.00  sec  14.0 MBytes  3.91 Mbits/sec    2             sender
    [  7]   0.00-30.01  sec  11.9 MBytes  3.32 Mbits/sec                  receiver
    [SUM]   0.00-30.00  sec  34.5 MBytes  9.65 Mbits/sec    4             sender
    [SUM]   0.00-30.01  sec  30.8 MBytes  8.60 Mbits/sec                  receiver
    ```

## References

- https://github.com/pikvm/pikvm
- RPI GPIO pinout: https://pinout.xyz/pinout/spi
- SPI reference: https://www.analog.com/en/resources/analog-dialogue/articles/introduction-to-spi-interface.html
- Overlays reference: https://raw.githubusercontent.com/raspberrypi/firmware/master/boot/overlays/README
- Systemd units: https://www.thedigitalpictureframe.com/ultimate-guide-systemd-autostart-scripts-raspberry-pi/
- ENC28J60 datasheet: https://ww1.microchip.com/downloads/en/DeviceDoc/39662c.pdf
- https://www.instructables.com/Super-Cheap-Ethernet-for-the-Raspberry-Pi/
- https://raspi.tv/2015/ethernet-on-pi-zero-how-to-put-an-ethernet-port-on-your-pi
- https://windix.medium.com/pikvm-with-raspberry-pi-zero-2-w-b5f9d4a6ff32
- https://docs.pikvm.org/v2/
- https://technotim.live/posts/pikvm-at-scale/
- https://technotim.live/posts/wake-on-lan/
- https://www.makeuseof.com/how-to-build-raspberry-pi-kvm/
- https://pimylifeup.com/raspberry-pi-rtc/
- https://maker.pro/raspberry-pi/tutorial/how-to-add-an-rtc-module-to-raspberry-pi
