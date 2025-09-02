# 3D Printer Configurations - Desciption

Base configuration files, notes, and documentation for my 3D printers.

# Git & Submodules Setup

```
# Set to the correct Printer
export PRINTER_DIR=PRINTER_TARGETED

sudo git clone https://github.com/kakwa/3dprinter-configs /home/3dprinter-configs
sudo mv $HOME ${HOME}.bk
sudo ln -s /home/3dprinter-configs/${PRINTER_DIR} $HOME
sudo rsync -Pizza ${HOME}.bk/ $HOME/
cd ~/
sudo chown -R ${USER}:${USER} /home/3dprinter-configs
git submodule update --init --recursive
reboot
```

## Printers

### Voron 0.2 - Fly Gemini v3

Voron 0.2 Siboor kit with FLY Gemini v3 board.

Configuration in: [voron0.2_fly-gemini](./voron0.2_fly-gemini/).

### Ender 3 Pro - Fly Gemini v3

Configuration in: [ender3_fly-gemini](./ender3_fly-gemini/).

# Board Setup

## Mellow Fly Gemini v3 Board

### Links

* **Documentation**: https://kakwasfork.github.io/mellow-3d.github.io
* **Documentation (.md files)**: https://github.com/kakwasfork/mellow-3d.github.io
* **Misc Board Assets & Binaries**: https://github.com/kakwasfork/Fly-Gemini-V3

### Downloads

