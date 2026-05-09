# dec-manifests — STM32MP branch

Manifest files for Diamond Technologies STM32MP DEC kits, based on
ST's OpenSTLinux distribution + the meta-dec-stm32 BSP layer.

For the i.MX flavor of dec-manifests, see branch `dec-scarthgap-6.6`.

---

## Quick Start

### 1. Install host build prerequisites

Build host should be Ubuntu 22.04 LTS or newer (Yocto Scarthgap target).
Install the standard Yocto-required packages plus a few ST tooling
extras:

```bash
sudo apt update
sudo apt install -y \
    bsdmainutils build-essential chrpath cpio debianutils diffstat \
    file gawk gcc-multilib git git-lfs iputils-ping libacl1 \
    libegl1-mesa libgmp-dev libmpc-dev libsdl1.2-dev libssl-dev \
    libusb-1.0-0 lz4 pylint python3 python3-git python3-jinja2 \
    python3-pexpect python3-pip python3-subunit socat texinfo \
    unzip wget xterm xz-utils zstd
```

If your host is an unsupported distro you'll see a warning when sourcing
`envsetup.sh` later — read the message and install whatever it asks for.

### 2. Configure git identity

Required for `repo` to work cleanly:

```bash
git config --global user.name  "Your Name"
git config --global user.email "you@example.com"
```

### 3. Install the `repo` tool

```bash
mkdir -p ~/.bin
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/.bin/repo
chmod a+rx ~/.bin/repo
export PATH="${HOME}/.bin:${PATH}"
```

(Add the `export PATH=...` line to your `~/.bashrc` so it survives new
shells.)

### 4. Initialize and sync

```bash
mkdir stm-yocto-bsp && cd stm-yocto-bsp
repo init -u https://github.com/DiamondTechno/dec-manifests \
          -b dec-stm32-scarthgap \
          -m stm32mp-6.6.78-r2.xml
repo sync -c -j$(nproc)
```

After sync the tree looks like:

```
stm-yocto-bsp/
├── layers/
│   ├── openembedded-core/        # Yocto Scarthgap base
│   ├── meta-openembedded/
│   ├── meta-st/
│   │   ├── meta-st-openstlinux/  # ST OpenSTLinux distro
│   │   ├── meta-st-stm32mp/      # STM32MP BSP (ST upstream)
│   │   ├── meta-st-stm32mp-addons/
│   │   └── scripts/              # ST envsetup.sh
│   ├── meta-dec-common/          # DEC vendor-neutral userland
│   └── meta-dec-stm32/           # DEC STM32MP25 BSP
└── sources/
    ├── linux-stm32/              # Kernel working tree (externalsrc)
    └── uboot-stm32/              # U-Boot working tree (externalsrc)
```

### 5. Source the setup wrapper AND accept the ST EULA

The manifest installs `setup_dec_stm32.sh` at the repo root. It wraps
ST's envsetup.sh with the right DISTRO/MACHINE preset and auto-wires
the EXTERNALSRC pointers for the kernel/u-boot working trees, so you
don't have to navigate the menu or hand-edit local.conf.

```bash
source ./setup_dec_stm32.sh
```

On first run, the underlying envsetup.sh displays the STMicroelectronics
SLA0048 EULA covering proprietary firmware and the Vivante GPU userland.
**You must read and accept it** — without acceptance, the GPU userland
and DDR firmware won't be built.

| Prompt | Answer |
|---|---|
| Read EULA? | `y`  *(scroll through; `q` to exit pager when done)* |
| Accept EULA? | `y` |

Acceptance is recorded as `ACCEPT_EULA_stm32mp25-dec-kit = "1"` in
your build dir's `conf/local.conf`. After the EULA the wrapper:

  - Creates `build-openstlinuxweston-stm32mp25-dec-kit/` if needed
  - Appends the two `EXTERNALSRC:pn-*` lines to local.conf
  - `cd`'s you into the build dir with `bitbake` on PATH

Re-sourcing on subsequent shells skips the menu and EULA (already
accepted) and just sets up the env.

#### Override the defaults

