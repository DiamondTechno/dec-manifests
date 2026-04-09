# dec-manifests

This repository houses the published manifest files supporting the [Diamond Technologies](https://www.diamondtechnologies.com) (DEC) products.

## Quick Start

### Install the `repo` tool

```bash
mkdir -p ~/.bin
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/.bin/repo
chmod a+rx ~/.bin/repo
export PATH="${HOME}/.bin:${PATH}"
```

### Initialize and sync

```bash
mkdir imx-yocto-bsp && cd imx-yocto-bsp
repo init -u https://github.com/DiamondTechno/dec-manifests -b dec-scarthgap-6.6 -m imx-6.6.36-2.1.0.xml
repo sync -c -j$(nproc)
```

### Set up the build environment

```bash
MACHINE=imx93-dec-kit source setup_dec.sh -b bld-wayland
```

### Build an image

```bash
bitbake dec-image-full
```

## Available Manifests

| Manifest | Yocto Release | NXP BSP | Supported Boards |
|----------|--------------|---------|-----------------|
| imx-6.6.36-2.1.0.xml | Scarthgap | 6.6.36_2.1.0 | i.MX93 DEC Kit |
