# Mellow Fly Gemini v3 Board

## Links

* **Documentation**: https://kakwasfork.github.io/mellow-3d.github.io
* **Documentation (.md files)**: https://github.com/kakwasfork/mellow-3d.github.io
* **Misc Board Assets & Binaries**: https://github.com/kakwasfork/Fly-Gemini-V3


## Downloads

### Custom Armbian Images
* **Armbian 13 (2025.08.16)**: [armbian-13_mellow-fly-gemini-v3.2025.08.16.img.xz](https://github.com/kakwa/3dprinter-configs/releases/download/2025.08.15/armbian-13_mellow-fly-gemini-v3.2025.08.16.img.xz) - Custom Armbian 13 image optimized for Fly Gemini v3 with Klipper

### Official Mellow Images
* **Mellow Official Image**: [FLY-v3.0_Flygemini_bullseye_current_5.10.85.img.xz](https://cdn.mellow.klipper.cn/IMG/Build/FLY-v3.0_Flygemini_bullseye_current_5.10.85.img.xz) - Official Mellow Armbian Bullseye image with kernel 5.10.85

### Package From Mellow
* **Board Support Package**: [armbian-bsp-cli-flygemini_21.11.0-trunk_arm64.deb](https://github.com/kakwa/3dprinter-configs/releases/download/2025.08.15/armbian-bsp-cli-flygemini_21.11.0-trunk_arm64.deb) - Armbian BSP package for Fly Gemini v3
* **Device Tree Blob**: [linux-dtb-current-sunxi64_21.11.0-trunk_arm64.deb](https://github.com/kakwa/3dprinter-configs/releases/download/2025.08.15/linux-dtb-current-sunxi64_21.11.0-trunk_arm64.deb) - Linux device tree blob for Allwinner H6 SoC
* **Linux Kernel Image**: [linux-image-current-sunxi64_21.11.0-trunk_arm64.deb](https://github.com/kakwa/3dprinter-configs/releases/download/2025.08.15/linux-image-current-sunxi64_21.11.0-trunk_arm64.deb) - Linux kernel image for Allwinner H6 SoC
* **U-Boot Bootloader**: [linux-u-boot-flygemini-current_21.11.0-trunk_arm64.deb](https://github.com/kakwa/3dprinter-configs/releases/download/2025.08.15/linux-u-boot-flygemini-current_21.11.0-trunk_arm64.deb) - U-Boot bootloader for Fly Gemini v3
* **HID Bootloader**: [hid_bootloader.bin](https://github.com/kakwa/3dprinter-configs/releases/download/2025.08.15/hid_bootloader.bin) - Human Interface Device (HID) Bootloader for the STM32 MCU.

## Flashing Instructions

You need a +8GB mciroSD card.

Copy Image on SD card:

```bash
# Replace /dev/sdX with your actual device (be very careful!)
# Use lsblk to identify your microSD card
lsblk

# Flash the image (replace /path/to/image.img with actual path)
curl -L "https://github.com/kakwa/3dprinter-configs/releases/download/2025.08.15/armbian-13_mellow-fly-gemini-v3.2025.08.16.img.xz" | \
   xz -d | \
   sudo dd bs=4M status=progress conv=fsync of=/dev/sdX

# Sync to ensure all data is written
sudo sync
```

## First Boot

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

# Kernel Update

TODO, still stuck on a 5.10.85 kernel. FYI trying a simple package upgrade results in a boot failure.

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

# Configure for STM32F103
make menuconfig
# Select: Micro-controller Architecture -> STMicroelectronics STM32
# Select: Processor model -> STM32F103
# Select: Bootloader offset -> 28KiB bootloader
# Select: Clock Reference -> 8 MHz crystal
# Select: Communication interface -> USB (on PA11/PA12)

# Build firmware
make clean
make

# The firmware will be saved as: out/klipper.bin
```

## Firmware Flashing

TODO

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


# Restart nginx to load new configurations
sudo systemctl restart nginx
```

## Udev

TODO

## NGinx

TODO

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
