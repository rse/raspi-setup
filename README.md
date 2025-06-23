
Raspberry Pi Setup
==================

This small repository collects the essential information from Dr.
Ralf S. Engelschall for making setups based on the awesome [Raspberry
Pi](https://www.raspberrypi.com/) mini-computing platform.

Favorite Devices
----------------

- [Raspberry Pi 5](https://www.raspberrypi.com/products/raspberry-pi-5/):
  The traditional Raspberry Pi, available in version 5 since 2023.
  It is based on the 64-bit 2.4GHz ARMv8 quad-core Cortex-A76 CPU, 8GB SDRAM,
  and has microSD slot, 2 USB ports, 2.4/5.0 GHz Wi-Fi, Bluetooth 4.2+BLE, HDMI and Ethernet.
  Can be purchased as an all-in-one product as
  [GeeekPi Raspberry Pi 5 Starter Kit](https://www.amazon.de/dp/B0CSBVH8K9).
  Purchase a Class 1 (A1) and at least speed class C10 microSD card like
  [SanDisk Extreme Pro](https://www.amazon.de/dp/B09X7BYSFG).

- [Raspberry Pi Zero 2 W](https://www.raspberrypi.com/products/raspberry-pi-5/):
  The smallest traditional Raspberry Pi, available in version 2 since 2021.
  It is based on the 64-bit 1.0GHz ARMv8 quad-core Cortex-A53 CPU, 512MB
  SDRAM, and has microSD slot, 2 USB ports, 2.4 GHz Wi-Fi, Bluetooth 4.2+BLE, and Mini-HDMI.
  Can be purchased as an all-in-one product as
  [GeeekPi Raspberry Pi Zero 2 W Starter Kit](https://www.amazon.de/dp/B0BHS3NG4B).
  Purchase a Class 1 (A1) and at least speed class C10 microSD card like
  [SanDisk Extreme Pro](https://www.amazon.de/dp/B09X7BYSFG).

Favorite OS
-----------

- [Raspberry Pi OS (64-bit)](https://www.raspberrypi.com/software/operating-systems/):
  The traditional desktop version of the Raspberry Pi OS. Primary interesting
  for [Raspberry Pi 500](https://www.raspberrypi.com/products/raspberry-pi-500/) and
  for [Raspberry Pi 5](https://www.raspberrypi.com/products/raspberry-pi-5/).

- [Raspberry Pi OS Lite (64-bit)](https://www.raspberrypi.com/software/operating-systems/)
  A reduced
  server version of the Raspberry Pi OS. Primary interesting
  for [Raspberry Pi 5](https://www.raspberrypi.com/products/raspberry-pi-5/)
  and [Raspberry Pi Zero 2 W](https://www.raspberrypi.com/products/raspberry-pi-5/).

Setup Level 0
-------------

- Purchase a [Raspberry Pi](https://www.raspberrypi.com/) device
  and install "Raspberry Pi OS" or "Raspbeery Pi OS Lite" (both based on Debian 12)
  as the operating system (OS) onto it with the
  official [Raspberry Pi OS Imager](https://www.raspberrypi.com/software/).
  Optimally, pre-configure the Raspberry Pi OS with SSH access and a
  username/password during the imaging process. Then connect the device
  to the network, boot it, let it get an IP with DHCP and then login
  with SSH.

- Upgrade to the latest OS state:

    ```
    sudo apt update
    sudo apt upgrade -y
    sudo apt autoremove --purge
    ```

- **OPTIONALLY** upgrade the Raspberry Pi firmware to the latest version.
  But **CAUTION**, this can make you a lot of trouble as it manually installs
  the latest Linux kernel instead of using the package management and as a
  result it (as of 2025-06-22) at least breaks the initramfs and this way
  the overlayfs feature:

    ```
    sudo rpi-update
    ```

- Finally, reboot the device:

    ```
    sudo reboot
    ```

Setup Level 1
-------------

- *OPTIONAL*: Raspberry Pi 5 only: underclock CPU to reduce power consumption and overall device heat:

    ```
    sudo vi /boot/firmware/config.txt
    | arm_boost=0
    | arm_freq=720
    ```

- *OPTIONAL*: Install more essential tools:

    ```
    sudo apt install -y bash tmux vim git make
    ```

- **OPTIONAL**: Install the latest Node.js 24 version:

    ```
    curl -fsSL https://deb.nodesource.com/setup_24.x -o nodesource_setup.sh
    sudo bash nodesource_setup.sh
    sudo apt-get install nodejs -y
    rm -f nodesource_setup.sh
    ```

- *OPTIONAL*: Reduce services of Raspberry Pi OS:

    ```
    sudo systemctl disable avahi-daemon
    sudo systemctl disable bluetooth.service
    sudo systemctl disable ModemManager
    ```

- Configure system for non-network booting into console mode:

    ```
    sudo raspi-config nonint do_boot_wait 0
    sudo raspi-config nonint do_boot_behaviour B1
    ```

- *OPTIONAL*: Configure WLAN access:

    ```
    sudo nmcli radio wifi on
    sudo raspi-config nonint do_wifi_country "<cc>"
    sudo raspi-config nonint do_wifi_ssid_passphrase "<ssid>" "<passphrase>"
    ```

- *OPTIONAL*: Configure fixed IP addresses on network interface:

    ```
    sudo nmcli con list
    sudo nmcli con mod "<id>" ipv4.addresses 10.0.0.100/24 \
        ipv4.gateway 10.0.0.1 ipv4.dns 10.0.0.1 ipv4.method manual
    sudo systemctl restart NetworkManager
    ```

    Notice: you can revert back to DHCP later with:

    ```
    sudo nmcli con mod "Wired connection 1" ipv4.method auto
    sudo systemctl restart NetworkManager
    ```

- Configure system for read-only operation:

    ```
    curl -o /usr/sbin/overlayfs-chroot https://github.com/rse/raspi-setup/raw/refs/heads/master/overlayfs-chroot
    chmod 755 /usr/sbin/overlayfs-chroot
    sudo raspi-config nonint enable_overlayfs
    sudo raspi-config nonint enable_bootro
    ```

- Finally, reboot the device:

    ```
    sudo reboot
    ```

Setup Level 2
-------------

Before any further installations, enter the writable filesystem with:

```
sudo overlayfs-chroot
sudo -u studio bash
```

- **EXAMPLE**: Install the Busylight service:

    ```
    cd $HOME
    git clone https://github.com/rse/busylight
    cd busylight
    make build
    make install
    make start
    ```

    You can later upgrade to the latest version at any time with:

    ```
    cd $HOME
    cd busylight
    make upgrade
    ```

- **EXAMPLE**: Install the Bitfocus Companion Sattelite service:

    ```
    curl https://raw.githubusercontent.com/bitfocus/companion-satellite/main/pi-image/install.sh | bash
    ```

    You can later upgrade to the latest version at any time with:

    ```
    sudo satellite-update
    ```

After any installations, exit the writable filesystem with:

```
exit
exit
sudo reboot
```

