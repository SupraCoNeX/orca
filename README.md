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

The source code of ORCA-UAPI consists of several patches and changes against OpenWrt Linux.   
The changes include:
- patches against mac80211
- patches against mt76 driver

It's available as part of the [scnx-openwrt](https://github.com/SupraCoNeX/scnx-openwrt) repository, which is a fork of the [upstream OpenWrt repository](https://github.com/openwrt/openwrt).

### ORCA-RCD

TBD

### RateMan

TBD

### MMRS

TBD

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
