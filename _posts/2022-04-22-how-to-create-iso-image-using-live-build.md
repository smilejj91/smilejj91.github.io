---
title:  "How to create ISO image using live-build"

categories:
  - OS
tags:
  - live-build
  - iso
  - debian
  - os
# 목차
toc: true
toc_sticky: true
---

# How to create ISO Image Using live-build

## 1. live-build

* ISO 이미지를 제작하기 위해서 사용하고 있는 패키지

## 2. file tree

```bash
config
  │ # stage (bootstrap -> chroot -> installer -> binary -> source)
  ├ bootstrap : base
  ├ chroot : squashfs와 동일하다고 보면 됨
  ├ installer : installer 폴더 구성
  ├ binary : add extra files (installer + bootloader + etc.)
  ├ source
  ├ common
  ├ archives (sources.list 관련)
  │ ├ {name}.list.chroot : live system에 존재 X
  │ ├ {name}.list.binary : live system에 존재 O
  │ └ .deb : local keyring package, live system에 존재 O
  ├ pacakge-lists (iso package list 관련)
  │ ├ .list.chroot_live : only live system
  │ ├ .list.chroot_install : live + installed system
  │ └ .list.binary : only installed system
  ├ hooks (hooking script)
  │ ├ live : live image 
  │ │ ├  .chroot : run after chroot stage
  │ │ └  .binary : run after binary stage
  │ └ normal : regular system image
  │   ├  .chroot : run after chroot stage
  │   └  .binary : run after binary stage
  │ # includes (file copy)
  ├ includes.bootstrap
  ├ includes.chroot_before_packages (before install packages)
  ├ includes.chroot_after_packages (after install packages)
  ├ includes.binary (binary stage)
  └ includes.installer
```

## 3. configuration

### Step 1. make config folder

```bash
$ lb config
```

### Step 2. modify files in config folder

#### bootstrap (Architecture, Distribution, component, repository, etc.)

```bash
...
LB_ARCHITECTURE="amd64"

# Select distribution to use
LB_DISTRIBUTION="bullseye"

# Select archive areas to use
LB_ARCHIVE_AREAS="main contrib non-free"

# Set parent mirror to bootstrap from
LB_PARENT_MIRROR_BOOTSTRAP="http://deb.debian.org/debian"
...
```

##### chroot (Chroot filesystem, keyring, linux-kernel, security/updates/backports repository enable/disable, etc.)

```bash
...
# Set chroot filesystem
LB_CHROOT_FILESYSTEM="squashfs"

# Set keyring packages
LB_KEYRING_PACKAGES="debian-archive-keyring"

# Set kernel packages to use
LB_LINUX_PACKAGES="linux-image linux-headers"

# Enable security updates
LB_SECURITY="true"

# Enable updates updates
LB_UPDATES="true"

# Enable backports updates (optional)
LB_BACKPORTS="false"
...
```

##### binary (image type, filesystem, kernel boot parameter, bootloader, disk label. etc.)

```bash
...
# Set image type
LB_IMAGE_TYPE="iso-hybrid"

# Set image filesystem
LB_BINARY_FILESYSTEM="fat32"

# Set boot parameters
LB_BOOTAPPEND_LIVE="boot=live components quiet splash live-config.locales=ko_KR.UTF-8 live-config.timezone=Asia/Seoul live-config.keyboard-layouts=kr"

# Set BIOS bootloader
LB_BOOTLOADER_BIOS="syslinux"

# Set EFI bootloader
LB_BOOTLOADER_EFI="grub-efi"

# Set hdd label
LB_HDD_LABEL="DEBIAN_LIVE"
...
```

##### common (apt configuration, cache, iso file name, iso type, etc.)

```bash
...
# Set package manager
LB_APT="apt"

# Set apt/aptitude recommends
LB_APT_RECOMMENDS="false"

# Set apt/aptitude security
LB_APT_SECURE="true"

# Control cache
LB_CACHE="true"

# Control if completed stages should be cached
LB_CACHE_STAGES="bootstrap"

# Set debconf(1) frontend to use
LB_DEBCONF_FRONTEND="noninteractive"

# Set init system
LB_INITSYSTEM="systemd"

# Set system type
LB_SYSTEM="live"

# Set base name of the image
LB_IMAGE_NAME="debian_bullsye"

# Set options to use with apt
APT_OPTIONS="--yes -o Acquire::Retries=5 -o Dpkg::Options::=--force-confdef"

# Set options to use with aptitude
APTITUDE_OPTIONS="--assume-yes -o Acquire::Retries=5"
...
```

##### package-lists/test.list.chroot_live (live system package)

```bash
...
calamares-settings-debian
squashfs-tools
rsync
dosfstools
live-tools
...
```

##### package-lists/test.list.chroot (live + installed system package)

```bash
...
xserver-xorg
xserver-xorg-legacy
xinit

gdm3
gnome-session
gnome-session-canberra
gnome-shell
gnome-shell-extensions

task-korean-gnome-desktop
task-english
desktop-base
gnome-terminal
gnome-control-center

sudo 
live-tools
user-setup
eject

firmware-iwlwifi
...
```

##### package-lists/test.list.binary (packages installed by the installer)

```bash
...
grub-efi-amd64
grub-efi-amd64-bin
gnome-online-accounts
gnome-initial-setup
...
```

#### Step 3. create iso image

```bash
$ lb config # apply configuration
$ lb build
```

## 4. link

[Debian Live Build Manual](ttps://live-team.pages.debian.net/live-manual/html/live-manual/index.en.html)
