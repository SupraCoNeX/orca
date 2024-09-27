# ORCA-UAPI

The ORCA-UAPI enables local access to information and control parameters of kernel space rate-control from the user space. In addition to enabling monitoring of status information, the UAPI allows to execute routines to perform rate setting based on an algorithm implemented in user space. The UAPI is designed for WiFi systems running a Linux kernel in general. However, our testing only includes OpenWrt-based WiFi Systems.

Latest version of OpenWrt with the patches and changes related to ORCA-UAPI is available at
[https://github.com/SupraCoNeX/scnx-openwrt](https://github.com/SupraCoNeX/scnx-openwrt)

The UAPI provides:
 - Monitoring of transmission status through `txs`, Received Signal Strength Indicator (`rxs`), and Rate control statistics (`stats`) information of an access point.
 - Rate and TX power control through setting of appropriate MRR chain per client/station per access point.
 - Data collection and management of network measurements.

## Interface

The API provides a debugfs + relayfs based interface to pass information to and accept commands from user space. This interface consists of four files which are created for each
wifi device in `/sys/kernel/debug/ieee80211/<phy>/rc/`:
| File         | Technique | Purpose      |
|:-------------|:----------|:-------------|
| [`api_info`](#api_info---static-api-information) | debugfs | *\[read-only\]* Upon read, prints static information about the API, like rate definitions, and per-phy information, including specific hardware capabilities, virtual interfaces, etc. Output is in CSV format. |
| [`api_control`](#api_control---commands)         | debugfs | *\[write-only\]* Upon write, a supported command is parsed and executed. The available commands are listed below.|
| [`api_event`](api_event---monitoring-tasks)      | relayfs | *\[read-only\]* Continuously emits monitoring information like txs and rxs traces, depending on what has been enabled before.|
| [`api_phy`](#api_phy---phy-specific-api-info)    | debugfs | *\[read-only\]* Emits phy-specific information including driver name, ifaces, TPC capabilities, etc.|

**All API outputs and commands use a CSV-like format**

## `api_info` - static API information

Example output of `api_info` for a PHY (`/sys/kernel/debug/ieee80211/<phy>/rc/api_info`):
```
orca_version;3;0;0
#group;index;offset;type;nss;bw;gi;airtime0;airtime1;airtime2;airtime3;airtime4;airtime5;airtime6;airtime7;airtime8;airtime9
#sta;action;macaddr;iface;rc_mode;tpc_mode;overhead_mcs;overhead_legacy;update_freq;sample_freq;mcs0;mcs1;mcs2;mcs3;mcs4;mcs5;mcs6;mcs7;mcs8;mcs9;mcs10;mcs11;mcs12;mcs13;mcs14;mcs15;mcs16;mcs17;mcs18;mcs19;mcs20;mcs21;mcs22;mcs23;mcs24;mcs25;mcs26;mcs27;mcs28;mcs29;mcs30;mcs31;mcs32;mcs33;mcs34;mcs35;mcs36;mcs37;mcs38;mcs39;mcs40;mcs41
#txs;macaddr;num_frames;num_acked;probe;rate0,count0,txpwr0;rate1,count1,txpwr1;rate2,count2,txpwr2;rate3,count3,txpwr3
#rxs;macaddr;overall_signal;signal_chain0;signal_chain1;signal_chain2;signal_chain3
#stats;macaddr;rate;avg_prob;avg_tp;cur_success;cur_attempts;hist_success;hist_attempts
#best_rates;macaddr;maxtp0;maxtp1;maxtp2;maxtp3;maxprob
#sample_rates;macaddr;inc0;inc1;inc2;inc3;inc4;jump0;jump1;jump2;jump3;jump4;slow0;slow1;slow2;slow3;slow4
#sample_table;cols;rows;column0;column1;column2;column3;column4;column5;column6;column7;column8;column9
#start;iface;txs,rxs,stats,tprc_echo
#stop;iface;txs,rxs,stats,tprc_echo
#set_rates;macaddr;rate0,count0;rate1,count1;rate2,count2;rate3,count3
#set_power;macaddr;txpwr0;txpwr1;txpwr2;txpwr3
#set_rates_power;macaddr;rate0,count0,txpwr0;rate1,count1,txpwr1;rate2,count2,txpwr2;rate3,count3,txpwr3
#set_probe;macaddr;rate,count,txpwr
#rc_mode;macaddr;mode;update_freq;sample_freq
#tpc_mode;macaddr;mode
#reset_stats;macaddr
#dump_features
#set_feature;feature;state
#get;property
group;0;0;ht;1;0;0;168980;b44c0;783c0;5a260;3c1e0;2d1a0;28180;24120;;
group;1;10;ht;2;0;0;b44c0;5a260;3c1e0;2d1a0;1e170;16950;14140;12110;;
group;2;20;ht;3;0;0;783d0;3c1e8;28198;1e170;14148;f130;d5d8;c060;;
group;3;30;ht;4;0;0;5a260;2d1a8;1e170;16950;f130;b4a8;a120;9088;;
group;4;40;ht;1;0;1;1448c0;a2460;6c3a0;51240;361e0;289a0;241a0;207a0;;
group;5;50;ht;2;0;1;a2470;51250;361e0;289b0;1b170;14560;12150;10450;;
group;6;60;ht;3;0;1;6c3a0;361e8;241a0;1b178;12158;d948;c0a8;ad50;;
group;7;70;ht;4;0;1;51250;289b0;1b178;14560;d948;a2c8;9130;8240;;
group;8;80;ht;1;1;0;ada50;56da0;39ec0;2b750;1cfd0;15ba0;13590;11650;;
group;9;90;ht;2;1;0;56da0;2b750;1cfd8;15ba8;e868;add0;9b40;8ba0;;
group;a;a0;ht;3;1;0;39ec0;1cfdc;13590;e86c;9b44;7434;6784;5cc4;;
group;b;b0;ht;4;1;0;2b750;15ba8;e86c;add4;7434;56e8;4e20;4650;;
group;c;c0;ht;1;1;1;9c4a0;4e2e0;34240;271f0;1a1a0;13910;116c0;faa0;;
group;d;d0;ht;2;1;1;4e2e0;271f8;1a1a8;13910;d160;9ca0;8bf0;7de0;;
group;e;e0;ht;3;1;1;34244;1a1ac;116cc;d160;8bf0;68c8;5d5c;53b0;;
group;f;f0;ht;4;1;1;271f8;13914;d160;9ca4;68c8;4e68;4680;3f78;;
group;10;100;cck;1;0;0;960e00;4c9100;1dcc00;106f00;949700;4b1a00;1c5500;ef800;;
group;11;110;ofdm;1;0;0;190640;10d880;cc1a0;8aac0;6a720;493e0;399e0;33c20;;
group;12;120;vht;1;0;0;168980;b44c0;783c0;5a260;3c1e0;2d1a0;28180;24120;1e160;1b180
group;13;130;vht;2;0;0;b44c0;5a260;3c1e0;2d1a0;1e170;16950;14140;12110;f130;d8c0
group;14;140;vht;3;0;0;783d0;3c1e8;28198;1e170;14148;f130;d5d8;c060;a120;9088
group;15;150;vht;4;0;0;5a260;2d1a8;1e170;16950;f130;b4a8;a120;9088;7918;6c60
group;16;160;vht;1;0;1;1448c0;a2460;6c3a0;51240;361e0;289a0;241a0;207a0;1b160;18660
group;17;170;vht;2;0;1;a2470;51250;361e0;289b0;1b170;14560;12150;10450;d940;c350
group;18;180;vht;3;0;1;6c3a0;361e8;241a0;1b178;12158;d948;c0a8;ad50;9130;8240
group;19;190;vht;4;0;1;51250;289b0;1b178;14560;d948;a2c8;9130;8240;6d28;61c0
group;1a;1a0;vht;1;1;0;ada50;56da0;39ec0;2b750;1cfd0;15ba0;13590;11650;e860;d0f0
group;1b;1b0;vht;2;1;0;56da0;2b750;1cfd8;15ba8;e868;add0;9b40;8ba0;7430;6878
group;1c;1c0;vht;3;1;0;39ec0;1cfdc;13590;e86c;9b44;7434;6784;5cc4;4e20;4650
group;1d;1d0;vht;4;1;0;2b750;15ba8;e86c;add4;7434;56e8;4e20;4650;3a98;34bc
group;1e;1e0;vht;1;1;1;9c4a0;4e2e0;34240;271f0;1a1a0;13910;116c0;faa0;d160;bc40
group;1f;1f0;vht;2;1;1;4e2e0;271f8;1a1a8;13910;d160;9ca0;8bf0;7de0;68c8;5e38
group;20;200;vht;3;1;1;34244;1a1ac;116cc;d160;8bf0;68c8;5d5c;53b0;4680;3f78
group;21;210;vht;4;1;1;271f8;13914;d160;9ca4;68c8;4e68;4680;3f78;34ec;2fa8
group;22;220;vht;1;2;0;50238;28198;1abb8;14148;d5d8;a120;8e90;80e8;6b68;60a8
group;23;230;vht;2;2;0;28198;14148;d5dc;a120;6b6c;510c;4748;4074;35b4;30d4
group;24;240;vht;3;2;0;1abbc;d5de;8e94;6b6c;474a;35b6;2fda;2af8;2422;203a
group;25;250;vht;4;2;0;1414a;a122;6b6c;510e;35b6;2904;2422;203a;1b58;186a
group;26;260;vht;1;2;1;48230;241a0;18128;12158;c0a8;9130;8080;7430;60e0;5730
group;27;270;vht;2;2;1;241a0;12158;c0ac;9134;60e0;4924;4058;3a34;3088;2c24
group;28;280;vht;3;2;1;18128;c0ac;8084;60e0;405a;3088;2b42;26de;20b6;1d32
group;29;290;vht;4;2;1;1215a;9136;60e0;4924;3088;251c;20b6;1d32;18ce;162a
sample_table;a;a;9,1,5,8,6,4,7,0,3,2;8,2,7,9,4,1,0,3,5,6;8,2,9,0,3,4,7,5,6,1;9,1,7,0,8,4,2,3,5,6;0,2,5,6,4,7,8,3,1,9;1,6,9,4,2,7,8,3,0,5;9,5,3,0,1,4,6,2,7,8;9,5,2,8,4,1,0,3,7,6;3,1,5,7,8,9,0,4,6,2;8,6,1,3,7,4,0,5,2,9
```

**Lines starting with `#` are format lines. They describe the exact format of all output lines and commands.**
Minstrel-HT internally uses a rate representation that differs from the default MCS index + further parameters by separating all rates into groups with continuous indices.
The output of `api_info` is mostly the same for all PHYs, especially the format lines and all lines starting with `group`. These `group` lines expose the rate representation that Minstrel-HT uses internally and is fixed at compile-time.

## `api_control` - Commands

The API currently provides the following commands:
* [`dump`](#dump)
* [`start`](#start)
* [`stop`](#stop)
* [`rc_mode`](#rc_mode)
* [`tpc_mode`](#tpc_mode)
* [`set_rates`](#set_rates)
* [`set_power`](#set_power)
* [`set_rates_power`](#set_rates_power)
* [`set_probe`](#set_probe)
* [`reset_stats`](#reset_stats)
* [`dump_features`](#dump_features)
* [`set_feature`](#set_feature)
* [`get`](#get)

The commands are described in detail in the following subsections.

### `dump`
	
Print out the supported data rate set for each client already connected - useful to separate tx_status packets that are supported by minstrel.   
		 
**Full syntax:** `dump`   
*No parameters.*
	
---
### `start`
Enable one or multiple monitoring mode. Currently this command accepts mode `txs`, `rxs`, `stats`, `sta` and `tprc_echo`. See below for explanation of each.   
		 
**Full syntax:** `start;<mode0>;<mode1>;...;<modeX>` (multiple mode can be chained)   
**Parameters:**   
| | | |
|:--|:--|:--|
|`modeX`| `string` | A supported monitoring mode, i.e. `txs`, `rxs`, `stats` or `tprc_echo`. See section *Monitoring modes* below for explanation of each. |   
		 
**Example:** `start;txs;rxs;tprc_echo`
	
---
### `stop`  
Disable one or multiple monitoring mode.   
		 
**Full syntax**: `start;<mode0>;<mode1>;...;<modeX>` (multiple mode can be chained)   
**Parameters:**   
| | | |
|:--|:--|:--|
|`modeX`| A supported monitoring mode, i.e. `txs`, `rxs`, `stats` or `tprc_echo`. See section *Monitoring modes* below for explanation of each. |
		 
**Example:** `stop;txs;rxs;tprc_echo`
	
---
### `rc_mode`   
Sets the Rate control operation mode for one or all connected STAs.  
		 
**Full syntax:** `rc_mode;<macaddr>;<mode>;<update_freq>;<sample_freq>`   
**Parameters:**
| | | |
|:--|:--|:--|
|`macaddr`| The MAC address of the STA for which the rc_mode should be set, e.g. `aa:bb:cc:dd:ee:ff`, or `all` to perform the action for all currently connected STAs. |
|`mode`   | Either `auto` or `manual`. |
|         | In `auto` mode, kernel-space Minstrel-HT rate control is running and performing rate selection. |
|         | In `manual` mode, kernel-space Minstrel-HT is turned off and statistics and rate selection can be performed via the API. |
| `update_freq` | (optional) The frequency with which Minstrel-HT updates its statistics in Hz, specified as HEX. |
| `sample_freq` | (optional) The frequency with which Minstrel-HT samples rates in Hz, specified as HEX. |
		 
**Example:** `rc_mode;aa:bb:cc:dd:ee:ff;manual;20;50` or `rc_mode;all;auto`   

---
### `tpc_mode`   
Set the Transmit power control operation mode for one or all connected STAs.  
		 
**Full syntax:** `tpc_mode;<macaddr>;<mode>`   
**Parameters:**
| | |
|:--|:--|
|`macaddr`| The MAC address of the STA for which the tpc_mode should be set, e.g. `aa:bb:cc:dd:ee:ff`, or `all` to perform the action for all currently connected STAs. |
|`mode`   | Either `auto` or `manual`. |
|         | In `auto` mode, kernel-space Minstrel-HT rate control is running and passing `-1` to the driver indicating that the default power should be used |
|         | In `manual` mode, kernel-space Minstrel-HT is turned off and statistics and tx power selection can be performed via the API. |
		 
**Example:** `rc_mode;aa:bb:cc:dd:ee:ff;manual` or `rc_mode;all;auto`   

---
### `set_rates`   
Set the rate table of the given STA with the given rates and counts while leaving TX power untouched.   
		 
**Full syntax:** `set_rates;<macaddr>;<stage0>;<stage1>;<stage2>;<stage3>`   
**Parameters:**   
| | |
|:--|:--|
| `macaddr` | The MAC address of the STA whose rate table should be modified, e.g. `aa:bb:cc:dd:ee:ff`. |
| `stageX`  | Comma-separated 2-tuple with transmission rate and try count, both values in HEX format, e.g. `d7,4`. The TX rate is an index corresponding to the rate representation that Minstrel-HT uses internally. It is not an MCS index! |
|           | Maximum number of stages is currently 4, minimum is 1. Unused stages can be omitted and will be set to `rate = -1` and `count = 0` in the rate table. |
		 
**Example:** `set_rates;aa:bb:cc:dd:ee:ff;d7,4;d2,4;c1,4` 

---
### `set_power`   
Set the rate table of the given STA with the given TX power while leaving rates and counts untouched.   
		 
**Full syntax:** `set_power;<macaddr>;<txpwr0>;<txpwr1>;<txpwr2>;<txpwr3>`   
**Parameters:**   
| | |
|:--|:--|
| `macaddr` | The MAC address of the STA whose rate table should be modified, e.g. `aa:bb:cc:dd:ee:ff`. |
| `txpwrX`  | The TX power index in HEX format, e.g. `a`. The TX power is an index corresponding to the TX power ranges which the driver defines in its TPC capabilities. It is not an absolute power value in dBm! |
|           | Maximum number of TX power entries that can be specified is currently 4, minimum is 1. However, the maximum depends on the capabilities of the WiFi hardware. Unused entries can be omitted and will be set to `-1` in the rate table. |
		 
**Example:** `set_power;aa:bb:cc:dd:ee:ff;a;c;1f` 

---
### `set_rates_power`   
Set the rate table of the given STA with the given rates, counts and tx power.   
		 
**Full syntax:** `set_rates_power;<macaddr>;<stage0>;<stage1>;<stage2>;<stage3>`   
**Parameters:**   
| | |
|:--|:--|
| `macaddr` | The MAC address of the STA whose rate table should be modified, e.g. `aa:bb:cc:dd:ee:ff`. |
| `stageX`  | Comma-separated 3-tuple with transmission rate, try count and tx power index. All three values must be given in HEX format, e.g. `d7,4,a`. The TX rate is an index corresponding to the rate representation that Minstrel-HT uses internally. It is not an MCS index! |
|           | The TX power is an index corresponding to the TX power ranges which the driver defines in its TPC capabilities. It is not an absolute power value in dBm! |
|           | Maximum number of stages is currently 4, minimum is 1. Unused stages can be omitted and will be set to `rate = -1` and `count = 0` in the rate table. |
	 
**Example:** `set_rates_power;aa:bb:cc:dd:ee:ff;d7,4,a;d2,4,c;c1,4,1f` 

---
### `set_probe`   
Set the given transmission parameters (rate, count, tx power) to be probed for the given STA.   
		 
**Full syntax:** `set_probe;<macaddr>;<rate>,<count>,<txpwr>`   
**Parameters:**   
| | |
|:--|:--|
| `macaddr` | The MAC address of the STA whose rate table should be modified, e.g. `aa:bb:cc:dd:ee:ff`. |
| `rate`    | The TX rate to be probed. The TX rate is an index corresponding to the rate representation that Minstrel-HT uses internally. It is not an MCS index! |
| `count`   | How often the probing rate should be tried at most. |
| `txpwr`   | The TX power index in HEX format, e.g. `a`. The TX power is an index corresponding to the TX power ranges which the driver defines in its TPC capabilities. It is not an absolute power value in dBm! |
		 
**Example:** `set_probe;aa:bb:cc:dd:ee:ff;d7,4,a`   
		 
---
### `reset_stats`
Reset the Minstrel-HT statistics for one or all connected STAs.   
		 
**Full syntax:** `reset_stats;<macaddr>`   
**Parameters:**   
| | |
|:--|:--|
| `macaddr` | The MAC address of the STA for which the statistics should be reset, e.g. `aa:bb:cc:dd:ee:ff`, or `all` to perform the action for all currently connected STAs. |

**Example:** `reset_stats;aa:bb:cc:dd:ee:ff`

---
### `dump_features`
Dumps all supported features and their current states as a 'ftrs' line. This can be used at runtime to monitor the states of features without having access to `api_phy` (e.g. via `orca_rcd`).

**Full syntax:** `dump_features`   
**Parameters:** `none`

**Example:** `dump_features`   
**Result:** `179bcbed680aefba;ftrs;4;adaptive_sens,1;tpc,0;pwr-user,f;force-rr,0`

---
### `set_feature`
Set the state of a feature, e.g. enable/disable TPC, enable/disable adaptive sensitivity, etc.

**Full syntax:** `set_feature;<feature>;<state>`   
**Parameters:**
| | |
|:--|:--|
| `feature` | The identifier of a feature, e.g. adaptive_sens, tpc, force-rr, etc.
| `state` | The state to be set for the feature. This must be an integer given in HEX. The meaning of the state depends on the feature, however on/off features like TPC typically use 0 for off and 1 for on. |

**Example:** `set_feature;adaptive_sens;1`

---
### `get`
Get the value of the specified property. This can be used the query values/properties that are not available or only once in the beginning (as it is the case with `orca_rcd`).
The result is issued as a `got` line (see example below).

**Full syntax:** `get;<property>`   
**Parameters:**
| | |
|:--|:--|
| `property` | One of the supported properties to be read. Currently, only `pwr-limit` (to read the power limit via `get_txpower`) is supported. |

**Example:** `get;pwr-limit`   
**Result:** `179bcc3aeae5e192;got;pwr-limit;1e`

---

### Command echoing

The commands `start`, `stop`, `rc_mode`, `tpc_mode` and `reset_stats` are always echoed after execution. E.g., after having successfully executed the command
```
rc_mode;aa:bb:cc:dd:ee:ff;manual
```
a corresponding echo line will appear in `api_event` output:
```
16c4added930f1b4;rc_mode;aa:bb:cc:dd:ee:ff;manual
```

This feature is provided to ensure:
* the mentioned commands do not have to be monitored separately and appear in the `api_event` log for full consistency and traceability
* in case multiple clients capture the output of `api_event`, all clients know that one of the clients issued a command

For the command `set_rates`, `set_power`, `set_rates_power` and `set_probe` echoing is not enabled by default but can be turned on, e.g. for debugging purposes. 
It can be enabled - analoguous to the monitoring modes - with the commands `start` and `stop` using the identifier `tprc_echo`, e.g. `start;tprc_echo`.   
The commands `dump`, `dump_features` and `get` are not echoed as they always produce some kind of output.

## `api_event` 

`api_event` is the exported file in the debugfs, backed by relayfs, that is used by ORCA UAPI to relay event information to clients in userspace. Most of the events can be enabled/disabled via monitoring modes, however, some kinds of events are always printed. 

### Station events

Beside the aforementioned command echoing, this applies also to the following:
- station adding
- station dump
- station capabilities update
- station removal
*Station dumping has to be specifically requested with the `dump` command.*

In case of these events, station action lines are emitted. The format syntax is as follows:
```
<timestamp>;sta;<action>;<macaddr>;<iface>;<rc_mode>;<tpc_mode>;<overhead>;<overhead_legacy>;<update_interval>;<sampling_interval>;<supported_rates>
```

| Field			| Description |
|:----------------------|:------------|
| `timestamp`		| Timestamp for system time (Unix epoch time) in nanoseconds in hex format. |
| `action`		| Can be `add`, `dump`, `update` or `remove`. |
| `macaddr`		| MAC address of the affected station. |
| `iface`		| Name of the interface this station is connected to. |
| `rc_mode`		| Current rc_mode of the station. |
| `tpc_mode`		| Current tpc_mode of the station. |
| `overhead`		| **TBD**
| `overhead_legacy`	| **TBD**
| `update_interval`	| Rate selection update interval. |
| `sampling_interval`	| Rate sampling interval. |
| `supported_rates`	| Bitmap of supported MCS rates in Minstrel group encoding. |

e.g.
```
wl2;0;sta;add;aa:bb:cc:dd:ee:ff;wl2-ap0;auto;auto;6c;3c;14;32;0;0;0;0;0;0;0;0;0;0;0;0;0;0;0;0;0;0;1ff;1ff;0;0;1ff;1ff;0;0;3ff;3ff;0;0;3ff;3ff;0;0;3ff;3ff;0;0;3ff;3ff;0;0
```

> TODO: explain example

### Monitoring events

For monitoring the status and events of a PHY, one or multiple monitoring mode must be activated using the aforementioned `start` command. The available monitoring mode are as followed with a short description. The format of each trace line that is emitted is explained below the table.

| Monitoring mode | Description |
|:----------------|:------------|
| `txs` | Monitor TX status events. Prints a `txs` trace line for each acknowledged transmitted frame. |
| `rxs` | Monitor RX status events. Prints a `rxs` trace line for each received frame. |
| `stats` | Monitor Minstrel-HT statistics. Prints `stats` and `best_rates` lines regularly after a fixed interval. |
| `tprc_echo` | Also echo the commands `set_rates`, `set_power`, `set_rates_power` and `set_probe`, analogous to the other commands. This is disabled by default due to possible performance issues caused by echoing these typically frequently used commands. |

### Monitoring information format
Once the monitoring modes have been triggered as mentioned above, the trace lines are printed to the corresponding `api_event` endpoint. Format of these lines is as follows:

#### Format of trace for `txs` information
```
<timestamp>;txs;<macaddr>;<num_frames>;<num_acked>;<probe>;<rate0>,<count0>,<txpwr0>;<rate1>,<count1>,<txpwr1>;<rate2>,<count2>,<txpwr2>;<rate3>,<count3>,<txpwr3>
```

| Field                  | Description |
|:-----------------------|:------------|
| `timestamp`            | Timestamp for system time (Unix epoch time) in nanoseconds in hex format. |
| `txs`                  | Denotes that the traces is for TX status. |
| `macaddr`              | MAC address of the station/client for which trace is received. |
| `num_frames`           | Number of data packets in a given TX frame. |
| `num_acked`            | Number of data packets of a frame which were successfully transmitted for which an `ACK` has been received. |
| `probe`                | Binary index for type of frame. If `probe` = 1 for probing frame, 0 otherwise. |
| `rate0,count0,txpwr0`  | 1st MCS rate (`rate0`) chosen for probing or data frame with `count0` attempts/tries and `txpwr0` transmit power index. |
| `rate1,count1,txpwr1`  | 2nd MCS rate (`rate1`) chosen for data frame with `count1` attempts/tries and `txpwr1` transmit power index. |
| `rate2,count2,txpwr2`  | 3rd MCS rate (`rate2`) chosen for data frame with `count2` attempts/tries and `txpwr2` transmit power index. |
| `rate3,count3,txpwr3`  | 4th MCS rate (`rate3`) chosen for data frame with `count3` attempts/tries and `txpwr3` transmit power index. |

_Note: In the rate table containing upto four rates, corresponding counts and tx powers, if a sequential rate-count-txpwr is not used, the rate, count and tx-power fields are empty._

E.g. 1. Successful transmission on 1st MCS rate
```
16c4added930f1b4;txs;cc:32:e5:9d:ab:58;3;3;0;d7,1,28;,,;,,;,,
```
Here we have a trace at timestamp, `1626196159.112026795`, for client with the MAC address of `cc:32:e5:9d:ab:58`, with `num_frames = 3`, `num_acked = 3`, `probe = 0` denotes that it was not a probing frame, index of 1st MCS rate tried `rate0` is `d7`, number of transmission tries for `rate0` was `count0 = 1` and a tx-power idx of `28` was used. In this case only one MCS rate was tried and successfully used. 

E.g. 2. Successful transmission on 2nd MCS rate 
```
16c4added930f1b4;txs;d4:a3:3d:5f:76:4a;1;1;1;266,2,1f;272,1,21;,,;,,
```
Here we have a trace at timestamp `1626189830.926593008` for a client with the MAC address `d4:a3:3d:5f:76:4a`, with `num_frames = 1`, `num_acked = 1`, `probe = 1` denotes that it was a probing frame, index of 1st MCS rate tried `rate0` is `266`, number of transmission tries for `rate0` was `count0 = 2` and a tx-power index of `1f` was used. In this case the `rate0` was not successful and hence a 2nd MCS rate with index `rate1` of `272` was tried `count1 = 1` times with a tx-pwoer index of `21` and this transmission was successful.

E.g. 3. Erroneous `txs` trace
```
16c4added930f1b4;txs;86:f9:1e:47:68:da;2;0;0;,,;,,;,,;,,
```
In this case, the trace implies that no MCS rate has been tried.

#### How to read the `rateX` fields
Consider again the example from the previous section:
```
16c4added930f1b4;txs;d4:a3:3d:5f:76:4a;1;1;1;266,2,1f;272,1,21;,,;,,
```
The first digits of `rateX` tell us in which rate group to look. The rightmost digit from the rate entries gives us the group offset. *Note, that these are hex digits!*
In our example, rate `266` refers to the `6`th rate from group `26` and `272` refers to the `2`nd rate from group `27`. Looking at the `group` output mentioned above, we can find the exact rates. What the API is telling us is that we first tried to send a frame at rate `266` twice before falling back to rate `272` where transmission succeeded after one attempt.

#### How to read the `txpwrX` fields

Keep in mind that the values are not absolute values in dBm, and the values are in HEX. The range and meaning of the values depends on what the driver defines for a WiFi device. Thus, they should be considered as abstract values. For example, ath9k defines power levels from 0 to 63, idx 0 corresponds to 0 dBm and the power levels have a value-distance of 0.5 dBm. Information about the power levels / ranges is exposed by `api_phy` and also given by Minstrel-RCD upon connecting, see [below](#api_phy---phy-specific-api-info) for a detailed explanation.

---
#### Format of `rxs` trace
```
<timestamp>;rxs;<macaddr>;<overall_signal>;<signal_chain1>;<signal_chain2>;<signal_chain3>;<signal_chain4>
```

| Field              | Description |
|:-------------------|:------------|
| `<timestamp>`      | Timestamp for system time (Unix epoch time) in nanoseconds in hex format. |
| `rxs`              | Denotes that the trace contains an RX status for a received frame. |
| `<macaddr>`        | MAC address of the station/client for which trace is received.|
| `<overall_signal>` | (signed 8-bit) The overall signal strength of the received frame. |
| `<signal_chainX>`  | (signed 8-bit) The signal strengths of the single antennas/RX chains. This depends on what the driver delivers, otherwise the corresponding field has the value `7f`. |

E.g.
```
phy1;17b67123fc065a5e;rxs;52:4a:6f:f3:c4:95;d3;ce;d1;7f;7f
```
Received a frame from a station with an overall signal strength of `-45` dBm (`d3`) and signal strengths of `-50` dBm (`ce`) on chain 1 and `-47` dBm (`d1`) on chain 2. Chain 3 and 4 are not delivered since the WiFi chip only has 2 antennas/RX chains.

---
#### Format of `stats` trace
```
<timestamp>;stats;<macaddr>;<rate>;<avg_prob>;<avg_tp>;<cur_success>;<cur_attempts>;<hist_success>;<hist_attempts>
```

| Field             | Description |
|:------------------|:------------|
| `<timestamp>`     | Timestamp for system time (Unix epoch time) in nanoseconds in hex format. |
| `stats`           | Denotes that the trace signals a Rate Control Statistics update, in particular an update of the statistics for one rate.|
| `<macaddr>`       | MAC address of the station/client for which trace is received.|
| `<rate>`          | The idx of the rate whose statistics were updated.|
| `<avg_prob>`      | Average success probability of the rate. |
| `<avg_tp>`        | Average estimated throughput of the rate. |
| `<cur_success>`   | Number of successes in the current interval. |
| `<cur_attempts>`  | Number of attempts in the current interval. |
| `<hist_success>`  | Number of successes in the last interval. |
| `<hist_attempts>` | Number of attempts in the last interval. |

E.g.
```
phy1;17503da1e84dea50;stats;04:f0:21:26:d9:25;c4;3e8;1a2;1;1;3f9;400
```

> TODO: Explain example

---
#### Format of `best_rates` trace

```
phy1;<timestamp>;best_rates;<macaddr>;<maxtp0>;<maxtp1>;<maxtp2>;<maxtp3>;<maxprob>
```

|Field|Description|
|:------|:----------|
|`best_rates`| Denotes that the trace contains current best rate selection.|
|`<macaddr>`| MAC address of the station/client for which trace is received.|
|`<maxtp0>`| Rate with highest estimated throughput. |
|`<maxtp1>`| Rate with second highest estimated throughput. |
|`<maxtp2>`| Rate with third highest estimated throughput. |
|`<maxtp3>`| Rate with fourth highest estimated throughput. |
|`<maxprob>`| Rate with highest success probability. |

phy1;17503da1e84ec73d;best_rates;04:f0:21:26:d9:25;94;93;c4;92;c4

> TODO: Explain example

## `api_phy` - PHY-specific API info

PHY-specific information is exposed separately so clients only need to read `api_info` once and can retrieve the much smaller PHY-specific info from `api_phy` for each PHY. This file uses the format as follows:
```
<type>;<info>
```
`<type>` denotes the type of information. Currently, the following types are used:
| Type  | Information content |
|:------|:--------------------|
| `drv` | The driver name that is loaded for the current PHY. |
| `if`  | Virtual interfaces associated with the current PHY. |
| `tpc` | Transmit power control capabilities announced by the driver. |
| `pwr_limit` | The current power limit [**unit: half-dBm**] that is configured for the radio. This power limit includes all limiting factors, e.g. user power limit, calibration, regulatory limits. |
| `mon` | Currently active monitor modes. |
| `ftrs` | A list of supported features and their current states. |
| `sta`(*) | None, one or multiple lines for each currently associated STA. The format is equal to the STA lines produced by `dump` or `start` **EXCEPT** for the `action``add` and the timestamp.

An example output of `api_phy` may be:
```
drv;ath9k
if;wlan2
tpc;mrr;1;0,40,0,2
pwr_limit;30
mon;txs
ftrs;3;adaptive_sens,1;tpc,0;pwr-user,18
```

### Content format of `tpc`
The content of the `tpc` has the following syntax:
```
tpc;<tpc_type>;<tpc_ranges>;<tpc_range_block0>;<tpc_range_block_1>
```

| Parameter            | Description |
|:---------------------|:------------|
| `<tpc_type>`         | `not`, `pkt` or `mrr`. Denotes the type of TPC support, respectively 'no support', 'tpc per packet' or 'tpc per mrr stage'.|
| `<tpc_ranges>`       | Number of following TPC range blocks.|
| `<tpc_range_blockX>` | One or multiple range blocks describing the different TPC power ranges that are supported. A range consists of a comma-separated 4-tuple with the values in order `start_idx`, `n_levels`, `start_pwr` and `pwr_step` (all HEX). Example: `0,40,0,2` describes a power range starting at idx 0 with 64 levels, the power level at idx 0 corresponds to (0 * 0.25) dBm and the range has a step width of (2 * 0.25 = 0.5) dBm. |

### Content format of `ftrs`
The content of the `ftrs` has the following syntax:
```
ftrs;<num_of_ftrs>;<ftr0_id>,<ftr0_state>;<ftr1_id>,<ftr1_state>;...
```

| Parameter            | Description |
|:---------------------|:------------|
| `<num_of_ftrs>`      | Number of supported features and also number of following blocks a parser has to expect. |
| `<ftrX_id>`          | Identifier of the feature, e.g. `tpc`, `adaptive_sens`, etc.|
| `<ftrX_state>`       | The current state of the feature given as integer in HEX format. |

### Available features
ORCA provides an abstract interface to set several features/functions through the ORCA UAPI. Drivers can announce support for several of the currently supported features:

| Identifier           | Description |
|:---------------------|:------------|
| `adaptive_sens`      | Adaptive sensitivity. Collective term for techniques/algorithms running in driver/firmware to continuously adapt the sensitivity/noise floor level to the environment. |
| `tpc`                | Fine-grained Transmit Power Control (TPC). |
| `force-rr`           | Force-Rate-Retry. mt7615-specific feature to prevent the driver/firmware from adjusting the MRR chain on its own. |
| `pwr-ack`            | TX power for ACK frames. [**unit: half-dBm**] |
| `pwr-rts`            | TX power for RTS/CTS frames. [**unit: half-dBm**] |
| `pwr-chirp`          | TX power for Chirp frames. [**unit: half-dBm**] |
| `pwr-rpt`            | TX power for Rpt frames. [**unit: half-dBm**] |
| `pwr-user`           | TX power limit that was set by the user (e.g. via config) [**unit: dBm**]. |

## Typical workflow: Set MRR chain with rates, counts and tx power + validation

Perform the following steps to set an MRR chain:
	
1. Enable txs monitoring for a given STA using the command:
```
start;txs
```
This command enables a continuous stream of TX status traces to be able to immediately validate the MRR setting.

2. Enable TPC feature
```
set_feature;tpc;1
```
This command enables the TPC feature for a PHY. This step is only necessary in case it is turned of by default.

3. Enable manual MRR setting using the command:
```
rc_mode;<macaddr>;manual
tpc_mode;<macaddr>;manual
```
This command disables the default kernel-space Minstrel-HT rate control algorithm and the current TX power control behaviour. `tpc_mode` can be omitted in case `set_rates` is used to only set rates and counts.

4. Set desired MRR chain using the command:
```
set_rates_power;<macaddr>;<stage0>;<stage1>;<stage2>;<stage3>
```    
See the command description [above](#api_control---commands) for the detailed format of the command.

## Minstrel-HT Rate Control Statistics

![alt tag](https://user-images.githubusercontent.com/79704080/112141900-385fd980-8bd6-11eb-99a2-5c18ff8e37e5.PNG)

> TODO: Add description of variables in table

## Feature-Control interface

To support querying and controlling several features of WiFi chipsets, a new, currently out-of-tree interface was introduced and implemented. This currently covers the following features:
- RTS/CTS, ACK, etc. tx power setting on ath9k
- enable/disable TPC
- set user TX power limit
- mt7615-only force-rate-retry
- enable/disable adaptive sensitivity (ath9k calls this ANI, mt76 calls this SCS or dynamic sensitivity)

While some of the features also may have been controlled via existing interfaces/structures (e.g. `ieee80211_conf`), not all demands could be satisfied with the existing interfaces. Thus, we decided to implement a new interface. Most notably, other interfaces do not support a proper direct/immediate access to the driver's features.

The implementation currently consists of the following patches:
- `package/kernel/mac80211/patches/subsys`
	- `8900-mac80211-add-unified-control-for-adaptive-sensitivity.patch`
   	- `9007-mac80211-add-further-control-knobs-in-feature-control-interface.patch`
- `package/kernel/mac80211/patches/ath9k`
  	- `8900-ath9k_support-setting-adaptive-sensitivity-through-mac80211.patch`
  	- `9008-ath9k_add-support-for-further-control-knobs-via-feature-ctrl.patch`
- `package/kernel/mt76/patches`
  	- `200-mt7615-add-control-of-adaptive-sensitivity-through-mac80211.patch`
  	- `201-mt7603-add-control-of-adaptive-sensitivity-through-mac80211.patch`
  	- `202-mt7615-add-support-for-further-control-knobs-through-feature-ctrl.patch`

In addition to that, several patches depend on these patches and ORCA comes with built-in support for this interface.

## Transmit Power Control in the Linux kernel

A few WiFi chipsets have extended control capabilities for the transmission power. In detail, this is the case for the Atheros 802.11 a/b/g/n AR9xxx and MT7615 chipsets we were focusing on. They allow fine-grained TPC (transmit power control) per-frame, however, the mac80211 subsystem doesn't provide any support or infrastructure for that. Thus, we also introduced a patchset alongside ORCA-UAPI that adds support for this to mac80211 and ath9k and mt76 drivers.   
This current patchset consists of:
- `package/kernel/mac80211/patches/subsys`
 	- `9000-mac80211-modify-tx-power-level-annotation.patch`
   	- `9001-mac80211-add-tx-power-annotation-in-control-path.patch`
   	- `9002-mac80211-add-hardware-flags-for-TPC-support.patch`
   	- `9003-mac80211-add-utility-function-for-tx_rate-rate_info-conversion.patch`
   	- `9004-mac80211-minstrel_ht-add-proper-handling-of-tx-power-control.patch`
   	- `9005-mac80211-add-tpc-feature-setting-via-mac80211.patch` *
   	- `9006-mac80211-add-ctrl-frame-tpc-setting-via-mac80211.patch` *
- `package/kernel/mac80211/patches/ath9k`
	- `9000-ath9k_register-hw-with-tx-power-levels.patch`
   	- `9001-ath9k_include-tx-power-in-tx-control-path.patch`
   	- `9002-ath9k_report-back-tx-power-in-status-path.patch`
   	- `9004-ath9k_export-max-power-per-ratetbl-to-debugfs.patch`
   	- `9005-ath9k_allow-enabling-disabling-tpc-via-mac80211-feature-interface.patch` *
   	- `9006-ath9k_allow-get-txpower-without-vif.patch`
   	- `9007-ath9k_add-support-for-control-frame-tpc.patch` *
- `package/kernel/mt76/patches`
  	- `9000-mt7615-add-TPC-control-infrastructure.patch`
  	- `9005-mt7615-enable-disable-TPC-via-feature-control-interface.patch` *
  	- `9010-mt7615-report-tx-power-in-tx-status.patch`

> Patches marked with * depend on the out-of-tree feature-control interface introduced alongside with ORCA!

The patchset introduces an abstraction of the TPC capabilities of different WiFi chipsets:
```
struct ieee80211_hw_txpower_range {
	u8 start_idx;
	u8 n_levels;
	s8 start_pwr;
	s8 pwr_step;
};
```
Valid tx-power indices are always `>= 0` per definition. However, a value of `-1` can be specified to pass the decision about the tx-power to use to the driver. For example, ath9k uses the maximum allowed tx-power when `-1` is specified.   
This structure has to be used by drivers to specify their supported transmit power levels by defining one or multiple ranges which are **discrete, sequentially indexed sets of power levels with a fixed value-distance**. Each driver aiming to support this has to provide this and additional information during PHY registration at the mac80211 subsystem.

Furthermore, the existing rate control infrastructure is extended to include the transmit power:
```diff
 struct ieee80211_sta_rates {
 	struct rcu_head rcu_head;
	struct {
		s8 idx;
 		u8 count_cts;
 		u8 count_rts;
 		u16 flags;
+		s16 txpower_idx;
 	} rate[IEEE80211_TX_RATE_TABLE_SIZE];
 };
```
The TX power is then annotated in this per-STA rate table as an index into the list of power levels that is defined by the aforementioned TX power ranges.

> Corresponding changes for the TX status path in mac80211 are NOT included in this patchset. They are already introduced in Linux kernel as per commit [`44fa75f207`](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=44fa75f207d8a106bc75e6230db61e961fdbf8a8).