```bash
# Use a different build dir
BUILD_DIR=my-build source ./setup_dec_stm32.sh

# Override DISTRO or MACHINE (rare for this kit)
DISTRO=openstlinux-weston MACHINE=stm32mp25-dec-kit \
    source ./setup_dec_stm32.sh
```

### 6. Build the image

```bash
bitbake dec-image-full
```

Output lands in `tmp-glibc/deploy/images/stm32mp25-dec-kit/`. The
final `.ext4` rootfs is named like
`dec-image-full-openstlinux-weston-stm32mp25-dec-kit.rootfs.ext4`.

### 7. Flash

#### To SD card

```bash
cd tmp-glibc/deploy/images/stm32mp25-dec-kit/
./scripts/create_sdcard_from_flashlayout.sh \
    flashlayout_dec-image-full/extensible/FlashLayout_sdcard_stm32mp-dec-kit-extensible.tsv

# Identify the SD device (don't paste sdX literally)
lsblk

sudo dd if=FlashLayout_sdcard_stm32mp-dec-kit-extensible.raw \
        of=/dev/sdX bs=8M conv=fsync status=progress
sync
```

#### To eMMC over USB DFU

Requires [STM32CubeProgrammer](https://www.st.com/en/development-tools/stm32cubeprog.html)
installed on the host (registration required for download).

1. Power off the board
2. Set BOOT pins to USB DFU mode (consult the carrier silkscreen / docs)
3. Connect a USB-C cable to the **USB-C(A)** port (the dwc3 port)
4. Power on; verify DFU enumeration: `lsusb | grep 0483:df11`
5. Run:
   ```bash
   STM32_Programmer_CLI -c port=usb1 \
       -w flashlayout_dec-image-full/optee/FlashLayout_emmc_stm32mp-dec-kit-optee.tsv
   ```
6. Power off, set BOOT pins to **eMMC boot**, power on

---

## Subsequent builds (already-set-up tree)

After a `repo sync` to pull updates:

```bash
cd stm-yocto-bsp
source ./setup_dec_stm32.sh   # detects existing build dir, no EULA prompt
bitbake dec-image-full
```

---

## Available Manifests

| Manifest | Yocto Release | OpenSTLinux | Linux Kernel | Supported Boards |
|----------|---------------|-------------|--------------|------------------|
| `stm32mp-6.6.78-r2.xml` | Scarthgap | v25.06.11 (stm32mp-r2) | 6.6.78 | STM32MP25 DEC Kit (00395 SOM on 00365v3 carrier) |

---

## Source repos referenced by this manifest

| Component | Repo | Branch |
|---|---|---|
| meta-dec-common | https://github.com/DiamondTechno/meta-dec-common | dec-scarthgap-6.6 |
| meta-dec-stm32 | https://github.com/DiamondTechno/meta-dec-stm32 | main |
| Linux kernel | https://github.com/DiamondTechno/linux-stm32 | stm32mp-dec-kit |
| U-Boot | https://github.com/DiamondTechno/u-boot-stm32 | stm32mp-dec-kit |
| ST OpenSTLinux layers | https://github.com/STMicroelectronics/{meta-st-openstlinux,meta-st-stm32mp,meta-st-stm32mp-addons,meta-st-scripts} | (pinned SRCREV) |

---

## Troubleshooting

**`envsetup.sh` re-prompts for everything on every shell:** the first
run creates the build dir and writes acceptance state. Subsequent runs
detect the existing build dir and skip the menu. If you're in the
wrong directory, the menu fires again.

**EULA prompt loops forever:** press `q` to exit the `more`/`less`
pager that's displaying the EULA, then answer `y` at the acceptance
prompt.

**`bitbake -e dec-image-full | grep ACCEPT_EULA`** shows nothing:
acceptance wasn't recorded. Open `conf/local.conf` and add
`ACCEPT_EULA_stm32mp25-dec-kit = "1"` manually, then re-run bitbake.

**`Nothing PROVIDES 'libntfs-3g'`** or similar missing-package errors:
make sure `meta-openembedded/meta-filesystems` is in your bblayers.conf.
The setup script should add it; if not, edit `conf/bblayers.conf` and
add the layer path.