#### Cleaned-Up Image
* **Armbian 13 (2025.08.16)**: [armbian-13_mellow-fly-gemini-v3.2025.08.16.img.xz](https://github.com/kakwa/3dprinter-configs/releases/download/2025.08.15/armbian-13_mellow-fly-gemini-v3.2025.08.16.img.xz) - Custom Armbian 13 image optimized for Fly Gemini v3 with Klipper

#### Official Mellow Images
* **Mellow Official Image**: [FLY-v3.0_Flygemini_bullseye_current_5.10.85.img.xz](https://cdn.mellow.klipper.cn/IMG/Build/FLY-v3.0_Flygemini_bullseye_current_5.10.85.img.xz) - Official Mellow Armbian Bullseye image with kernel 5.10.85

#### Package From Mellow
* **Board Support Package**: [armbian-bsp-cli-flygemini_21.11.0-trunk_arm64.deb](https://github.com/kakwa/3dprinter-configs/releases/download/2025.08.15/armbian-bsp-cli-flygemini_21.11.0-trunk_arm64.deb) - Armbian BSP package for Fly Gemini v3
* **Device Tree Blob**: [linux-dtb-current-sunxi64_21.11.0-trunk_arm64.deb](https://github.com/kakwa/3dprinter-configs/releases/download/2025.08.15/linux-dtb-current-sunxi64_21.11.0-trunk_arm64.deb) - Linux device tree blob for Allwinner H6 SoC
* **Linux Kernel Image**: [linux-image-current-sunxi64_21.11.0-trunk_arm64.deb](https://github.com/kakwa/3dprinter-configs/releases/download/2025.08.15/linux-image-current-sunxi64_21.11.0-trunk_arm64.deb) - Linux kernel image for Allwinner H6 SoC
* **U-Boot Bootloader**: [linux-u-boot-flygemini-current_21.11.0-trunk_arm64.deb](https://github.com/kakwa/3dprinter-configs/releases/download/2025.08.15/linux-u-boot-flygemini-current_21.11.0-trunk_arm64.deb) - U-Boot bootloader for Fly Gemini v3
* **HID Bootloader**: [hid_bootloader.bin](https://github.com/kakwa/3dprinter-configs/releases/download/2025.08.15/hid_bootloader.bin) - Human Interface Device (HID) Bootloader for the STM32 MCU.
* **FLY-Config**: [FLY-Config](https://github.com/kakwa/3dprinter-configs/releases/download/2025.08.15/FLY-Config) - Configuration/Setup Program (Golang CLI).
* **Fly-Tool**: [Fly-Tools](https://github.com/kakwa/3dprinter-configs/releases/download/2025.08.15/Fly-Tools) - Firmware & MCU/USB Info/Admin tools (Golang Web Server, Port 9999).

### Kernel Update

TODO, still stuck on a 5.10.85 kernel.

### Flashing Instructions

You need a +8GB mciroSD card.

Copy Image on SD card:

```bash
# Replace /dev/sdX with your actual device (be very careful!)
# Use lsblk to identify your microSD card
lsblk

# Flash the image (replace /path/to/image.img with actual path)
curl -q -L "https://github.com/kakwa/3dprinter-configs/releases/download/2025.08.15/armbian-13_mellow-fly-gemini-v3.2025.08.16.img.xz" | \
   xz -d | \
   sudo dd bs=4M status=progress conv=fsync of=/dev/sdX

# Sync to ensure all data is written
sudo sync
```

### First Boot

The default credentials are:

- **Username**: `fly`
- **Password**: `mellow`

Once connected to the board, you can resize the root FS:

```bash
sudo resize2fs /dev/mmcblk0p2
```

Regenerate ssh hosts keys:
```bash
sudo /bin/rm -v /etc/ssh/ssh_host_*
sudo dpkg-reconfigure openssh-server
sudo systemctl restart ssh
```

Set a proper password:
```bash
passwd
sudo passwd
```

Add you ssh key(s):

```bash
# get your keys from your main computer
# cat ~/.ssh/id_*pub

mkdir ~/.ssh
cat >~/.ssh/authorized_keys <<EOF
<your keys>
EOF

chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

Connect the board to wifi if necessary:

```bash
sudo nmtui
```

# Klipper Build and Install (MCU & Display Firmwares)

## Klipper Update

```bash
# Navigate to Klipper directory
cd ~/klipper

# Pull latest changes
git pull
```

## VirtualEnv Setup

```bash
# Create virtual environment
python3 -m venv ~/klippy-env

# Activate virtual environment
source ~/klippy-env/bin/activate

# Install dependencies
pip install -r ~/klipper/scripts/klippy-requirements.txt
```

## Configuration and Build

### MCU Firmware (STM32)

```bash
# Navigate to Klipper directory
cd ~/klipper

# Copy the configuration over
cp ../klipper_config/mcu.config ./.config

# Check it for STM32F405
make menuconfig
# Select: Micro-controller Architecture -> STMicroelectronics STM32
# Select: Processor model -> STM32F405
# Select: Bootloader offset -> 32KiB bootloader
# Select: Clock Reference -> 8 MHz crystal
# Select: Communication interface -> USB (on PA11/PA12)

# Build firmware
make clean
make

# The firmware will be saved as: out/klipper.bin
```

## Firmware Flashing

`FIXME` should not be a board specific command.

Run the following command
```
sudo geminiv3-flash
```

If necessary, do a [Factory Reset](https://mellow-3d.github.io/fly-gemini_v3_boot_oem.html) of the MCU.

# Moonraker Install (Management UI Backend)

## Moonraker Update

```bash
# Navigate to Moonraker directory
cd ~/moonraker

# Pull latest changes
git pull

# Restart Moonraker service
sudo systemctl restart moonraker
```

## VirtualEnv Setup

```bash
# Create virtual environment
python3 -m venv ~/moonraker-env

# Activate virtual environment
source ~/moonraker-env/bin/activate

# Install dependencies
pip install -r ~/moonraker/scripts/moonraker-requirements.txt

# Install Moonraker
pip install -e ~/moonraker
```

# System & Services Setup

## Reset Print History Stats

```bash
curl -X POST http://127.0.0.1:7125/server/history/reset_totals
```

## Hostname & mDNS/Avahi

Optional, but make the printer easier to find on the network:

```bash
sudo echo "YOUR_PRINTER_NAME" >/etc/hostname
sudo hostname -F /etc/hostname
```

```bash
sudo cp -r root/etc/avahi/* /etc/avahi/
sudo apt install avahi-daemon
```

The Printer should become visible as `YOUR_PRINTER_NAME.local`.

## Klipper SystemD Services

```bash
# Copy system configuration files to their proper locations
sudo cp -r root/etc/systemd/system/* /tc/systemd/system/

# Reload systemd to recognize new services
sudo systemctl daemon-reload

# Enable and start the services
sudo systemctl enable klipper
sudo systemctl enable moonraker
sudo systemctl enable ustreamer

# Restart services
sudo systemctl restart klipper
sudo systemctl restart moonraker
```

## Nginx

```bash
sudo rm /etc/nginx/sites-enabled/*
sudo cp root/etc/nginx/sites-available/* /etc/nginx/sites-available/
sudo cp root/etc/nginx/sites-enabled/*   /etc/nginx/sites-enabled/
sudo systemctl restart nginx
sudo systemctl enable nginx
```

# UI - Mainsail/Fluidd

## Mainsail

### Get Current Version

```
jq . mainsail/release_info.json 
```

### Update

```shell
# Home
cd ~/

# Get latest version using jq for cleaner JSON parsing
VERSION=$(curl -s "https://api.github.com/repos/mainsail-crew/mainsail/releases/latest" | \
          jq -r '.tag_name | ltrimstr("v")')

echo "Installing Mainsail version: $VERSION"

# Clean and recreate directory, then download and extract  
! [ -z "$VERSION" ] && rm -rf mainsail/ && mkdir -p mainsail/ && cd mainsail/ && \
curl -sL "https://github.com/mainsail-crew/mainsail/releases/download/v${VERSION}/mainsail.zip" -o mainsail.zip && \
unzip  mainsail.zip && \
rm -f mainsail.zip
cd -
```

### Enable

```
sudo ln -fs /etc/nginx/sites-available/mainsail /etc/nginx/sites-enabled/klipper_webui
sudo systemctl restart nginx
```

## Fluidd

### Get Current Version

```
jq . fluidd/release_info.json 
```

### Update

```shell
# Home
cd ~/

# Get latest version from GitHub API
VERSION=$(curl -s "https://api.github.com/repos/fluidd-core/fluidd/releases/latest" | \
          grep '"tag_name":' | \
          sed -E 's/.*"v([^"]+)".*/\1/')

echo "Installing Fluidd version: $VERSION"

# Clean and recreate directory, then download and extract
! [ -z "$VERSION" ] && rm -rf fluidd/ && mkdir -p fluidd/ && cd fluidd/ && \
curl -sL "https://github.com/fluidd-core/fluidd/releases/download/v${VERSION}/fluidd.zip" -o fluidd.zip && \
unzip fluidd.zip && \
rm -f fluidd.zip
cd -
```

### Enable

```
sudo ln -fs /etc/nginx/sites-available/fluidd /etc/nginx/sites-enabled/klipper_webui
sudo systemctl restart nginx
```
