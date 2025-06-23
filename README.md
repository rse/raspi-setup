
Raspberry Pi Setup
------------------

- Purchase a [Raspberry Pi](https://www.raspberrypi.com/) (most easily with an all-in-one kit
  like the [GeeekPi Starter Kit für Raspberry Pi 5](https://www.amazon.de/dp/B0CSBVH8K9?ref=ppx_yo2ov_dt_b_fed_asin_title&th=1)) and
  install "Raspberry Pi OS Lite" (based on Debian 12.10)
  as the operating system (OS) onto it with the
  official [Raspberry Pi OS Imager](https://www.raspberrypi.com/software/).
  Pre-configure the Raspberry Pi OS with SSH access and a username/password
  during the imaging process. Then connect the device to the
  network, boot it and login with SSH.

- Upgrade the OS to latest Debian 12 version:

    ```
    sudo apt update
    sudo apt upgrade -y
    sudo apt autoremove --purge
    ```

- Install the essential tools:

    ```
    sudo apt install -y vim bash git make
    ```

- Optionally upgrade the Raspberry Pi 5 firmware to the latest version.
  But CAUTION, this can make you a lot of trouble as it manually installs
  the latest Linux kernel instead of using the package management and as a
  result it (as of 2025-06-22) at least breaks the initramfs and this way
  the overlayfs feature:

    ```
    sudo rpi-update
    sudo reboot
    ```

- Underclock the Raspberry Pi 5 to reduce power consumption and heat:

    ```
    sudo vi /boot/firmware/config.txt
    | arm_boost=0
    | arm_freq=720
    ```

- Reduce services of Raspberry Pi OS:

    ```
    sudo systemctl disable avahi-daemon
    sudo systemctl disable bluetooth.service
    sudo systemctl disable ModemManager
    ```

- Install the latest Node.js 24 version:

    ```
    curl -fsSL https://deb.nodesource.com/setup_24.x -o nodesource_setup.sh
    sudo bash nodesource_setup.sh
    sudo apt-get install nodejs -y
    ```

- Install the Busylight service:

    ```
    cd $HOME
    git clone https://github.com/rse/busylight
    cd busylight
    make build
    make install
    make start
    ```

- Configure system for non-network booting into console mode:

    ```
    sudo raspi-config nonint do_boot_wait 0
    sudo raspi-config nonint do_boot_behaviour B1
    ```

- Optionally configure WLAN:

    ```
    sudo nmcli radio wifi on
    sudo raspi-config nonint do_wifi_country "<cc>"
    sudo raspi-config nonint do_wifi_ssid_passphrase "<ssid>" "<passphrase>"
    ```

- Optionally configure fixed IP addresses on Ethernet interface:

    ```
    sudo nmcli con mod "Wired connection 1" ipv4.addresses 10.1.0.94/24 \
        ipv4.gateway 10.1.0.1 ipv4.dns 10.1.0.1 ipv4.method manual
    sudo systemctl restart NetworkManager
    ```

    You can revert back to DHCP later with:

    ```
    sudo nmcli con mod "Wired connection 1" ipv4.method auto
    sudo systemctl restart NetworkManager
    ```

- Configure system for read-only operation:

    ```
    sudo raspi-config nonint enable_overlayfs
    sudo raspi-config nonint enable_bootro
    sudo reboot
    ```

