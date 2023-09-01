# Alvium CSI2 driver for Xilinx ZCU106 Evaluation Kit

## Overview
This repository contains the manifest files for building the Allied Vision Alvium reference image for the Xilinx ZCU106 Evaluation Kit 

## Compatibility
This release is compatible with:
- Xilinx ZCU106 Evaluation Kit
- Alvium MIPI CSI-2 cameras with firmware 11.1 or higher
- Qt 5.15.x 

The CSI2 clock frequency is configured to 750000000 Hz.

This release supports v4l2 and GenICam for CSI2 access.
For GenICam for CSI2 access, Vimba X 2023-2 is required.


## Quick Start
### Prerequisites
-  Xilinx ZCU106 Evaluation Kit
-  Host PC: Install the requirements as defined in the [Yocto Project Reference Manual](https://docs.yoctoproject.org/4.1.4/ref-manual/system-requirements.html#required-packages-for-the-build-host)  
-  Alvium camera with Firmware 11.1
-  SD Card 8 GB

### Installation

Tip: For the prebake image, skip steps 1-4 and start with step 5.

To install the driver and layer:

1. Install the repo tool
    ```shell
    mkdir -p ~/.bin
    export PATH="${HOME}/.bin:${PATH}"
    curl https://storage.googleapis.com/git-repo-downloads/repo > ~/.bin/repo
    chmod a+rx ~/.bin/repo
    echo "export PATH=\"\${HOME}/.bin:\${PATH}\""  >> ~/.bashrc
    ```
2. Initialize project directory
    ```shell
    mkdir -p zcu106-yocto
    cd zcu106-yocto
    repo init https://github.com/alliedvision/alvium-manifest-zcu106
    repo sync
    ```
3. Prepare yocto build
    ```shell
    source setup-avt-release.sh
    ```
4. Build the AVT Image with the command:  
    ```shell
    bitbake avs-image-alvium-validation
    ```
5. Flash the image to the SD card.  If you have built the image with yocto, you can find the image here:
            <build_dir>/tmp/deploy/images/zcu106-zynqmp/avs-image-alvium-validation-zcu106-zynqmp.rootfs.wic.bz2
6. Boot the board.
7. Check if the camera firmware version is 11.1 or higher. If the camera has an earlier firmware, perform an update with Vimba X Firmware Updater.

### SDK Installation

Tip: For the prebake image, skip step 1 - 2 and start with step 3.

1. Perform steps 1 - 3 from previous instructions.
2. Build the SDK installer with the command:
    ```shell
    bitbake avs-image-alvium-validation -c populate_sdk
    ```
3. Install the SDK on your system by running the installer script. If you have built the sdk, you can find it here:
   <build_dir>/tmp/deploy/sdk/poky-glibc-x86_64-avs-image-alvium-validation-cortexa72-cortexa53-zcu106-zynqmp-toolchain-4.1.4.sh
4. If you have installed the sdk to the default location. You can setup the build enviorment by running:
   ```shell
   source /opt/poky/4.1.4/environment-setup-cortexa72-cortexa53-poky-linux 
   ```

### Board configuration
Before a camera can be used a bitstream must be loaded into the FPGA part of the ZynqMP. 
The bitstream must be loaded as root.
This is done using the following commands:
```shell 
    su
    avt-load-bitstream <bitstream name>
```
The following bitstreams are available:
- raw8: This bitstream is needed for GenICam for CSI2 streaming. The camera streams in Mono or Bayer format using v4l2.  
- yuv: The camera can stream with YUV422.
- rgb: The camera can stream with RGB24.

## Limitations
### General
- Only one fixed format per bitstream is support

### Video4Linux2 compliance
Set following v4l2-compliance test are expected to fail:
- VIDIOC_QUERY_EXT_CTRL/QUERYMENU
- VIDIOC_G/S_CTRL
- VIDIOC_(UN)SUBSCRIBE_EVENT/DQEVENT
- VIDIOC_G_FMT
- VIDIOC_TRY_FMT
- VIDIOC_S_FMT
- VIDIOC_REQBUFS/CREATE_BUFS/QUERYBUF

### Allied Vision V4L2Viewer
- The V4L2Viewer does not support all formats that the hardware supports.

### Vimba X
- When using the weston terminal, you have to manually set the GenICam TL search environment variable.
    ```shell 
    export GENICAM_GENTL64_PATH=<path to VimbaX directory>/cti
    ```


## How to start a stream

cat  /sys/bus/i2c/devices/1-003c/device_temperature

e.g. for Alvium 1500 C-500c on Port CSI1 of zcu106:

```shell
avt-load-bitstream yuv
v4l2-ctl -d /dev/v4l-subdev2 --set-ctrl exposure=20000000,gain=100,brightness=0 --set-subdev-selection top=0,left=0,width=1920,height=1080
gst-launch-1.0 v4l2src device=/dev/video3 ! video/x-raw,width=1920,height=1080,framerate=30/1,io-mode=dmabuf ! waylandsink sync=false -v
```
                