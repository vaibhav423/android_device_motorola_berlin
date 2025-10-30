# v4l2loopback Kernel Module for Motorola Berlin

This directory contains the v4l2loopback kernel module for the Motorola Berlin (edge 20) device.

## What is v4l2loopback?

v4l2loopback is a kernel module that creates virtual video devices (V4L2 loopback devices). These devices appear as normal video devices to applications, but instead of capturing from a physical camera, they allow other applications to generate or stream video content.

## Building the Module

The v4l2loopback kernel module is integrated into the LineageOS/Android build system for the Motorola Berlin device.

### Prerequisites

1. A complete LineageOS/Android build environment set up for building kernel modules
2. Kernel headers for the sm7325 (yupik) platform
3. Cross-compilation toolchain (usually provided by the Android build environment)

### Build Instructions

The module is built automatically when building the entire device image:

```bash
# From the root of your LineageOS source tree
source build/envsetup.sh
lunch lineage_berlin-userdebug
make -j$(nproc --all)
```

The built kernel module will be located at:
```
out/target/product/berlin/vendor/lib/modules/v4l2loopback.ko
```

### Building Only the Kernel Module

To build only the v4l2loopback kernel module:

```bash
# From the root of your LineageOS source tree
source build/envsetup.sh
lunch lineage_berlin-userdebug
make v4l2loopback.ko
```

## Installation on Device

The module is automatically installed to the device when flashing the ROM. It will be located at:
```
/vendor/lib/modules/v4l2loopback.ko
```

The module is configured to load automatically at boot via the `modules.load` configuration.

## Manual Loading/Unloading

If you need to manually load or unload the module on a rooted device:

### Loading the Module

```bash
adb shell
su
insmod /vendor/lib/modules/v4l2loopback.ko
```

Optional parameters:
```bash
# Create 2 video devices
insmod /vendor/lib/modules/v4l2loopback.ko devices=2

# Create devices with specific video numbers
insmod /vendor/lib/modules/v4l2loopback.ko video_nr=5,6

# Set device labels
insmod /vendor/lib/modules/v4l2loopback.ko card_label="Virtual Camera 1","Virtual Camera 2"
```

### Unloading the Module

```bash
adb shell
su
rmmod v4l2loopback
```

### Checking Module Status

```bash
adb shell
su
lsmod | grep v4l2loopback
```

## Using the Virtual Camera

Once loaded, the module creates virtual video devices (e.g., `/dev/video0`, `/dev/video1`, etc.).

You can verify the devices are created:
```bash
adb shell
su
ls -l /dev/video*
```

To write to a virtual camera device, you'll need an application that can output V4L2 video. Some examples:

- GStreamer pipelines
- FFmpeg
- Custom applications using V4L2 API

Example using GStreamer to write a test pattern:
```bash
gst-launch-1.0 videotestsrc ! v4l2sink device=/dev/video0
```

## Troubleshooting

### Module fails to load

1. Check kernel logs:
   ```bash
   adb shell dmesg | grep v4l2loopback
   ```

2. Verify module exists:
   ```bash
   adb shell ls -l /vendor/lib/modules/v4l2loopback.ko
   ```

3. Check kernel version compatibility - the module must be compiled against the same kernel version running on the device

### No video devices created

Check module parameters and kernel logs. The module might need explicit device creation parameters.

## Source Code

The v4l2loopback module is based on the upstream project:
- GitHub: https://github.com/umlaeute/v4l2loopback
- License: GPL-2.0

## Notes

- This module requires root access to load/unload
- The device must have kernel module loading support enabled (CONFIG_MODULES=y)
- SELinux policies may need to be adjusted for applications to access the virtual devices
- The module is compiled for the specific kernel version of the Motorola Berlin device
