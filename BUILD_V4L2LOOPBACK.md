# Building v4l2loopback Kernel Module for Motorola Berlin

This document provides comprehensive instructions for building and deploying the v4l2loopback kernel module on the Motorola Berlin (edge 20) device.

## Overview

The v4l2loopback kernel module has been integrated into this device tree and will be built as part of the kernel module compilation process. This module creates virtual V4L2 (Video4Linux2) devices that can be used for video streaming, virtual cameras, and video processing applications.

## Prerequisites

### 1. Build Environment

You need a complete LineageOS/Android build environment. If you don't have one set up, follow the official LineageOS build guide:
https://wiki.lineageos.org/devices/berlin/build/

Required:
- Ubuntu 20.04/22.04 or similar Linux distribution
- At least 200GB of free disk space
- 16GB RAM (32GB recommended)
- Fast internet connection

### 2. LineageOS Source Code

```bash
# Initialize repo
repo init -u https://github.com/LineageOS/android.git -b lineage-21.0 --git-lfs

# Sync the source (this will take a while)
repo sync -c -j$(nproc --all) --force-sync --no-clone-bundle --no-tags
```

### 3. Device-Specific Sources

```bash
# Clone this device tree into the appropriate location
cd device/motorola
git clone https://github.com/vaibhav423/android_device_motorola_berlin berlin

# You'll also need the common device tree and vendor files
# Follow the lineage.dependencies file for the complete list
```

## Building Process

### Option 1: Build Complete ROM (Recommended)

This will build the entire LineageOS ROM including the v4l2loopback kernel module:

```bash
# Navigate to the root of your LineageOS source tree
cd ~/android/lineage

# Set up build environment
source build/envsetup.sh

# Select the device
lunch lineage_berlin-userdebug

# Build everything (this will take several hours)
mka bacon -j$(nproc --all)
```

The v4l2loopback kernel module will be built automatically and included in the ROM at:
- **Build output**: `out/target/product/berlin/vendor/lib/modules/v4l2loopback.ko`
- **Device location** (after flashing): `/vendor/lib/modules/v4l2loopback.ko`

### Option 2: Build Only Kernel Modules

If you only want to rebuild the kernel modules without building the entire ROM:

```bash
# Set up environment
source build/envsetup.sh
lunch lineage_berlin-userdebug

# Build kernel modules only
mka modules

# Or build specific module
mka v4l2loopback.ko
```

### Option 3: Standalone Cross-Compilation

For advanced users who want to build the module outside the Android build system:

```bash
cd device/motorola/berlin/kernel_modules/v4l2loopback

# Set up cross-compilation environment
export ARCH=arm64
export CROSS_COMPILE=aarch64-linux-android-
export KERNEL_DIR=/path/to/kernel/source/for/sm7325

# Build the module
make -C $KERNEL_DIR M=$(pwd) modules

# Output will be v4l2loopback.ko
```

**Note**: For standalone compilation, you need:
- Kernel source tree for sm7325 (yupik) platform
- Proper cross-compilation toolchain
- Kernel headers matching the running kernel on your device

## Verification

After building, verify the module was created:

```bash
# Check if the module exists
ls -lh out/target/product/berlin/vendor/lib/modules/v4l2loopback.ko

# Check module information
file out/target/product/berlin/vendor/lib/modules/v4l2loopback.ko
modinfo out/target/product/berlin/vendor/lib/modules/v4l2loopback.ko
```

Expected output should show:
- File type: ELF 64-bit LSB relocatable, ARM aarch64
- Module information including version, description, author, and license

## Installation

### Method 1: Flash Complete ROM

1. Boot device into recovery mode
2. Flash the built ROM zip file: `out/target/product/berlin/lineage-*.zip`
3. Reboot

The module will be automatically installed to `/vendor/lib/modules/v4l2loopback.ko`

### Method 2: Push Module Only (For Testing)

If you only want to test the module without flashing the entire ROM:

```bash
# Boot device normally (must be rooted)
adb root
adb remount

# Push the module
adb push out/target/product/berlin/vendor/lib/modules/v4l2loopback.ko /vendor/lib/modules/

# Fix permissions
adb shell chmod 644 /vendor/lib/modules/v4l2loopback.ko

# Reboot to load the module
adb reboot
```

## Loading the Module

### Automatic Loading (Default)

The module is configured to load automatically at boot via the `modules.load` configuration file. After flashing the ROM or rebooting, verify it's loaded:

```bash
adb shell
su
lsmod | grep v4l2loopback
```

### Manual Loading

If you need to load the module manually:

