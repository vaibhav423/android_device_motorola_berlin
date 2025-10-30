# v4l2loopback Quick Start Guide

## What You Get

After building and flashing this ROM, your Motorola Berlin device will have:
- `/vendor/lib/modules/v4l2loopback.ko` - The kernel module
- Automatic loading at boot (configured in `modules.load`)
- Virtual V4L2 video devices ready to use

## Quick Commands

### Check if Module is Loaded
```bash
adb shell su -c "lsmod | grep v4l2loopback"
```

### Manual Load (if needed)
```bash
adb shell su -c "insmod /vendor/lib/modules/v4l2loopback.ko devices=2"
```

### Verify Video Devices
```bash
adb shell su -c "ls -l /dev/video*"
```

### Unload Module
```bash
adb shell su -c "rmmod v4l2loopback"
```

## Common Use Cases

### Single Virtual Camera
```bash
adb shell su -c "insmod /vendor/lib/modules/v4l2loopback.ko devices=1 video_nr=10 card_label='Virtual Cam'"
```

### Multiple Virtual Cameras
```bash
adb shell su -c "insmod /vendor/lib/modules/v4l2loopback.ko devices=2 video_nr=10,11"
```

### Exclusive Mode (Recommended for most apps)
```bash
adb shell su -c "insmod /vendor/lib/modules/v4l2loopback.ko exclusive_caps=1"
```

## Module Parameters

| Parameter | Description | Example |
|-----------|-------------|---------|
| `devices=N` | Number of virtual devices | `devices=2` |
| `video_nr=X,Y` | Device numbers | `video_nr=10,11` |
| `card_label="Name"` | Device name | `card_label="My Camera"` |
| `exclusive_caps=1` | Exclusive capture/output mode | `exclusive_caps=1` |
| `max_buffers=N` | Maximum number of buffers | `max_buffers=8` |
| `max_openers=N` | Max simultaneous openers | `max_openers=2` |

## Checking Logs

### Kernel Messages
```bash
adb shell su -c "dmesg | grep v4l2loopback"
```

### Module Info
```bash
adb shell su -c "modinfo /vendor/lib/modules/v4l2loopback.ko"
```

## Need Help?

See the full documentation:
- `README.md` - Detailed usage and troubleshooting
- `../../../BUILD_V4L2LOOPBACK.md` - Complete build instructions
