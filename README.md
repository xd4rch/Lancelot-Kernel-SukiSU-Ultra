# SukiSU Ultra + SusFS Kernel for Redmi 9 (lancelot)

A custom, stealth-oriented Android 4.14 kernel built specifically for **Redmi 9 / 9 Prime (`lancelot`)** running LineageOS 20 (Android 13).

This kernel implements **SukiSU Ultra** via direct non-GKI manual system-call hooking, combined with full **SusFS** kernel-level isolation, **OverlayFS** module support, and **KPM (Kernel Patch Module)** binary injection.

---

## Key Features

* **SukiSU Ultra (v4.2.0+ Compliant):** Full root elevation (`uid=0`, `u:r:su:s0`) via non-GKI manual syscall hooks (`execveat`, `stat`, `faccessat`).
* **Selective App Root Isolation:** Physical `/system/bin/su` is never written to disk. Unprivileged or root-detecting apps receive `-ENOENT` (`No such file or directory`) instead of permission denial.
* **SusFS (Suspicious File System):** Deep kernel-level hiding for root paths, bind mounts, and kstat tampering to defeat advanced banking, DRM, and enterprise integrity checks.
* **KPM (Kernel Patch Module):** Injected runtime loader interface support for dynamic kernel module extensions.
* **OverlayFS Enabled:** Native support for Magic Mount and modern systemless root modules.
* **Built with Proton Clang:** Compiled using modern toolchains with targeted optimizations for the MediaTek MT6768 (Helio G80) platform.
* **Automated CI/CD Builds:** Pure GitHub Actions workflow generating flashable AnyKernel3 deployment packages.

---

## Supported Devices & ROMs

| Attribute | Specification |
| --- | --- |
| **Target Device** | Xiaomi Redmi 9 / Redmi 9 Prime (`lancelot`) |
| **SoC** | MediaTek MT6768 (Helio G80) |
| **Base Kernel** | Linux 4.14.320-perf (Android Common Kernel fork) |
| **Target OS** | LineageOS 20 (Android 13 / AOSP 13) |
| **Root Solution** | SukiSU Ultra Manager (`com.sukisu.ultra`) |

---

## Installation Guide

### Prerequisites

* **Unlocked Bootloader**: The device must have an unlocked bootloader.
* **Custom Recovery**: TWRP, OrangeFox, or PBRP installed.
* **Backup**: Always take a full NANDroid/Boot partition backup before flashing.
* **Manager Application**: Install the latest [SukiSU Ultra Manager APK](https://github.com/SukiSU-Ultra/SukiSU-Ultra/releases) on your device.

---

### Method 1: Flash via Custom Recovery (Recommended)

1. Download the latest `SukiSU-Ultra-Lancelot-LOS20-<BUILD_TIME>.zip` from the **Actions** artifacts tab or **Releases** page.
2. Transfer the ZIP package to your device's internal storage or an SD card.
3. Reboot your device into recovery mode (**Power + Volume Up**).
4. Tap **Install**, locate the ZIP file, and swipe to flash.
* AnyKernel3 will automatically unpack the boot image, patch the kernel binary directly into `mmcblk0p34` (`/boot`), and preserve your existing ramdisk.


5. Once completed, tap **Reboot System**.

---

### Method 2: Flash via Fastboot (Manual Unpack)

If you do not have custom recovery access:

1. Extract `Image.gz` from the AnyKernel3 flashable package.
2. Unpack your current working ROM's `boot.img` using tools like `magiskboot` or Android Image Kitchen.
3. Replace the stock kernel binary with your newly extracted `Image.gz`.
4. Repack the boot image (`new-boot.img`).
5. Boot into Fastboot mode (**Power + Volume Down**) and flash:
```bash
fastboot flash boot new-boot.img
fastboot reboot

```



---

## Post-Install Verification

Open a terminal (or ADB shell) and verify that all kernel hooks and managers are responding correctly:

```bash
# Verify kernel release and compilation string
uname -r

# Verify that unprivileged shell can elevate directly through sucompat
adb shell "su -c id"
# Expected response:
# uid=0(root) gid=0(root) ... context=u:r:su:s0

# Check SukiSU feature support table
adb shell "su -c '/data/adb/ksu/bin/ksud feature list'"

# Check hook engine status
adb shell "su -c '/data/adb/ksu/bin/ksud debug info'"
# Expected hook flags: flags: 0x2 (Manual Hook Mode)

```

Launch the **SukiSU Ultra** app. The primary status card will indicate `Working (Manual)` with full UAPI binding.

---

## Build from Source (GitHub Actions)

This repository is designed to build headlessly via GitHub Actions without requiring local cross-compilation environments.

### Step-by-Step Instructions

1. **Fork this repository** to your personal GitHub account.
2. In your forked repository, navigate to the **Actions** tab.
3. Enable workflows if prompted by GitHub.
4. Select the **Build Kernel** workflow in the left sidebar.
5. Click the **Run workflow** dropdown button:
* Select the target branch (`main`).
* Choose the device target (`lancelot`).


6. Click **Run workflow** to initiate compilation.
7. Once finished (typically ~10–15 minutes), click on the completed run.
8. Download the flashable ZIP artifact from the **Artifacts** section at the bottom of the page.

---

## Configuration Reference (`config-suki.env`)

All core workflow variables are centralized in `config-suki.env`:

| Variable | Description |
| --- | --- |
| `KERNEL_SOURCE` | Git URL of the upstream Android kernel source tree. |
| `KERNEL_SOURCE_BRANCH` | Target kernel source branch. |
| `CLANG_SOURCE` | Direct archive download URL for the Proton Clang toolchain. |
| `KERNELSU_SETUP_SOURCE` | Installation script URL for the SukiSU/KernelSU tree. |
| `KERNELSU_TAG` | SukiSU release tag or commit hash targeted during source preparation. |
| `KPM_SUPPORT` | Set to `true` to inject runtime KPM patching via `patch_linux`. |
| `ANYKERNEL3_SOURCE` | Upstream AnyKernel3 repository used to bundle the final installer. |

---

## Credits & Acknowledgments

* **LineageOS**: Android open-source base & kernel tree foundations.
* **SukiSU Ultra Team**: Advanced KernelSU distribution and manager implementation.
* **KernelSU (tiann)**: Revolutionary kernel-level root philosophy and core foundation.
* **KernelSU-Next**: Non-GKI reference implementations and architectural insights.
* **SIMFS / SusFS (sidex1024)**: Kernel-space stealth mount and integrity protection engine.
* **osm0sis**: [AnyKernel3](https://github.com/osm0sis/AnyKernel3) systemless boot partition deployment tool.
* **kdrag0n**: [Proton Clang](https://github.com/kdrag0n/proton-clang) LLVM toolchain.
