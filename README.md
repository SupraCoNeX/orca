# ORCA Framework

The ORCA (Open-Source Resource Control API) framework is a toolset enabling efficient, fine-grained, and distributed resource control with off-the-shelf Linux-based WiFi transceivers.

## Components

### Quick overview

| Component | Repository | Description |
|:----------|:------------|:------------|
| ORCA-UAPI | [scnx-openwrt](https://github.com/SupraCoNeX/scnx-openwrt) | Kernel to userspace interface for Linux Kernel |
| ORCA-RCD | [orca-rcd](https://github.com/SupraCoNeX/orca-rcd) | Remote control daemon to use ORCA-UAPI remotely |
| RateMan | [scnx-rateman](https://github.com/SupraCoNeX/scnx-rateman) | Python-based package for easy-to-use resource control on multiple access points |
| MMRRS (Manual MRR Setter) | [scnx-manual-mrr-setter](https://github.com/SupraCoNeX/scnx-manual-mrr-setter) | Example application, using Rateman to perform fixed-setting resource control |

### ORCA-UAPI

The ORCA-UAPI is the core part of the ORCA framework which exposes otherwise internal information and controllable parameters to user space by leveraging `debugfs` and `relayfs` interfaces on a single device.

The source code of ORCA-UAPI consists of several patches and changes against OpenWrt Linux.   
The changes include:
- patches against mac80211
- patches against mt76 driver
- changes to Makefiles, Kconfig

It's available as part of the [scnx-openwrt](https://github.com/SupraCoNeX/scnx-openwrt) repository, which is a fork of the [upstream OpenWrt repository](https://github.com/openwrt/openwrt).

### ORCA-RCD

The purpose of the ORCA-RCD (remote control daemon) is to make the local interfaces, that are provided by ORCA-UAPI, available in a network for remote monitoring and management.

The source code is available in [orca-rcd](https://github.com/SupraCoNeX/orca-rcd). This repository is a OpenWrt-compatible package feed which contains the ORCA-RCD source code within the package directory. This feed is already included in the `feeds.conf.default` in [scnx-openwrt](https://github.com/SupraCoNeX/scnx-openwrt).

### RateMan

RateMan (Rate Manager) is a userspace utility implemented in Python that connects to ORCA-RCD of multiple nodes and provides functionality to work with multiple access points and perform centralised rate control.

The source code is available in [scnx-rateman](https://github.com/SupraCoNeX/scnx-rateman).

### MMRS

TBD

## How to get started

To have a simple setup to work with ORCA you need:
- (a) a WiFi-capable device which is supported by OpenWrt, see [here](https://openwrt.org/supported_devices). *Please note that ORCA currently only works with WiFi standards 802.11 a/b/g/n/ac.*
- (b) a WiFi client device, e.g. your smartphone
- (c) a device that can run Python packages and is performance-wise able to act as a controller

If you have a proper device, you can proceed with:

1. Build an OpenWrt image for that device (a) with the [scnx-openwrt](https://github.com/SupraCoNeX/scnx-openwrt) tree instead of the official OpenWrt tree. If you do not know how to do that, follow the official OpenWrt guide [here](https://openwrt.org/docs/guide-developer/toolchain/beginners-build-guide)

    > The ORCA-UAPI is enabled by default in mac80211 in the tree, however, ORCA-RCD needs to be properly included and selected in the OpenWrt build config.
    > Alternatively, the patches/changes can also be applied to a plain OpenWrt repository. Thus, there's no need to use our fork.
2. Flash the image to your device. The procedure depends on your device and if it's already running OpenWrt or still running the vendor's firmware. Refer to the OpenWrt device page for more information.
3. After flashing, access your device via SSH and activate/run orca-rcd. See the ORCA-RCD documentation to see how to do that.

Your device is now properly prepared for usage with ORCA.
Now head over to your controller device (c) to setup the Python-based components of ORCA. Fur further installation instructions, refer to the repository of RateMan [scnx-rateman](https://github.com/SupraCoNeX/scnx-rateman).

## If you use them in your research, please cite our software packages using the following reference.

@inproceedings{orcawifirc,
	title        = {Open-source Resource Control API for real IEEE 802.11 Networks},
	author       = {First Author and Second Author},
	year         = 2024,
	booktitle    = {The 30th Annual International Conference on Mobile Computing and Networking (ACM MobiCom '24), November 18--22, 2024, Washington D.C., DC, USA},
	location     = {Washington D.C., DC, USA},
	publisher    = {ACM},
	address      = {New York, NY, USA},
	doi          = {10.1145/3636534.3697314},
	isbn         = {979-8-4007-0489-5/24/11}
}
