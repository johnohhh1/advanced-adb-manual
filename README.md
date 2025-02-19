# Advanced ADB Manual: A Comprehensive Guide for Developers and Security Researchers

## Table of Contents
1. [Introduction](#introduction)
2. [Setup and Prerequisites](#setup-and-prerequisites)
3. [Core ADB Commands](#core-adb-commands)
4. [Advanced Shell Operations](#advanced-shell-operations)
5. [System Modifications](#system-modifications)
6. [Security and Permissions](#security-and-permissions)
7. [APK Analysis and Modification](#apk-analysis-and-modification)
8. [Automation and Scripting](#automation-and-scripting)
9. [Device Recovery and Troubleshooting](#device-recovery-and-troubleshooting)
10. [Best Practices and Common Pitfalls](#best-practices-and-common-pitfalls)

## Introduction

Android Debug Bridge (ADB) is a versatile command-line tool that serves as a bridge between your development machine and Android devices. This manual covers advanced usage scenarios for developers, security researchers, and power users who need to perform complex device operations, system modifications, and security analysis.

### Important Safety Notice
Before proceeding, understand that many commands in this guide:
- Require root access
- Can permanently modify system files
- May void device warranties
- Could potentially render devices unusable if performed incorrectly

Always maintain backups and verify commands before execution.

## Setup and Prerequisites

### Required Tools
```bash
# Install Platform Tools (contains ADB)
# For Linux:
sudo apt-get install android-tools-adb android-tools-fastboot

# For macOS:
brew install android-platform-tools

# For Windows:
# Download from: https://developer.android.com/studio/releases/platform-tools
```

### Initial Device Setup
1. Enable Developer Options:
   - Go to Settings → About Phone
   - Tap "Build Number" seven times
   - Enter device PIN/pattern if prompted

2. Enable USB Debugging:
   - Go to Settings → Developer Options
   - Enable "USB Debugging"
   - Optional: Enable "OEM Unlocking" for bootloader access

### Verifying Connection
```bash
# List connected devices
adb devices

# Expected output:
# List of devices attached
# XXXXXXXX    device    # (where XXXXXXXX is your device ID)
```

## Core ADB Commands

### Device Management
```bash
# Start ADB server
adb start-server

# Kill ADB server
adb kill-server

# Reboot device
adb reboot                  # Normal reboot
adb reboot bootloader       # Reboot to bootloader
adb reboot recovery         # Reboot to recovery
```

### File Operations
```bash
# Push file to device
adb push local_file.txt /sdcard/
# Pull file from device
adb pull /sdcard/remote_file.txt ./

# Execute shell command directly
adb shell "ls -l /sdcard/"

# Interactive shell
adb shell
```

### Package Management
```bash
# Install APK
adb install -r app.apk          # -r: replace existing app
adb install -d app.apk          # -d: allow downgrade
adb install -g app.apk          # -g: grant all permissions

# Uninstall package
adb uninstall com.example.app
adb uninstall -k com.example.app  # -k: keep data/cache
```

## Advanced Shell Operations

### Root Access and System Modifications
```bash
# Enter root shell (requires rooted device)
adb shell
su

# Remount system partition as read-write
mount -o rw,remount /system

# Modify system files (example: build.prop)
cp /system/build.prop /sdcard/build.prop.backup
sed -i 's/ro.sf.lcd_density=480/ro.sf.lcd_density=320/' /system/build.prop
```

### Process Management
```bash
# List all processes
ps -A

# Find specific process
ps | grep com.example.app

# Kill process by PID
kill -9 <PID>

# Monitor process in real-time
top
```

## Security and Permissions

### SELinux Management
```bash
# Check SELinux status
getenforce

# Temporarily set to permissive (requires root)
setenforce 0

# Restore enforcing mode
setenforce 1

# View SELinux context
ls -Z /system/bin/
```

### File Permissions
```bash
# Change file ownership
chown root:root /path/to/file

# Modify file permissions
chmod 644 /path/to/file

# Restore SELinux context
restorecon -R /path/to/file
```

## APK Analysis and Modification

### Decompiling APKs
```bash
# Using apktool
apktool d app.apk -o app_decoded

# Modify files in app_decoded/

# Rebuild APK
apktool b app_decoded -o app_modified.apk
```

### Signing APKs
```bash
# Generate keystore
keytool -genkey -v -keystore debug.keystore -alias androiddebugkey \
  -storepass android -keypass android -keyalg RSA -keysize 2048 \
  -validity 10000

# Sign APK
jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 \
  -keystore debug.keystore app_modified.apk androiddebugkey

# Optimize APK
zipalign -v 4 app_modified.apk app_final.apk
```

## Automation and Scripting

### Basic Automation Script
```bash
#!/bin/bash

# Check device connection
if ! adb devices | grep -q "device$"; then
    echo "No device connected"
    exit 1
fi

# Backup important directories
backup_dirs() {
    echo "Backing up important directories..."
    adb pull /sdcard/DCIM ./backup/
    adb pull /sdcard/Download ./backup/
}

# Clear app data
clear_app_data() {
    local package_name=$1
    echo "Clearing data for $package_name"
    adb shell pm clear $package_name
}

# Main execution
backup_dirs
clear_app_data "com.example.app"
```

## Device Recovery and Troubleshooting

### Common Issues and Solutions

#### Permission Denied
```bash
# Problem: Permission denied when modifying system files
# Solution:

# 1. Check if root access works
adb shell
su
# If this fails, device needs to be rooted

# 2. Remount system with proper permissions
mount -o rw,remount /system

# 3. If still failing, check SELinux
getenforce
# If "Enforcing", temporarily set to permissive
setenforce 0
```

#### Device Not Detected
```bash
# 1. Restart ADB server
adb kill-server
adb start-server

# 2. Check USB debugging is enabled
# 3. Try different USB ports/cables
# 4. Verify drivers (Windows only)
```

### Emergency Recovery
```bash
# Boot to fastboot
adb reboot bootloader

# Flash recovery image
fastboot flash recovery recovery.img

# Flash system image (last resort)
fastboot flash system system.img

# Reboot device
fastboot reboot
```

## Best Practices and Common Pitfalls

### Safety Measures
1. Always backup before system modifications
2. Use systemless modifications when possible
3. Maintain known-good recovery images
4. Document all system changes

### Performance Optimization
```bash
# Monitor system performance
adb shell dumpsys cpuinfo
adb shell dumpsys meminfo
adb shell "top -n 1"

# Clear app cache
adb shell pm clear com.example.app
```

### Network Analysis
```bash
# Port forwarding for local debugging
adb forward tcp:8080 tcp:8080

# Capture network traffic
adb shell "tcpdump -n -w /sdcard/capture.pcap"
```

## Tablet Recovery and Rooting Guide

### Pre-Recovery Preparation
```bash
# Backup all data first
adb backup -all -f backup.ab

# Check current device status
adb shell getprop ro.bootloader.locked
adb shell getprop ro.boot.flash.locked
adb shell getprop ro.boot.verifiedbootstate

# Check current tablet info
adb shell getprop ro.product.model
adb shell getprop ro.build.version.release
adb shell getprop ro.build.version.sdk
```

### Bootloader and Recovery Operations
```bash
# Boot into bootloader
adb reboot bootloader

# Check if device is detected in fastboot
fastboot devices

# Check bootloader status
fastboot getvar all
fastboot oem device-info

# Unlock bootloader (if supported)
fastboot oem unlock
# or on newer devices
fastboot flashing unlock

# Flash custom recovery (e.g., TWRP)
fastboot flash recovery twrp.img
fastboot boot twrp.img  # Test without permanent flash

# Boot directly to recovery
adb reboot recovery
```

### Root Process Commands
```bash
# Push root package to device
adb push magisk.zip /sdcard/

# In TWRP recovery:
adb shell
# Navigate to correct directory
cd /sdcard/
# Install Magisk or other root solution
twrp install magisk.zip

# Verify root after reboot
adb shell
su
whoami  # Should return 'root'
```

### Recovery Troubleshooting
```bash
# If tablet is stuck in bootloop
adb wait-for-device
adb shell getprop sys.boot_completed  # Check if system booted

# Force stop problematic apps
adb shell pm list packages
adb shell am force-stop package.name

# Clear cache if system is unstable
adb shell pm clear package.name
adb shell rm -rf /data/dalvik-cache/*

# Factory reset from fastboot (last resort)
fastboot erase userdata
fastboot erase cache
fastboot reboot
```

### Advanced Recovery Operations
```bash
# Dump full partition list
adb shell cat /proc/partitions

# Backup specific partitions
adb shell dd if=/dev/block/bootdevice/by-name/boot of=/sdcard/boot.img
adb shell dd if=/dev/block/bootdevice/by-name/recovery of=/sdcard/recovery.img

# Flash specific partitions
fastboot flash boot boot.img
fastboot flash recovery recovery.img

# Reset permissions after recovery
adb shell
su
chmod 644 /system/etc/hosts
chown root:root /system/etc/hosts
restorecon /system/etc/hosts
```

### Safety Checks and Verification
```bash
# Verify system integrity
adb shell dm verity status

# Check encryption status
adb shell getprop ro.crypto.state

# Verify SafetyNet status (after rooting)
adb shell am start -n com.google.android.gms/.security.settings.SafetyNetActivity

# Check for common root detection
adb shell which su
adb shell ls -l /system/xbin/su
adb shell ls -l /system/bin/su
```

### Malicious App Investigation and System Analysis

```bash
# List all installed packages including system apps
adb shell pm list packages -f -u

# Get detailed package information
adb shell dumpsys package suspect.package.name

# List all permissions granted to an app
adb shell dumpsys package suspect.package.name | grep permission

# Check running services
adb shell dumpsys activity services | grep -i suspect.package.name

# Monitor app activities in real-time
adb shell dumpsys activity activities | grep -i suspect.package.name

# Check for overlay windows
adb shell dumpsys window windows | grep -E 'Window #|Package:'

# Find hidden processes
adb shell ps -A | grep suspect.package.name
adb shell ps -ef | grep suspect.package.name

# Track app's CPU usage
adb shell top -H | grep suspect.package.name

# Monitor network connections
adb shell netstat -tupn
adb shell cat /proc/net/tcp
adb shell cat /proc/net/udp

# Check for suspicious broadcast receivers
adb shell dumpsys package suspect.package.name | grep -A5 "Receiver Resolver Table"

# Investigate app's data directory
adb shell ls -la /data/data/suspect.package.name/
```

### Advanced App Control and Analysis

```bash
# Force stop an app that keeps restarting
adb shell am force-stop suspect.package.name

# Disable app components
adb shell pm disable-user --user 0 suspect.package.name
adb shell pm disable-user --user 0 'suspect.package.name/com.example.ServiceName'

# Check for apps with SYSTEM_ALERT_WINDOW permission
adb shell dumpsys window windows | grep -E 'Window #|SYSTEM_ALERT_WINDOW'

# List all services running as system
adb shell ps -A | grep system

# Monitor activity stack changes
adb shell dumpsys activity activities | grep -B 2 -A 2 "Hist"

# Check for apps with admin privileges
adb shell dumpsys device_policy

# Investigate app launch behavior
adb shell dumpsys activity broadcasts | grep suspect.package.name

# Monitor input events
adb shell getevent

# Check app's storage usage
adb shell du -h /data/data/suspect.package.name

# Export app's shared preferences
adb shell cp -R /data/data/suspect.package.name/shared_prefs /sdcard/backup/
adb pull /sdcard/backup/

# Analyze app's native libraries
adb shell ls -R /data/app/*/suspect.package.name*/lib/
```

### Deep System Analysis

```bash
# Monitor system events
adb shell logcat -b events

# Check for modified system files
adb shell find /system -type f -mtime -7

# Examine startup scripts
adb shell ls -l /system/etc/init.d/
adb shell cat /system/build.prop | grep boot

# Look for suspicious mounted filesystems
adb shell mount | grep -i "nosuid\|noexec"

# Check SELinux violations
adb shell dmesg | grep "avc: denied"

# Monitor file system changes
adb shell inotifywait -m -r /data/data/suspect.package.name/

# Analyze system properties
adb shell getprop | grep -i "debug\|secure"

# Check for modified boot image
adb shell dd if=/dev/block/bootdevice/by-name/boot of=/sdcard/boot.img
adb pull /sdcard/boot.img
# Then analyze boot.img offline

# Monitor process creation
adb shell strace -f -e trace=process -p $(pidof suspect.package.name)
```

### Advanced Removal Techniques

```bash
# Remove app without package manager
adb shell pm uninstall -k --user 0 suspect.package.name

# Force remove app data
adb shell rm -rf /data/data/suspect.package.name
adb shell rm -rf /data/user/0/suspect.package.name

# Remove app from system partition (requires root)
adb shell
su
mount -o rw,remount /system
rm -rf /system/app/SuspectApp/
rm -rf /system/priv-app/SuspectApp/

# Clean up remaining traces
adb shell pm clear suspect.package.name
adb shell am stopservice -n suspect.package.name/.services.SuspectService

# Remove app's cache
adb shell rm -rf /data/dalvik-cache/*/suspect.package.name*

# Check and remove scheduled jobs
adb shell dumpsys jobscheduler | grep suspect.package.name
adb shell cmd jobscheduler cancel suspect.package.name job_id

# Remove app's users
adb shell pm remove-user $(adb shell pm list users | grep suspect.package.name)
```

### System Recovery After Intrusion

```bash
# Verify system integrity
adb shell dm verity status
adb shell getprop ro.boot.verifiedbootstate

# Check for persistence mechanisms
adb shell ls -lR /system/etc/init/
adb shell ls -lR /system/etc/init.d/
adb shell grep -r "suspect.package.name" /system/

# Reset app permissions
adb shell pm reset-permissions
adb shell pm revoke suspect.package.name android.permission.SYSTEM_ALERT_WINDOW

# Clean system logs
adb shell logcat -c
adb shell dmesg -c

# Reset network settings
adb shell settings delete global http_proxy
adb shell settings delete global global_http_proxy_host
adb shell settings delete global global_http_proxy_port

# Check for modified system apps
adb shell pm list packages -f -s

# Verify boot partition
adb shell cat /proc/cmdline

# Reset security settings
adb shell settings put global development_settings_enabled 0
adb shell settings put global adb_enabled 0
```

### AOSP and Custom ROM Development

#### Setting Up Build Environment
```bash
# Install required packages (Ubuntu/Debian)
sudo apt-get update && sudo apt-get install git-core gnupg flex bison build-essential zip curl zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386 lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z1-dev libgl1-mesa-dev libxml2-utils xsltproc unzip fontconfig

# Install repo tool
mkdir ~/bin
PATH=~/bin:$PATH
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo

# Configure Git
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

#### Downloading AOSP Source
```bash
# Create source directory
mkdir AOSP
cd AOSP

# Initialize repo tool with AOSP
repo init -u https://android.googlesource.com/platform/manifest -b android-13.0.0_r1

# For specific device trees, add local manifest:
mkdir -p .repo/local_manifests
cat > .repo/local_manifests/local_manifest.xml << EOF
<?xml version="1.0" encoding="UTF-8"?>
<manifest>
  <project name="device_manufacturer_codename" 
           path="device/manufacturer/codename"
           remote="github"
           revision="branch" />
</manifest>
EOF

# Sync the source
repo sync -c -j$(nproc --all)
```

#### Building AOSP
```bash
# Set up environment
source build/envsetup.sh

# Choose target device
lunch aosp_device_codename-userdebug

# Clean build directory (optional)
make clean
make clobber

# Start the build
make -j$(nproc --all)

# The output will be in out/target/product/device/
```

#### Custom ROM Development
```bash
# Clone device-specific repositories
git clone https://github.com/manufacturer/device_codename.git device/manufacturer/codename
git clone https://github.com/manufacturer/vendor_codename.git vendor/manufacturer/codename
git clone https://github.com/manufacturer/kernel_codename.git kernel/manufacturer/codename

# Apply patches
cd device/manufacturer/codename
git apply < ~/patches/custom_feature.patch

# Modify device configs
vim device/manufacturer/codename/device.mk
vim device/manufacturer/codename/BoardConfig.mk

# Add custom features
mkdir -p overlay/frameworks/base/core/res/res/values/
vim overlay/frameworks/base/core/res/res/values/config.xml
```

#### ROM Installation and Testing
```bash
# Create signed release package
make otapackage
# or for test builds
make bacon

# Push ROM to device
adb push out/target/product/device/rom_name.zip /sdcard/

# In TWRP recovery:
adb shell twrp wipe system
adb shell twrp wipe data
adb shell twrp wipe cache
adb shell twrp wipe dalvik
adb shell twrp install /sdcard/rom_name.zip
```

#### Custom ROM Debugging
```bash
# Enable verbose logging
adb shell setprop persist.sys.verbose LogLevel

# Capture detailed logs during boot
adb shell dmesg > dmesg.log
adb logcat -d > boot_logcat.log

# Check SELinux denials
adb shell cat /proc/last_kmsg > last_kmsg.log
adb shell dmesg | grep "avc: denied" > selinux_denials.log

# Monitor system services
adb shell watch service list

# Test hardware features
adb shell dumpsys sensorservice
adb shell dumpsys battery
adb shell dumpsys wifi
```

#### ROM Source Control and Versioning
```bash
# Create branch for custom features
cd device/manufacturer/codename
git checkout -b custom_feature

# Track changes
git status
git add modified_files
git commit -m "feature: Add custom implementation"

# Create patch for sharing
git format-patch -1 HEAD

# Export changes as patch series
git format-patch -M master

# Apply patch series
git am < series
```

#### Building Recovery Images
```bash
# Build TWRP
source build/envsetup.sh
lunch omni_device-eng
make -j$(nproc --all) recoveryimage

# Flash custom recovery
fastboot flash recovery out/target/product/device/recovery.img

# Backup stock recovery
adb shell dd if=/dev/block/bootdevice/by-name/recovery of=/sdcard/stock_recovery.img
adb pull /sdcard/stock_recovery.img
```

### Chip-Specific ROM Development

#### Allwinner A100/A133 Development
```bash
# Clone Allwinner BSP
git clone https://github.com/allwinner-android/android_bsp.git
cd android_bsp

# Setup build environment for Allwinner
source build/envsetup.sh
lunch # Select your device configuration

# Allwinner-specific kernel configuration
cd kernel/allwinner
make sunxi_defconfig
make menuconfig

# Common Allwinner patches
# Fix GPU driver issues
patch -p1 < patches/gpu/mali-450-fixes.patch

# Configure display parameters
vim device/allwinner/a100/BoardConfig.mk
# Add display configuration:
TARGET_DISPLAY_HDMI_SUPPORT := true
TARGET_DISPLAY_LCD_SUPPORT := true
TARGET_DISPLAY_RESOLUTION := 1920x1080

# Optimize memory configuration
vim device/allwinner/a100/init.rc
# Add:
on boot
    write /proc/sys/vm/swappiness 60
    write /proc/sys/vm/dirty_background_ratio 5
    write /proc/sys/vm/dirty_ratio 90
```

#### Allwinner Debug Commands
```bash
# Debug display issues
adb shell cat /sys/class/disp/disp/attr/sys
adb shell cat /sys/power/wake_lock

# Check DRAM frequency
adb shell cat /sys/class/devfreq/sunxi-ddrfreq/cur_freq

# Monitor thermal status
adb shell cat /sys/class/thermal/thermal_zone0/temp

# Debug GPU issues
adb shell cat /sys/modules/mali/parameters/
```

#### Rockchip RK31xx/RK33xx Development
```bash
# Clone Rockchip BSP
git clone https://github.com/rockchip-android/manifest.git -b android-11.0
repo init -u https://github.com/rockchip-android/manifest -b android-11.0
repo sync -j$(nproc --all)

# Setup Rockchip build environment
source build/envsetup.sh
lunch rk3399_android-userdebug # or your specific model

# Rockchip kernel configuration
cd kernel
make rockchip_defconfig
make menuconfig

# Apply common Rockchip patches
# Fix USB OTG
patch -p1 < device/rockchip/common/patches/usb/otg-fix.patch

# Configure display and GPU
vim device/rockchip/rk3399/BoardConfig.mk
# Add:
BOARD_SUPPORT_HDMI := true
BOARD_SUPPORT_DP := true
TARGET_BOARD_PLATFORM_GPU := mali-t860
```

#### RK31xx/RK33xx Optimization
```bash
# CPU frequency optimization
adb shell
echo performance > /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor
echo performance > /sys/devices/system/cpu/cpu4/cpufreq/scaling_governor

# GPU frequency settings
echo performance > /sys/class/misc/mali0/device/devfreq/ff9a0000.gpu/governor

# Memory optimization
vim device/rockchip/rk3399/init.rc
# Add:
on boot
    write /proc/sys/vm/page-cluster 0
    write /proc/sys/kernel/sched_child_runs_first 1
    write /sys/block/mmcblk0/queue/read_ahead_kb 2048
```

#### Advanced Debugging for Both Platforms
```bash
# Debug boot process
adb shell dmesg | grep -i "fail\|error\|denied"

# Monitor CPU frequencies
adb shell watch -n1 "cat /sys/devices/system/cpu/cpu*/cpufreq/scaling_cur_freq"

# Check thermal throttling
adb shell cat /sys/class/thermal/thermal_zone*/temp

# GPU debugging
adb shell dumpsys gfxinfo

# Memory analysis
adb shell procrank
adb shell cat /proc/meminfo
```

#### Custom Vendor Modifications

##### Allwinner A100/A133
```bash
# Modify boot logo
imgtools -pack boot_logo.img resources/

# Configure secure boot
build/allwinner/pack_tools/pack -p sun50iw11p1_android_secure

# Optimize display driver
vim hardware/aw/display/display.c
# Add custom timings:
static struct disp_video_timings custom_timing = {
    .pixel_clock = 74250000,
    .hor_front_porch = 110,
    .hor_back_porch = 220,
    .hor_sync_time = 40,
    .ver_front_porch = 5,
    .ver_back_porch = 20,
    .ver_sync_time = 5,
};
```

##### Rockchip RK31xx/RK33xx
```bash
# Configure secure boot
./boot_merger --pack --verbose --loader loader.bin

# Modify GPU frequency table
vim drivers/gpu/arm/mali400/platform/rk/mali_scaling.h
# Modify:
static u32 mali_dvfs_clk[] = {
    100000,
    200000,
    300000,
    400000,
};

# Custom charging animation
vim device/rockchip/common/charge/images.mk
# Add:
PRODUCT_COPY_FILES += \
    device/rockchip/common/charge/images/battery_0.bmp:system/etc/charge/battery_0.bmp \
    device/rockchip/common/charge/images/battery_1.bmp:system/etc/charge/battery_1.bmp
```

#### Performance Profiling Tools
```bash
# CPU profiling
adb shell simpleperf stat -p $(pidof com.android.systemui)

# Memory tracking
adb shell dumpsys meminfo -a > meminfo.txt

# GPU profiling
adb shell dumpsys gfxinfo com.android.systemui framestats

# I/O performance
adb shell iostat -m 1

# Network performance
adb shell netstat -i
```

#### Recovery and Firmware Tools
```bash
# Allwinner firmware update
sudo sunxi-fel -p 
sudo sunxi-fel write 0x4A000000 u-boot-sunxi-with-spl.bin
sudo sunxi-fel exe 0x4A000000

# Rockchip firmware update
sudo upgrade_tool ef MiniLoaderAll.bin
sudo upgrade_tool wl 0 Image
sudo upgrade_tool rd

# Emergency recovery
# Allwinner
sudo sunxi-fel spl recovery-spl.bin
sudo sunxi-fel -p write 0x4A000000 recovery.img

# Rockchip
sudo upgrade_tool ef update.img
```

### Advanced Boot Process Debugging

#### Detailed Boot Analysis
```bash
# Capture complete boot log
adb shell dmesg > boot_log.txt
adb logcat -d > logcat_boot.txt

# Monitor init process
adb shell dmesg | grep -i "init:"

# Track service startup
adb shell cat /proc/last_kmsg > last_boot.txt
adb shell getprop > properties_dump.txt

# Analyze boot timing
adb shell dmesg | grep "initcall" > initcall_debug.txt
```

#### Allwinner Boot Debugging
```bash
# Debug SPL loading
sunxi-fel ver
sunxi-fel hexdump 0x0 0x200

# Monitor DDR initialization
adb shell cat /sys/class/sunxi_dump/sunxi_dump
adb shell cat /sys/class/disp/disp/attr/sys

# Track boot stages
adb shell cat /proc/cmdline
adb shell cat /proc/interrupts
```

#### Rockchip Boot Analysis
```bash
# Check loader status
rkdeveloptool ld
rkdeveloptool rd

# Monitor boot sequence
adb shell cat /proc/rk_dmsg
adb shell cat /sys/kernel/debug/clk/clk_summary

# Analyze DDR frequency
adb shell cat /sys/class/devfreq/dmc/cur_freq
adb shell cat /sys/class/devfreq/dmc/available_frequencies
```

### Advanced Performance Optimization

#### CPU and GPU Optimization
```bash
# CPU governor tweaks
adb shell "echo interactive > /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor"
adb shell "echo 90 > /sys/devices/system/cpu/cpu0/cpufreq/interactive/target_loads"

# GPU frequency control
# Allwinner
adb shell "echo 0 > /sys/devices/platform/mali-utgard/dvfs_enable"
adb shell "echo 432000000 > /sys/devices/platform/mali-utgard/max_freq"

# Rockchip
adb shell "echo performance > /sys/class/misc/mali0/device/devfreq/ff9a0000.gpu/governor"
adb shell "echo 800000000 > /sys/class/misc/mali0/device/devfreq/ff9a0000.gpu/max_freq"
```

#### Memory Optimization
```bash
# Kernel memory parameters
adb shell "echo 1 > /proc/sys/vm/compact_memory"
adb shell "echo 0 > /proc/sys/vm/swappiness"
adb shell "echo 100 > /proc/sys/vm/vfs_cache_pressure"

# ZRAM configuration
adb shell "echo 1073741824 > /sys/block/zram0/disksize"
adb shell "mkswap /dev/block/zram0"
adb shell "swapon /dev/block/zram0"
```

#### I/O Optimization
```bash
# Storage optimization
adb shell "echo noop > /sys/block/mmcblk0/queue/scheduler"
adb shell "echo 2048 > /sys/block/mmcblk0/queue/read_ahead_kb"

# File system trim
adb shell "fstrim -v /data"
adb shell "fstrim -v /cache"
```

### Hardware Driver Modifications

#### Display Driver Modifications
```bash
# Allwinner display debugging
adb shell "echo 1 > /sys/class/disp/disp/attr/debug"
adb shell cat /sys/class/disp/disp/attr/debug

# Custom timing implementation
vim kernel/drivers/video/fbdev/sunxi/disp2/disp/dev_disp.c
'''c
static struct disp_video_timings custom_timing = {
    .pixel_clk = 297000000,
    .hor_pixels = 3840,
    .ver_pixels = 2160,
    .hor_total_time = 4400,
    .ver_total_time = 2250,
    .hor_front_porch = 176,
    .hor_back_porch = 296,
    .ver_front_porch = 8,
    .ver_back_porch = 72,
};
'''
```

#### Touch Driver Modifications
```bash
# Debug touch events
adb shell getevent -l

# Modify touch sensitivity
vim kernel/drivers/input/touchscreen/gsl3673/gsl_ts_driver.c
'''c
#define GSL_PRESSURE_FACTOR 25
#define GSL_TIMER_INTERVAL 12
'''

# Test touch response
adb shell sendevent /dev/input/event2 3 57 1
adb shell sendevent /dev/input/event2 3 53 500
adb shell sendevent /dev/input/event2 3 54 500
```

#### Audio Driver Tweaks
```bash
# Debug audio routing
adb shell tinymix
adb shell tinyplay /sdcard/test.wav

# Modify audio parameters
vim kernel/sound/soc/sunxi/sun50i-codec.c
'''c
static const struct reg_default sun50i_codec_reg_defaults[] = {
    { SUNXI_DAC_DPC,    0x84000000 },
    { SUNXI_DAC_FIFOC,  0x00000000 },
    { SUNXI_DAC_VOL,    0xa0a0 },
};
'''
```

### Extended Recovery Scenarios

#### Allwinner Recovery Options
```bash
# Recovery from brick (FEL mode)
sudo sunxi-fel -l
sudo sunxi-fel write 0x4A000000 boot0.bin
sudo sunxi-fel write 0x4A800000 uboot.bin
sudo sunxi-fel exe 0x4A800000

# eMMC recovery
sudo sunxi-fel -p spl recovery-emmc.bin
dd if=/dev/zero of=/dev/mmcblk0 bs=1M count=32
dd if=u-boot-sunxi-with-spl.bin of=/dev/mmcblk0 bs=1024 seek=8

# NAND recovery
sudo sunxi-nand-part -f a64 /dev/mtd0
sunxi-nand-image-builder -c config.cfg
```

#### Rockchip Recovery Procedures
```bash
# Loader mode recovery
sudo rkdeveloptool rd
sudo rkdeveloptool wl 0 loader.bin

# Partition table recovery
sudo rkdeveloptool gpt
sudo rkdeveloptool prm 0x00000040 parameter.txt

# Flash entire image
sudo rkdeveloptool wl 0 Image
sudo rkdeveloptool ul update.img
```

### Advanced Security Configurations

#### Secure Boot Implementation
```bash
# Allwinner secure boot
# Generate keys
openssl genrsa -out private.pem 2048
openssl rsa -in private.pem -pubout -out public.pem

# Sign bootloader
sudo sunxi-fel-sign -k private.pem boot0.bin boot0.bin.signed

# Rockchip secure boot
# Generate keys
./boot_merger --rsa_len 2048 --rsa_sha 256 --rsa_gen_keys

# Sign images
./boot_merger --pack --rsa_key private.pem
```

#### SELinux Policy Modifications
```bash
# Check current policy
adb shell sesearch --allow
adb shell seinfo

# Create custom policy
vim device/manufacturer/device/sepolicy/file_contexts
'''
/dev/block/platform/sunxi-mmc.0/by-name/UDISK     u:object_r:userdata_block_device:s0
/dev/block/platform/ff0f0000.dwmmc/by-name/user   u:object_r:userdata_block_device:s0
'''

# Compile and push new policy
make sepolicy
adb push out/target/product/device/root/sepolicy /sepolicy
```

#### Verified Boot Configuration
```bash
# Enable dm-verity
vim device/manufacturer/device/fstab.device
'''
/dev/block/platform/sunxi-mmc.0/by-name/system    /system    ext4    ro,barrier=1    wait,verify
'''

# Generate verity metadata
make veritysetup
verity_tree_id=$(make -j1 -f build/tools/verity/verity_tree BUILD_NUMBER=0000 PRODUCT_OUT=out/target/product/device get-verity-metadata)
```

#### Encryption Setup
```bash
# Enable FDE
vim device/manufacturer/device/fstab.device
'''
/dev/block/platform/sunxi-mmc.0/by-name/userdata    /data    ext4    noatime,nosuid,nodev,barrier=1,noauto_da_alloc    wait,check,encryptable=footer
'''

# Enable FBE
vim device/manufacturer/device/fstab.device
'''
/dev/block/platform/sunxi-mmc.0/by-name/userdata    /data    ext4    noatime,nosuid,nodev,barrier=1,noauto_da_alloc    wait,check,fileencryption=aes-256-xts:aes-256-cts
'''
```

### APKTool and APK Manipulation

#### APKTool Setup and Usage
```bash
# Download latest APKTool
wget https://bitbucket.org/iBotPeaches/apktool/downloads/apktool_2.7.0.jar -O apktool.jar
wget https://raw.githubusercontent.com/iBotPeaches/Apktool/master/scripts/linux/apktool -O apktool

# Setup APKTool
chmod +x apktool
chmod +x apktool.jar
sudo mv apktool /usr/local/bin/
sudo mv apktool.jar /usr/local/bin/

# Basic APKTool commands
# Decompile APK
apktool d app.apk -o app_decoded

# Recompile APK
apktool b app_decoded -o app_modified.apk

# Force decompile (bypass errors)
apktool d -f app.apk -o app_decoded

# Keep original resources
apktool d -r app.apk -o app_decoded
```

#### Advanced APKTool Operations
```bash
# Decompile with specific framework
apktool if framework-res.apk
apktool d --frame-path=/path/to/framework app.apk

# Debug mode decompilation
apktool d -d app.apk

# Handle resource tables
apktool empty-framework-dir
apktool install-framework framework-res.apk

# Extract specific files
apktool d app.apk -s -o app_decoded

# Use specific API level
apktool d -api 25 app.apk
```

#### APK Modification Examples
```bash
# Modify app permissions
vim app_decoded/AndroidManifest.xml

# Change app name
vim app_decoded/res/values/strings.xml

# Modify layout
vim app_decoded/res/layout/activity_main.xml

# Edit Smali code
vim app_decoded/smali/com/example/app/MainActivity.smali

# Add new resources
cp new_image.png app_decoded/res/drawable/

# Modify configurations
vim app_decoded/apktool.yml
```

#### APK Signing and Verification
```bash
# Generate keystore
keytool -genkey -v -keystore debug.keystore \
  -alias androiddebugkey -keyalg RSA -keysize 2048 \
  -validity 10000 -storepass android -keypass android

# Sign APK (old method)
jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 \
  -keystore debug.keystore app_modified.apk androiddebugkey

# Sign APK (new method)
apksigner sign --ks debug.keystore app_modified.apk

# Verify signature
apksigner verify app_modified.apk

# Check APK content
aapt dump badging app_modified.apk
```

### Verity Setup and Management

#### Basic Verity Setup
```bash
# Generate verity key
openssl genrsa -out verity_key.pem 2048
openssl rsa -in verity_key.pem -pubout -out verity_key.pub

# Convert key to DER format
openssl pkcs8 -in verity_key.pem -topk8 -nocrypt -outform DER -out verity_key.der

# Setup verity for system partition
make_ext4fs -L system -S file_contexts -l 2147483648 \
  -a system system.img system_dir/
fec -v -e -p system.img

# Enable verity in fstab
vim device/manufacturer/device/fstab.device
'''
/dev/block/platform/soc/1da4000.ufshc/by-name/system /system ext4 ro,barrier=1 wait,verify
'''
```

#### Advanced Verity Configuration
```bash
# Generate verity metadata
verity_tree_id=$(make -j1 -f build/tools/verity/verity_tree \
  BUILD_NUMBER=0000 PRODUCT_OUT=out/target/product/device \
  get-verity-metadata)

# Setup device mapper
dm_setup_device() {
    local name=$1
    local dev=$2
    local ro=$3
    local target=$4
    local args=$5
    echo "0 $(blockdev --getsize $dev) $target $args" | \
        dmsetup create $name
}

# Enable verity on custom partition
veritysetup format \
    --hash=sha256 \
    --data-block-size=4096 \
    --hash-block-size=4096 \
    --salt=salt \
    system.img \
    system_verity.img
```

#### Verity Testing and Verification
```bash
# Test verity setup
veritysetup verify system.img system_verity.img

# Check verity status
adb shell dm-tool status
adb shell cat /sys/module/dm_verity/parameters/status

# Verify partition hash
dd if=/dev/block/bootdevice/by-name/system bs=4096 | \
    sha256sum > system_hash.txt

# Test verity performance
time dd if=/dev/block/dm-0 of=/dev/null bs=4M

# Debug verity issues
adb shell cat /proc/sys/kernel/panic_on_corruption
adb shell dmesg | grep -i "dm-verity"
```

#### Handling Verity Errors
```bash
# Disable verity for testing
adb disable-verity
adb reboot

# Re-enable verity
adb enable-verity
adb reboot

# Force disable verity (emergency)
adb shell magisk resetprop ro.boot.veritymode disabled
adb shell magisk resetprop ro.boot.verifiedbootstate orange

# Verify boot state
adb shell getprop ro.boot.verifiedbootstate
adb shell getprop ro.boot.flash.locked
```

#### Custom Verity Implementation
```bash
# Create custom verity table
echo "1 device_size verity version hashtype device hashdevice data_block_size hash_block_size num_data_blocks hash_start_block algorithm root_digest salt" > verity_table

# Load custom table
dmsetup create veritydev --table "$(cat verity_table)"

# Monitor verity events
adb shell "echo 1 > /sys/module/dm_verity/parameters/debug"
adb shell dmesg -w | grep "dm-verity"

# Custom error handling
vim device/manufacturer/device/init.rc
'''
on verity-logging
    exec u:r:su:s0 root root -- /system/bin/handle_verity_error.sh
'''
```

### Advanced APK Modification Scenarios

#### Complex APK Modifications
```bash
# Inject custom code into existing methods
# Find target method in smali
grep -r "targetMethod" app_decoded/smali/

# Add logging
.method public targetMethod()V
    .locals 2
    
    const-string v0, "DebugTag"
    const-string v1, "Method called"
    invoke-static {v0, v1}, Landroid/util/Log;->d(Ljava/lang/String;Ljava/lang/String;)I
    
    # Original method code
    return-void
.end method

# Modify native libraries
# Extract lib
cp app_decoded/lib/arm64-v8a/libnative.so ./
# Patch using hex editor
hexedit libnative.so
# Repack
cp libnative.so app_decoded/lib/arm64-v8a/

# Add custom activities
vim app_decoded/AndroidManifest.xml
'''xml
<activity android:name=".NewActivity"
    android:exported="false">
    <intent-filter>
        <action android:name="com.example.CUSTOM_ACTION" />
    </intent-filter>
</activity>
'''

# Create corresponding smali
vim app_decoded/smali/com/example/app/NewActivity.smali
```

#### Advanced Method Hooking
```bash
# Hook system calls
.method public static hook_SystemProperties()V
    .locals 4
    
    const-string v0, "ro.secure"
    const-string v1, "0"
    
    invoke-static {v0, v1}, Landroid/os/SystemProperties;->set(Ljava/lang/String;Ljava/lang/String;)V
    return-void
.end method

# Modify app signing verification
.method public static disableSignatureCheck()V
    .locals 2
    
    const/4 v0, 0x1
    const-string v1, "signature_check_disabled"
    invoke-static {v1, v0}, Landroid/os/SystemProperties;->set(Ljava/lang/String;Z)V
    return-void
.end method
```

### Advanced Signing Key Analysis

#### Key Extraction Techniques
```bash
# Extract certificate from APK
unzip -p app.apk META-INF/CERT.RSA | keytool -printcert

# Dump certificate details
openssl pkcs7 -in META-INF/CERT.RSA -print_certs -inform DER

# Extract public key
openssl x509 -inform DER -in META-INF/CERT.RSA -pubkey -noout > pubkey.pem

# Analyze keystore
keytool -list -v -keystore keystore.jks

# Brute force keystore password
for pass in $(cat wordlist.txt); do
    keytool -list -keystore keystore.jks -storepass "$pass" 2>/dev/null
    if [ $? -eq 0 ]; then
        echo "Password found: $pass"
        break
    fi
done
```

#### Hash Cracking Methods
```bash
# Extract hash from keystore
keytool -list -v -keystore keystore.jks > keystore_info.txt
grep -A2 "MD5:" keystore_info.txt > hash.txt

# Use hashcat
hashcat -m 9900 hash.txt wordlist.txt
hashcat -m 9900 hash.txt -r rules/best64.rule wordlist.txt

# Custom rainbow table generation
rtgen md5 loweralpha-numeric 1 8 0 1000 1000 example
rtsort example.rt
rtdump example.rt > rainbow_table.txt
```

### Enhanced Dynamic Partition Management

#### Basic Dynamic Partition Operations
```bash
# Check dynamic partition status
adb shell dmctl list
adb shell ls -l /dev/block/mapper/

# Resize dynamic partition
adb shell dmctl resize system_a size_in_bytes

# Create new dynamic partition
adb shell dmctl create new_partition size_in_bytes

# Delete dynamic partition
adb shell dmctl delete partition_name

# Convert static to dynamic
adb shell sgdisk --resize-table=64 /dev/block/sda
```

#### Advanced Dynamic Partition Management
```bash
# Modify super partition table
adb shell lptools modify super_empty.img \
    --metadata-size 65536 \
    --super-name super \
    --group main:4294967296

# Create custom partition layout
cat > layout.txt << EOF
system,2147483648,main
vendor,1073741824,main
product,536870912,main
EOF

# Build super image
adb shell lpmake \
    --metadata-size 65536 \
    --super-name super \
    --metadata-slots 2 \
    --device super:4294967296 \
    --group main:4294967296 \
    --partition system_a:readonly:2147483648:main \
    --partition vendor_a:readonly:1073741824:main \
    --image-list layout.txt \
    --sparse \
    --output super.img
```

#### Dynamic Partition Recovery
```bash
# Backup super partition
adb shell dd if=/dev/block/by-name/super of=/sdcard/super.img

# Analyze super partition
adb shell lptools info /dev/block/by-name/super

# Fix corrupted metadata
adb shell lptools recover /dev/block/by-name/super

# Emergency partition resize
adb shell dmctl resize-all
```

### Advanced Verity Security

#### Custom Verity Chains
```bash
# Create chained verification
openssl genrsa -out root_key.pem 4096
openssl req -new -key root_key.pem -out root_cert.pem

openssl genrsa -out signing_key.pem 2048
openssl req -new -key signing_key.pem \
    -out signing_cert.pem \
    -CA root_cert.pem \
    -CAkey root_key.pem

# Implement chain verification
verify_chain() {
    openssl verify -CAfile root_cert.pem signing_cert.pem
    openssl verify -CAfile signing_cert.pem target_cert.pem
}
```

#### Advanced Error Handling
```bash
# Custom error handler script
cat > handle_verity_error.sh << 'EOF'
#!/system/bin/sh
log_error() {
    local error_code=$1
    local partition=$2
    echo "$(date) Verity error $error_code on $partition" \
        >> /data/verity_errors.log
}

handle_corruption() {
    local partition=$1
    # Attempt recovery
    if [ -e "/data/backup/${partition}.img" ]; then
        dd if=/data/backup/${partition}.img \
           of=/dev/block/by-name/${partition}
        return 0
    fi
    return 1
}

main() {
    case "$1" in
        "corruption")
            log_error 1 "$2"
            handle_corruption "$2"
            ;;
        "signature")
            log_error 2 "$2"
            # Custom signature handling
            ;;
    esac
}

main "$@"
EOF
```

### Device-Specific Implementations

#### MediaTek Specific
```bash
# Extract preloader
./mtk_preloader_tools -extract preloader.bin

# Dump BROM
./mtk_brom_dumper /dev/ttyACM0

# Bypass AUTH
./bypass_auth.py --port /dev/ttyACM0

# Flash with scatter
./mtk_flash_tool -scatter MT6789_Android_scatter.txt -download
```

#### Qualcomm Specific
```bash
# EDL mode operations
./edl.py --loader prog_emmc_firehose_8916.mbn --memory emmc

# Dump partitions in EDL
./edl.py --dump partition gpt_backup1.bin

# Program partitions
./edl.py --write partition file.bin

# Qualcomm-specific verity
./verity_tool --partition system \
    --key verity.pk8 \
    --cert verity.x509.pem
```

### Advanced Debugging Techniques

#### Runtime Analysis
```bash
# Monitor method calls
adb shell am trace-ipc start com.example.app

# Analyze native crashes
adb shell debuggerd -b $(pid)

# Profile app performance
adb shell simpleperf record -p $(pid)
adb shell simpleperf report

# Memory analysis
adb shell am dumpheap $(pid) /data/local/tmp/heap.hprof
```

#### Advanced Logging
```bash
# Enable verbose logging
adb shell setprop log.tag.suspect_app VERBOSE

# Monitor specific tags
adb logcat -v threadtime *:S MYTAG:V

# Create log points
adb shell log -p v -t MYTAG "Custom checkpoint reached"

# Track system events
adb shell dumpsys activity service com.example.app
```

### Extended Device-Specific Implementations

#### Samsung Specific
```bash
# Enter download mode
adb reboot download

# Odin operations
# Flash with Heimdall
sudo heimdall detect
sudo heimdall print-pit > pit.txt
sudo heimdall flash --RECOVERY recovery.img --no-reboot

# Knox operations
adb shell pm list packages | grep knox
adb shell am force-stop com.samsung.knox.containeragent

# RMM/KG State
adb shell getprop ro.boot.warranty_bit
adb shell getprop ro.boot.kmflag
```

#### Huawei/HiSilicon Specific
```bash
# Enter fastboot
adb reboot bootloader

# Check bootloader status
fastboot getvar unlocked

# Flash with HiSuite
./update_hi_suite.sh --proloader=proloader.img \
    --fastboot=fastboot.img \
    --system=system.img

# Handle FRP
fastboot flash frp_a frp.img
fastboot erase frp

# Project Config
./hiconfig --create-config \
    --board hi3660 \
    --product HWMHA
```

#### OPPO/OnePlus Specific
```bash
# MSMDownloadTool operations
./msmdownloadtool --port=/dev/ttyUSB0 \
    --rawprogram=rawprogram.xml \
    --patch=patch.xml \
    --sender=8096

# ColorOS operations
adb shell oplus_log
adb shell dumpsys oplussensor

# Handle .ozip encryption
python3 ozipdecrypt.py firmware.ozip

# Engineer mode
adb shell am start com.oplus.engineermode/.EngineerMode
```

### Advanced Partition Management

#### A/B Partition Handling
```bash
# Check current slot
adb shell getprop ro.boot.slot_suffix

# Force switch slot
adb shell bootctl set-active-boot-slot 1

# Copy between slots
dd if=/dev/block/by-name/system_a \
   of=/dev/block/by-name/system_b \
   bs=4M

# Sync partitions
for part in system vendor boot; do
    adb shell dd if=/dev/block/by-name/${part}_a \
        of=/dev/block/by-name/${part}_b \
        bs=4M
done
```

#### Logical Partition Management
```bash
# Create super partition layout
cat > super_layout.txt << EOF
metadata_size=65536
metadata_slots=3
super_name=super
size=6442450944

[group:main]
size=6442450944
group=main

[partition:system_a]
size=2147483648
group=main

[partition:vendor_a]
size=1073741824
group=main
EOF

# Build super image
lpmake --metadata-size 65536 \
    --super-name super \
    --metadata-slots 3 \
    --device super:6442450944 \
    --group main:6442450944 \
    --partition system_a:readonly:2147483648:main \
    --partition vendor_a:readonly:1073741824:main \
    --output super.img

# Resize partitions on-the-fly
lptools resize system_a 3221225472
```

#### Recovery Partition Operations
```bash
# Backup all partitions
for part in $(adb shell ls /dev/block/by-name/); do
    adb shell dd if=/dev/block/by-name/$part \
        of=/sdcard/backup/$part.img bs=4M
done

# Create custom partition table
cat > partition.xml << EOF
<?xml version="1.0" encoding="UTF-8"?>
<partition_layout version="1.0">
    <partition name="boot" size="67108864" />
    <partition name="system" size="3221225472" />
    <partition name="vendor" size="1073741824" />
    <partition name="userdata" size="remaining" />
</partition_layout>
EOF

# Apply partition table
fastboot flash partition partition.xml
```

### Enhanced Debugging Tools

#### Memory Analysis Tools
```bash
# Native heap analysis
adb shell heapsnap -p $(pidof com.example.app)

# Track memory allocations
adb shell am start-activity \
    --track-allocation com.example.app/.MainActivity

# Memory leak detection
adb shell dumpsys meminfo -a com.example.app > meminfo.txt
python3 analyze_memory.py meminfo.txt

# Heap diffing
adb shell am dumpheap $(pidof com.example.app) before.hprof
# Perform actions
adb shell am dumpheap $(pidof com.example.app) after.hprof
python3 compare_heaps.py before.hprof after.hprof
```

#### Process Tracing Tools
```bash
# Trace system calls
adb shell strace -p $(pidof com.example.app)

# Profile CPU usage
adb shell simpleperf record -p $(pidof com.example.app)
adb shell simpleperf report

# Monitor file operations
adb shell inotifywait -m -r /data/data/com.example.app/

# Track binder transactions
adb shell debuggerd -b $(pidof com.example.app)
```

#### Network Analysis
```bash
# Capture network traffic
adb shell tcpdump -i any -w /sdcard/capture.pcap

# Monitor specific ports
adb shell netstat -tunp | grep com.example.app

# SSL traffic analysis
adb shell "SSLKEYLOGFILE=/sdcard/ssl-keys.log \
    am start -n com.example.app/.MainActivity"

# Track socket operations
adb shell strace -e trace=network -p $(pidof com.example.app)
```

### Advanced Security Configurations

#### Custom SELinux Policies
```bash
# Create custom SELinux policy
cat > custom.te << EOF
type custom_domain;
type custom_file;

allow custom_domain custom_file:file { read write open };
allow custom_domain system_file:dir search;
EOF

# Compile policy
checkmodule -M -m -o custom.mod custom.te
semodule_package -o custom.pp -m custom.mod

# Load policy
adb push custom.pp /data/local/tmp/
adb shell su -c "semodule -i /data/local/tmp/custom.pp"
```

#### Advanced Encryption
```bash
# Generate encryption key
openssl genpkey -algorithm RSA -out private.pem -pkeyopt rsa_keygen_bits:4096

# Implement custom encryption
cat > encrypt_partition.sh << 'EOF'
#!/system/bin/sh
BLOCK_SIZE=4096
CIPHER="aes-256-xts"

setup_encryption() {
    local device=$1
    local keyfile=$2
    
    # Generate key
    dd if=/dev/urandom of=$keyfile bs=512 count=1
    
    # Setup dm-crypt
    cryptsetup luksFormat --type luks2 \
        --cipher $CIPHER \
        --key-size 512 \
        --hash sha512 \
        $device $keyfile
}

mount_encrypted() {
    local device=$1
    local keyfile=$2
    local mountpoint=$3
    
    cryptsetup luksOpen --key-file $keyfile \
        $device encrypted_data
        
    mount /dev/mapper/encrypted_data $mountpoint
}
EOF
```

#### Security Hardening
```bash
# Disable ADB authentication
adb shell settings put global adb_enabled 0

# Enable secure startup
cat > secure_boot.rc << EOF
on init
    # Verify boot image
    exec u:r:su:s0 root root -- /system/bin/verify_boot_image.sh
    
    # Apply security settings
    write /proc/sys/kernel/kptr_restrict 2
    write /proc/sys/kernel/dmesg_restrict 1
    write /proc/sys/kernel/perf_event_paranoid 3
EOF

# Implement secure boot checks
cat > verify_boot_image.sh << 'EOF'
#!/system/bin/sh
verify_signature() {
    local image=$1
    local pubkey=$2
    openssl dgst -sha256 -verify $pubkey -signature \
        ${image}.sig $image
}

check_secure_boot() {
    if [ "$(getprop ro.boot.verifiedbootstate)" != "green" ]; then
        return 1
    fi
    return 0
}

main() {
    if ! check_secure_boot; then
        echo "Secure boot verification failed"
        return 1
    fi
    
    for image in boot recovery system; do
        if ! verify_signature \
            /dev/block/by-name/$image \
            /verity_key.pub; then
            echo "Signature verification failed for $image"
            return 1
        fi
    done
}

main "$@"
EOF
```

### Advanced Bootloader Operations

#### Universal Bootloader Analysis
```bash
# Dump bootloader
dd if=/dev/block/bootloader of=/sdcard/bootloader.img
hexdump -C bootloader.img > bootloader_dump.txt

# Memory map analysis
adb shell cat /proc/iomem > iomem.txt
adb shell cat /proc/bootloader/memory_map > bootloader_map.txt

# Extract bootloader parameters
strings bootloader.img | grep -i "version\|serial\|unlock"

# Analyze boot process
adb shell dmesg | grep -i "boot\|loader\|init" > boot_process.log
```

#### Device-Specific Bootloader Operations

##### Xiaomi/MTK
```bash
# Enter EDL mode
adb reboot edl

# Use MTK bypass
./mtk_bypass --port=/dev/ttyACM0 \
    --preloader=preloader.bin \
    --scatter=MT6789_scatter.txt

# Auth bypass
python3 mtk_auth_bypass.py --serial=/dev/ttyACM0 \
    --brom=brom.bin \
    --payload=payload.bin

# Memory dump in EDL
./edl_dump.py --start=0x00000000 --size=0x8000000 \
    --output=memory_dump.bin
```

##### Pixel/Google
```bash
# Bootloader unlock data
fastboot getvar unlock_data

# Custom AVB key registration
python3 make_vbmeta.py --key /path/to/key.pem \
    --algorithm SHA256_RSA4096 \
    --flags 2 \
    --output vbmeta.img

# Verify boot chain
fastboot getvar anti_rollback_version
fastboot getvar boot-verification-key
```

#### Memory Map Analysis

##### Memory Region Analysis
```bash
# Create memory map parser
cat > parse_memmap.py << 'EOF'
#!/usr/bin/env python3
import sys

def parse_memmap(filename):
    regions = []
    with open(filename, 'r') as f:
        for line in f:
            if '-' in line:
                start, end = line.split('-')
                perms = line.split()[1]
                name = line.split()[-1] if len(line.split()) > 5 else "unknown"
                regions.append({
                    'start': int(start, 16),
                    'end': int(end, 16),
                    'perms': perms,
                    'name': name
                })
    return regions

def analyze_regions(regions):
    for region in regions:
        size = region['end'] - region['start']
        print(f"Region: {region['name']}")
        print(f"Start: 0x{region['start']:08x}")
        print(f"End: 0x{region['end']:08x}")
        print(f"Size: {size:,} bytes")
        print(f"Permissions: {region['perms']}\n")

if __name__ == "__main__":
    memmap_file = sys.argv[1]
    regions = parse_memmap(memmap_file)
    analyze_regions(regions)
EOF

chmod +x parse_memmap.py

# Dump and analyze memory regions
adb shell cat /proc/self/maps > memory_regions.txt
./parse_memmap.py memory_regions.txt
```

##### Advanced Memory Analysis
```bash
# Create memory scanner
cat > memory_scanner.py << 'EOF'
#!/usr/bin/env python3
import sys
import struct

def scan_memory(pid, pattern):
    regions = []
    with open(f'/proc/{pid}/maps', 'r') as maps:
        for line in maps:
            addr = line.split()[0]
            start, end = [int(x, 16) for x in addr.split('-')]
            if 'r' in line.split()[1]:
                regions.append((start, end))
    
    matches = []
    with open(f'/proc/{pid}/mem', 'rb') as mem:
        for start, end in regions:
            try:
                mem.seek(start)
                chunk = mem.read(end - start)
                offset = 0
                while True:
                    offset = chunk.find(pattern, offset)
                    if offset == -1:
                        break
                    matches.append(start + offset)
                    offset += 1
            except:
                continue
    return matches

def pattern_search(pid, pattern_file):
    with open(pattern_file, 'rb') as f:
        pattern = f.read()
    return scan_memory(pid, pattern)
EOF

chmod +x memory_scanner.py
```

### Extended Partition Management

#### Advanced Partition Recovery
```bash
# Create partition backup script
cat > backup_partitions.sh << 'EOF'
#!/system/bin/sh

BACKUP_DIR="/sdcard/partition_backup"
mkdir -p $BACKUP_DIR

backup_partition() {
    local partition=$1
    local filename="${BACKUP_DIR}/${partition}_$(date +%Y%m%d_%H%M%S).img"
    
    # Get partition size
    local size=$(blockdev --getsize64 /dev/block/by-name/$partition)
    
    # Backup with progress
    dd if=/dev/block/by-name/$partition of=$filename \
        bs=4M status=progress
    
    # Create checksum
    sha256sum $filename > "${filename}.sha256"
}

# Backup critical partitions
for partition in boot recovery system vendor; do
    backup_partition $partition
done
EOF

chmod +x backup_partitions.sh
```

#### Dynamic Partition Manipulation
```bash
# Create dynamic partition manager
cat > dynamic_partition_manager.sh << 'EOF'
#!/system/bin/sh

create_dynamic_partition() {
    local name=$1
    local size=$2
    local group=$3
    
    lpmake --metadata-size 65536 \
        --super-name super \
        --metadata-slots 2 \
        --device super:$((size * 2)) \
        --group $group:$size \
        --partition ${name}:readonly:$size:$group \
        --sparse \
        --output ${name}.img
}

resize_dynamic_partition() {
    local name=$1
    local new_size=$2
    
    lptools resize $name $new_size
    
    # Verify resize
    if [ $? -eq 0 ]; then
        echo "Successfully resized $name to $new_size bytes"
    else
        echo "Failed to resize $name"
        return 1
    fi
}

EOF

chmod +x dynamic_partition_manager.sh
```

### Advanced Debugging Tools

#### Kernel Debug Tools
```bash
# Create kernel tracer
cat > kernel_tracer.sh << 'EOF'
#!/system/bin/sh

setup_ftrace() {
    echo 1 > /sys/kernel/debug/tracing/tracing_on
    echo function > /sys/kernel/debug/tracing/current_tracer
    echo 1 > /sys/kernel/debug/tracing/events/sched/sched_switch/enable
    echo 1 > /sys/kernel/debug/tracing/events/sched/sched_wakeup/enable
}

trace_process() {
    local pid=$1
    echo $pid > /sys/kernel/debug/tracing/set_ftrace_pid
    cat /sys/kernel/debug/tracing/trace_pipe
}

monitor_syscalls() {
    local pid=$1
    strace -f -tt -T -p $pid 2>&1
}

EOF

chmod +x kernel_tracer.sh
```

#### Memory Leak Detection
```bash
# Create memory leak detector
cat > leak_detector.py << 'EOF'
#!/usr/bin/env python3
import sys
import re
from collections import defaultdict

class MemorySnapshot:
    def __init__(self):
        self.allocations = defaultdict(int)
        
    def parse_trace(self, trace_file):
        with open(trace_file, 'r') as f:
            for line in f:
                if 'malloc' in line:
                    size = int(re.search(r'size=(\d+)', line).group(1))
                    addr = re.search(r'addr=0x([0-9a-f]+)', line).group(1)
                    self.allocations[addr] = size
                elif 'free' in line:
                    addr = re.search(r'addr=0x([0-9a-f]+)', line).group(1)
                    if addr in self.allocations:
                        del self.allocations[addr]
                        
    def get_leaks(self):
        return sum(self.allocations.values())

def main():
    if len(sys.argv) != 2:
        print("Usage: leak_detector.py <trace_file>")
        sys.exit(1)
        
    snapshot = MemorySnapshot()
    snapshot.parse_trace(sys.argv[1])
    print(f"Total memory leaked: {snapshot.get_leaks()} bytes")

if __name__ == "__main__":
    main()
EOF

chmod +x leak_detector.py
```

### Enhanced Security Measures

#### Secure Boot Implementation
```bash
# Create secure boot verifier
cat > verify_secure_boot.sh << 'EOF'
#!/system/bin/sh

check_secure_boot() {
    local bootstate=$(getprop ro.boot.verifiedbootstate)
    local flash_lock=$(getprop ro.boot.flash.locked)
    local vbmeta_digest=$(getprop ro.boot.vbmeta.digest)
    
    # Check boot state
    if [ "$bootstate" != "green" ]; then
        echo "Warning: Device not in verified boot state"
        return 1
    fi
    
    # Check flash lock
    if [ "$flash_lock" != "1" ]; then
        echo "Warning: Bootloader unlocked"
        return 1
    fi
    
    # Verify vbmeta digest
    if [ -z "$vbmeta_digest" ]; then
        echo "Warning: No vbmeta digest found"
        return 1
    fi
    
    return 0
}

verify_partition_hashes() {
    for partition in system vendor boot; do
        local hash=$(sha256sum /dev/block/by-name/$partition)
        local stored_hash=$(cat /verity_hashes/$partition)
        
        if [ "$hash" != "$stored_hash" ]; then
            echo "Warning: Hash mismatch for $partition"
            return 1
        fi
    done
    return 0
}

main() {
    if ! check_secure_boot; then
        echo "Secure boot verification failed"
        exit 1
    fi
    
    if ! verify_partition_hashes; then
        echo "Partition verification failed"
        exit 1
    fi
    
    echo "All security checks passed"
    exit 0
}

main "$@"
EOF

chmod +x verify_secure_boot.sh
```

#### Advanced SELinux Implementation
```bash
# Create custom SELinux policy
cat > custom_selinux.te << EOF
policy_module(custom, 1.0)

type custom_process_t;
type custom_file_t;

allow custom_process_t custom_file_t:file { read write execute };
allow custom_process_t kernel:system module_request;
allow custom_process_t self:capability { net_admin sys_module };

# Network access rules
allow custom_process_t self:tcp_socket { create bind connect listen accept };
allow custom_process_t node_t:tcp_socket node_bind;
allow custom_process_t port_t:tcp_socket name_bind;

# File system rules
allow custom_process_t fs_t:filesystem { mount remount unmount };
allow custom_process_t labeledfs:filesystem remount;

# Process management
allow custom_process_t domain:process { signal sigchld };
allow custom_process_t self:process { fork setcurrent setsched };
EOF

# Compile and load policy
checkmodule -M -m -o custom_selinux.mod custom_selinux.te
semodule_package -o custom_selinux.pp -m custom_selinux.mod
semodule -i custom_selinux.pp
```

### Reverse Engineering Tools

#### Frida Integration
```bash
# Install Frida
pip install frida-tools
adb push frida-server /data/local/tmp/
adb shell "chmod 755 /data/local/tmp/frida-server"

# Start Frida server
adb shell "/data/local/tmp/frida-server &"

# Basic Frida script
cat > trace_crypto.js << 'EOF'
Java.perform(function() {
    var cipher = Java.use('javax.crypto.Cipher');
    cipher.doFinal.overload('[B').implementation = function(bytes) {
        console.log('Crypto operation detected');
        console.log('Input:', bytes);
        var ret = this.doFinal(bytes);
        console.log('Output:', ret);
        return ret;
    };
});
EOF

# Run Frida script
frida -U -l trace_crypto.js com.example.app
```

#### Advanced Frida Usage
```python
# Dynamic hooking script
cat > advanced_hook.js << 'EOF'
Java.perform(function() {
    // Hook class loaders
    Java.enumerateClassLoaders({
        onMatch: function(loader) {
            try {
                Java.classFactory.loader = loader;
                var targetClass = Java.use("com.example.security.Crypto");
                console.log("Found target class in loader:", loader);
            } catch(e) {}
        },
        onComplete: function() {}
    });

    // Hook native library
    Interceptor.attach(Module.findExportByName(null, "JNI_OnLoad"), {
        onEnter: function(args) {
            console.log("JNI_OnLoad called");
            this.context = args[0];
        },
        onLeave: function(retval) {
            console.log("JNI_OnLoad returned:", retval);
        }
    });
});
EOF
```

#### IDA Pro Integration
```bash
# Create IDA Python script
cat > ida_analysis.py << 'EOF'
from idaapi import *
from idautils import *
from idc import *

def find_crypto_functions():
    for func in Functions():
        name = get_func_name(func)
        if "crypt" in name.lower():
            print(f"Found crypto function at {hex(func)}: {name}")
            
def trace_xrefs():
    for func in Functions():
        for ref in XrefsTo(func, 0):
            print(f"Cross-reference to {hex(func)} from {hex(ref.frm)}")

def analyze_strings():
    strings = Strings()
    for s in strings:
        if "key" in str(s).lower():
            print(f"Potential key string at {hex(s.ea)}: {str(s)}")

if __name__ == "__main__":
    find_crypto_functions()
    trace_xrefs()
    analyze_strings()
EOF
```

#### Ghidra Analysis
```java
// Ghidra script for analyzing native libraries
cat > GhidraAnalysis.java << 'EOF'
import ghidra.app.script.GhidraScript;
import ghidra.program.model.listing.*;
import ghidra.program.model.symbol.*;

public class GhidraAnalysis extends GhidraScript {
    @Override
    public void run() throws Exception {
        println("Starting analysis...");
        
        // Find all functions
        FunctionIterator functions = currentProgram.getFunctionManager()
            .getFunctions(true);
        
        // Analyze each function
        for (Function function : functions) {
            // Look for specific patterns
            if (function.getName().contains("decrypt")) {
                println("Found decryption function: " + function.getName());
                analyzeFunction(function);
            }
        }
    }
    
    private void analyzeFunction(Function function) {
        // Get references to this function
        Reference[] references = getReferencesTo(function.getEntryPoint());
        println("References to " + function.getName() + ": " + references.length);
        
        // Analyze parameters
        Parameter[] parameters = function.getParameters();
        for (Parameter param : parameters) {
            println("Parameter: " + param.getName() + " " + param.getDataType());
        }
    }
}
EOF
```

### Advanced Device-Specific Bootloader Operations

#### LG Bootloader Operations
```bash
# Enter download mode
adb reboot download

# LGUP operations
./LGUP -COM4 -PARTITION "boot" -FILE boot.bin

# Unlock bootloader
fastboot oem unlock
fastboot flashing unlock_critical

# KDZ operations
python3 kdztools.py --extract KDZ_FILE.kdz
```

#### Sony Bootloader Operations
```bash
# Enter flashmode
adb reboot bootloader
fastboot devices

# Flash with Emma
java -jar Emma.jar -r

# Flash with Flashtool
./flashtool.sh -clean-flash -device device_id -ftf firmware.ftf

# Handle TA partition
backup_ta.sh
restore_ta.sh
```

#### Motorola Bootloader Operations
```bash
# Enter bootloader
adb reboot bootloader

# Flash with RSDLite
./RSDLite --bootloader=bootloader.img --system=system.img

# Handle critical partitions
fastboot flash partition gpt.bin
fastboot flash bootloader bootloader.img
fastboot flash modem NON-HLOS.bin
```

### Advanced Memory Analysis Tools

#### Memory Dump Analysis
```python
# Create memory analyzer
cat > memory_analyzer.py << 'EOF'
#!/usr/bin/env python3
import sys
import struct
import binascii

class MemoryAnalyzer:
    def __init__(self, dump_file):
        self.dump_file = dump_file
        self.patterns = {
            'aes_key': rb'.{32}',
            'rsa_key': rb'-----BEGIN RSA PRIVATE KEY-----',
            'sqlite': rb'SQLite format 3\x00'
        }
        
    def scan_patterns(self):
        with open(self.dump_file, 'rb') as f:
            data = f.read()
            for pattern_name, pattern in self.patterns.items():
                matches = self.find_pattern(data, pattern)
                print(f"Found {len(matches)} matches for {pattern_name}")
                for match in matches:
                    print(f"Offset: 0x{match:08x}")
                    
    def find_pattern(self, data, pattern):
        matches = []
        offset = 0
        while True:
            offset = data.find(pattern, offset)
            if offset == -1:
                break
            matches.append(offset)
            offset += 1
        return matches
        
    def analyze_region(self, start_offset, size):
        with open(self.dump_file, 'rb') as f:
            f.seek(start_offset)
            data = f.read(size)
            print(f"Region analysis (0x{start_offset:08x} - 0x{start_offset+size:08x}):")
            print(f"Entropy: {self.calculate_entropy(data)}")
            print(f"Common strings: {self.find_strings(data)}")
            
    def calculate_entropy(self, data):
        # Shannon entropy calculation
        byte_count = {}
        for byte in data:
            byte_count[byte] = byte_count.get(byte, 0) + 1
        entropy = 0
        for count in byte_count.values():
            probability = count / len(data)
            entropy -= probability * math.log2(probability)
        return entropy
        
    def find_strings(self, data):
        strings = []
        current_string = ""
        for byte in data:
            if 32 <= byte <= 126:
                current_string += chr(byte)
            else:
                if len(current_string) >= 4:
                    strings.append(current_string)
                current_string = ""
        return strings

if __name__ == "__main__":
    if len(sys.argv) != 2:
        print("Usage: memory_analyzer.py <dump_file>")
        sys.exit(1)
    
    analyzer = MemoryAnalyzer(sys.argv[1])
    analyzer.scan_patterns()
EOF

chmod +x memory_analyzer.py
```

### Advanced Partition Recovery Scenarios

#### Emergency Recovery Tools
```bash
# Create recovery toolkit
cat > recovery_toolkit.sh << 'EOF'
#!/system/bin/sh

# Check partition health
check_partition() {
    local partition=$1
    e2fsck -f /dev/block/by-name/$partition
    local result=$?
    case $result in
        0) echo "Partition $partition is clean" ;;
        1) echo "Partition $partition was fixed" ;;
        2) echo "Partition $partition needs reboot" ;;
        4) echo "Partition $partition has errors" ;;
        *) echo "Partition $partition check failed" ;;
    esac
    return $result
}

# Backup partition
backup_partition() {
    local partition=$1
    local backup_file="/sdcard/backup/${partition}_$(date +%Y%m%d_%H%M%S).img"
    dd if=/dev/block/by-name/$partition of=$backup_file bs=4M
    sha256sum $backup_file > "${backup_file}.sha256"
}

# Restore partition
restore_partition() {
    local partition=$1
    local backup_file=$2
    
    # Verify checksum
    sha256sum -c "${backup_file}.sha256"
    if [ $? -ne 0 ]; then
        echo "Checksum verification failed"
        return 1
    fi
    
    # Restore
    dd if=$backup_file of=/dev/block/by-name/$partition bs=4M
}

# Fix corrupted GPT
fix_gpt() {
    sgdisk --verify /dev/block/mmcblk0
    if [ $? -ne 0 ]; then
        echo "Attempting to recover GPT..."
        sgdisk --rebuild-gpt /dev/block/mmcblk0
    fi
}
EOF

chmod +x recovery_toolkit.sh
```

### Enhanced Security Configurations

#### Security Audit Tools
```bash
# Create security auditor
cat > security_audit.sh << 'EOF'
#!/system/bin/sh

check_selinux() {
    getenforce
    sestatus
    ls -Z /data/data
}

check_permissions() {
    find /data/data -type f -perm -04000 -ls
    find /system -type f -perm -02000 -ls
}

check_processes() {
    ps -ef | grep "root"
    netstat -tulnp
}

check_certificates() {
    ls -R /system/etc/security/cacerts/
    for cert in /system/etc/security/cacerts/*; do
        openssl x509 -in $cert -text -noout
    done
}

check_kernel_parameters() {
    cat /proc/sys/kernel/kptr_restrict
    cat /proc/sys/kernel/dmesg_restrict
    cat /proc/sys/kernel/perf_event_paranoid
}

generate_report() {
    echo "Security Audit Report" > audit_report.txt
    echo "===================" >> audit_report.txt
    echo "\nSELinux Status:" >> audit_report.txt
    check_selinux >> audit_report.txt
    echo "\nSUID Files:" >> audit_report.txt
    check_permissions >> audit_report.txt
    echo "\nRoot Processes:" >> audit_report.txt
    check_processes >> audit_report.txt
    echo "\nCertificates:" >> audit_report.txt
    check_certificates >> audit_report.txt
    echo "\nKernel Parameters:" >> audit_report.txt
    check_kernel_parameters >> audit_report.txt
}

main() {
    generate_report
    echo "Audit complete. Check audit_report.txt"
}

main "$@"
EOF

chmod +x security_audit.sh
```

### Post-Recovery Setup
```bash
# Verify system stability
adb shell dumpsys battery
adb shell dumpsys cpuinfo
adb shell dumpsys meminfo

# Check for any system errors
adb logcat -b all -d > post_recovery_log.txt

# Verify critical system services
adb shell ps -A | grep system_server
adb shell service list

# Reset app preferences if needed
adb shell pm reset-permissions
```

## Additional Resources

- [Official Android Developer Documentation](https://developer.android.com/studio/command-line/adb)
- [Android Security Documentation](https://source.android.com/security)
- [Platform Tools Downloads](https://developer.android.com/studio/releases/platform-tools)

## Disclaimer

This guide is intended for educational and professional development purposes only. Users are responsible for ensuring all actions comply with relevant laws, regulations, and device policies. Some commands may void warranties or cause permanent device modifications.

---

*Last Updated: January 2025*
