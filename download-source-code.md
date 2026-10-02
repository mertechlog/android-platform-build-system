# Aosp source download and build setup

This document covers the complete sequence for downloading AOSP source code, configuring the development environment, selecting a platform/product target, and building Android.

---

## prerequisites

### host system

Recommended host environment:

* 64-bit Linux
* Ubuntu 20.04 or newer supported Ubuntu LTS
* 64 GB RAM or more
* 400 GB or more free disk space
* Stable and high-speed internet connection

Check the system:

```bash
# CPU cores
nproc --all

# RAM
free -h

# Available disk space
df -h
```

Official documentation:

* https://source.android.com/docs/setup/start

---

## internet connection

AOSP consists of a large number of Git repositories. The `repo` tool is used to manage and download these repositories.

The main source server is:

```text
https://android.googlesource.com/
```

A reliable internet connection is strongly recommended because the initial source download is large.
```bash
# Quick check — Internet connectivity
ping -c 4 8.8.8.8
ping -c 4 google.com
```
---

## install required packages

Install the required Ubuntu packages:

```bash
sudo apt-get update

sudo apt-get install \
    git-core \
    gnupg \
    flex \
    bison \
    build-essential \
    zip \
    curl \
    zlib1g-dev \
    libc6-dev-i386 \
    x11proto-core-dev \
    libx11-dev \
    lib32z1-dev \
    libgl1-mesa-dev \
    libxml2-utils \
    xsltproc \
    unzip \
    fontconfig
```

Official documentation:

https://source.android.com/docs/setup/start

---

## configure git

AOSP uses Git repositories, so configure your Git identity before working with the source tree.

```bash
git config --global user.name "Your Name"
git config --global user.email "your-email@example.com"
```

Verify:

```bash
git config --global --list
```
---

## install repo

`repo` is Google's repository-management tool. AOSP consists of many Git repositories, and `repo` coordinates them as a single source tree.

Install Repo:

```bash
sudo apt-get update
sudo apt-get install repo
```

Verify:

```bash
repo version
```

Official documentation:

https://source.android.com/docs/setup/reference/repo

---

##  create aosp workspace

Create a directory for the AOSP source tree:

```bash
mkdir ~/aosp
cd ~/aosp
```

The complete source tree will be downloaded inside this directory.

Example:

```text
~/aosp/
```

---

# aosp source branches and releases

Android source code is organized around Android platform releases and release branches.

For a current development checkout, use:

```text
android-latest-release
```

`android-latest-release` tracks the latest AOSP release branch.

At the time of writing, it points to:

```text
android17-release
```

### release branch examples

| Branch                    | Purpose                    |
| ------------------------- | -------------------------- |
| `android-latest-release`  | Latest AOSP release branch |
| `android17-release`       | Android 17 release branch  |
| `android16-release`       | Android 16 release branch  |
| `android15-release`       | Android 15 release branch  |
| `android14-release`       | Android 14 release branch  |
| older `androidXX-release` | Historical Android release |

The exact available branches and tags should always be checked in the AOSP manifest repository.

Official documentation:

https://source.android.com/docs/setup/reference/build-numbers

AOSP manifest:

https://android.googlesource.com/platform/manifest

---

# mobile, tablet, watch and automotive

Android platform source is shared across different products and form factors.

The relationship is approximately:

```text
                    Android platform
                           |
        +------------------+------------------+
        |                  |                  |
      Phone             Automotive         Other
        |                  |               form factors
   Cuttlefish             AAOS
        |                  |
   Framework             Car Framework
   Graphics              CarService
   Audio                 VHAL
   WMS                   CarAudio
   SurfaceFlinger
```

The selected product/build target determines what configuration and components are built.

---

# repo init

Initialize the AOSP source tree.

Recommended current-release initialization:

```bash
repo init \
    --partial-clone \
    --no-use-superproject \
    -b android-latest-release \
    -u https://android.googlesource.com/platform/manifest
```

The important parameters are:

```text
-u
    Manifest repository

-b
    Branch to checkout

--partial-clone
    Reduce the amount of Git history/data required

--no-use-superproject
    Do not use the Repo superproject
```

Verify Repo:

```bash
repo version
```

Inspect the manifest:

```bash
repo manifest
```

Official documentation:

https://source.android.com/docs/setup/download

---

# repo sync

After `repo init`, download the source repositories:

```bash
repo sync -c -j8
or
repo sync -c -j$(nproc --all)
```

### meaning of the options

```text
-c
    Sync only the current branch

-j8
    Use 8 parallel synchronization jobs
```
Then increase the value if the system has sufficient resources.

Official documentation:

https://source.android.com/docs/setup/reference/repo

---

# verify source tree

After synchronization completes:

```bash
repo status
```

You can also check the number of projects:

```bash
repo list
```

Example source tree:

```text
aosp/
├── art/
├── bionic/
├── bootable/
├── build/
├── device/
├── external/
├── frameworks/
├── hardware/
├── packages/
├── system/
├── tools/
├── vendor/
└── ...
```

---

# initialize android build environment

Before selecting a build target, initialize the Android build environment:

```bash
source build/envsetup.sh
```

This makes Android build commands available in the current shell.

For example:

```bash
hmm
```

can be used to display available Android build commands.

---

# lunch

`lunch` is used to select the Android product and build configuration.

The general target format is:

```text
<product>-<release_config>-<build_variant>
```

The exact target names depend on the AOSP branch that has been checked out.

Run:

```bash
lunch
```

to display the available targets for the current source tree.

```bash
lunch aosp_cf_x86_64-userdebug
```
The exact target names depend on the checked-out AOSP branch.
---

# build variants

Android normally provides three major build variants:

| Variant     | Purpose                          |
| ----------- | -------------------------------- |
| `user`      | Production-oriented build        |
| `userdebug` | Debuggable build for development |
| `eng`       | Engineering/development build    |

### user

Used for production-style builds.

Characteristics:

```text
- Limited debugging
- Production-oriented configuration
- Suitable for release-style testing
```

### userdebug

Used extensively for platform development and debugging.

Characteristics:

```text
- Debuggable
- Root/adb debugging capabilities
- Development-friendly
- Suitable for AOSP platform development
```

For your AOSP platform learning, `userdebug` is generally the useful variant.

### eng

Engineering build with additional debugging/development capabilities.

### Example: AOSP Build

Example build setup used for Android 15 AAOS:

```bash
repo init --partial-clone --no-use-superproject \
    -b android-15.0.0_r16 \
    -u https://android.googlesource.com/platform/manifest

repo sync -c -j8

source build/envsetup.sh

lunch sdk_car_x86_64-trunk_staging-userdebug

m -j$(nproc)
```

**Build target:** `sdk_car_x86_64-trunk_staging-userdebug`

This is an **Android Automotive OS (AAOS) x86_64 userdebug target**, primarily intended for emulator/virtual development and AOSP platform development.


Useful when deep platform development requires maximum debugging support.

---