```bash
adb shell
su
insmod /vendor/lib/modules/v4l2loopback.ko devices=2
```

Common parameters:
- `devices=N` - Number of virtual devices to create (default: 2)
- `video_nr=X,Y` - Specific video device numbers (e.g., 5,6 for /dev/video5 and /dev/video6)
- `card_label="Name1","Name2"` - Labels for the virtual devices
- `exclusive_caps=1` - Create devices that only support exclusive capture or output

Example with parameters:
```bash
insmod /vendor/lib/modules/v4l2loopback.ko devices=1 video_nr=10 card_label="Virtual Camera"
```

## Usage

Once loaded, verify virtual video devices are created:

```bash
adb shell
su
ls -l /dev/video*
```

You should see the virtual video devices (e.g., `/dev/video0`, `/dev/video1`).

### Testing the Virtual Camera

You can test the module using V4L2 tools or applications:

```bash
# List video devices and their capabilities
v4l2-ctl --list-devices

# Check device info
v4l2-ctl -d /dev/video0 --all
```

## Troubleshooting

### Module fails to build

1. **Missing kernel headers**: Ensure kernel is built first
   ```bash
   mka kernel
   ```

2. **Compiler version mismatch**: Use the Android build system's compiler
   ```bash
   # Don't use external compilers; use the build system
   source build/envsetup.sh
   lunch lineage_berlin-userdebug
   ```

3. **Clean build**: Try cleaning and rebuilding
   ```bash
   make clean
   mka modules
   ```

### Module fails to load

1. **Kernel version mismatch**: Module must match running kernel version
   ```bash
   # Check running kernel version
   adb shell uname -r
   
   # Check module version
   modinfo out/target/product/berlin/vendor/lib/modules/v4l2loopback.ko | grep vermagic
   ```

2. **Missing dependencies**: Check kernel config
   ```bash
   # Required kernel configs:
   CONFIG_MODULES=y
   CONFIG_VIDEO_DEV=y
   CONFIG_VIDEO_V4L2=y
   ```

3. **SELinux denials**: Check logcat and dmesg
   ```bash
   adb logcat | grep avc
   adb shell dmesg | grep v4l2loopback
   ```

### No video devices created

The module might load but not create devices. Check:

```bash
# Check if module is loaded
adb shell lsmod | grep v4l2loopback

# Check kernel logs
adb shell dmesg | grep -i v4l2

# Try loading with explicit parameters
adb shell insmod /vendor/lib/modules/v4l2loopback.ko devices=1 video_nr=10
```

## Advanced Configuration

### Persistent Module Parameters

To set default parameters for automatic loading, you can create a modprobe configuration:

```bash
# Create config file on device
adb shell
su
echo "options v4l2loopback devices=2 video_nr=10,11 exclusive_caps=1" > /vendor/etc/modprobe.d/v4l2loopback.conf
```

### SELinux Policy

If you encounter SELinux denials, you may need to add custom policies. Check denials:

```bash
adb shell
su
dmesg | grep avc | grep v4l2loopback
```

## Technical Details

- **Module Name**: v4l2loopback.ko
- **Source**: Based on https://github.com/umlaeute/v4l2loopback
- **License**: GPL-2.0
- **Target Architecture**: ARM64 (aarch64)
- **Kernel Platform**: Qualcomm SM7325 (Yupik)
- **Device**: Motorola Berlin (edge 20)

## File Locations

| Description | Path |
|-------------|------|
| Source code | `device/motorola/berlin/kernel_modules/v4l2loopback/` |
| Build configuration | `device/motorola/berlin/kernel_modules/v4l2loopback/Android.bp` |
| Module loading config | `device/motorola/berlin/modules.load` |
| Built module | `out/target/product/berlin/vendor/lib/modules/v4l2loopback.ko` |
| Device location | `/vendor/lib/modules/v4l2loopback.ko` |

## Additional Resources

- v4l2loopback upstream: https://github.com/umlaeute/v4l2loopback
- LineageOS Build Guide: https://wiki.lineageos.org/devices/berlin/build/
- V4L2 Documentation: https://www.kernel.org/doc/html/latest/userspace-api/media/v4l/v4l2.html
- Module README: `device/motorola/berlin/kernel_modules/v4l2loopback/README.md`

## Support

For issues specific to:
- **Building**: Check LineageOS build documentation
- **v4l2loopback functionality**: Refer to upstream project documentation
- **Device-specific issues**: Open an issue in this repository

## License

The v4l2loopback kernel module is licensed under GPL-2.0. See `kernel_modules/v4l2loopback/LICENSE` for details.
