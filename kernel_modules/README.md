# Kernel Modules for Motorola Berlin

This directory contains custom kernel modules for the Motorola Berlin (edge 20) device.

## Available Modules

### v4l2loopback
Creates virtual V4L2 (Video4Linux2) loopback devices for video streaming and virtual camera applications.

**Documentation:**
- [v4l2loopback/README.md](v4l2loopback/README.md) - Detailed usage instructions
- [v4l2loopback/QUICK_START.md](v4l2loopback/QUICK_START.md) - Quick reference guide
- [../BUILD_V4L2LOOPBACK.md](../BUILD_V4L2LOOPBACK.md) - Complete build instructions

**Features:**
- Virtual camera devices for video processing
- Multiple simultaneous virtual devices
- Compatible with standard V4L2 applications
- Configurable device parameters
- Automatic loading at boot

**Status:** ✅ Integrated and ready to build

## Building Kernel Modules

All kernel modules in this directory are automatically built as part of the device build process.

```bash
# Build entire device (includes all kernel modules)
source build/envsetup.sh
lunch lineage_berlin-userdebug
mka bacon

# Build only kernel modules
mka modules
```

## Adding New Kernel Modules

To add a new kernel module to this device tree:

1. Create a new directory under `kernel_modules/`
2. Add source files (`.c`, `.h`)
3. Create build files:
   - `Android.bp` - For Android build system integration
   - `Kbuild` - For kernel module build
   - `Makefile` - Optional, for standalone builds
4. Add module to `device.mk` in `PRODUCT_PACKAGES`
5. Add module to `modules.load` for automatic loading (optional)
6. Document the module in a README

### Example Android.bp for Kernel Module

```blueprint
kernel_module {
    name: "mymodule.ko",
    vendor: true,
    srcs: [
        "mymodule.c",
    ],
    kernel_build: "kernel",
}
```

### Example Kbuild

```makefile
obj-m := mymodule.o
```

## Module Loading

Modules can be loaded in two ways:

### 1. Automatic (Recommended)
Add the module filename to `modules.load` in the device root:
```
mymodule.ko
```

### 2. Manual
Load using `insmod` after boot:
```bash
adb shell su -c "insmod /vendor/lib/modules/mymodule.ko"
```

## Installed Location

All kernel modules are installed to:
```
/vendor/lib/modules/
```

## Requirements

- Root access required for loading/unloading modules
- Kernel must have `CONFIG_MODULES=y` enabled
- Module must match kernel version exactly
- SELinux may require policy adjustments

## License

Each kernel module has its own license. Check the LICENSE or source files in each module's directory.
