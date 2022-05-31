[![GitHub Release](https://img.shields.io/github/release/Vaskivskyi/ha-asusrouter.svg?style=for-the-badge&color=blue)](https://github.com/Vaskivskyi/ha-asusrouter/releases) [![License](https://img.shields.io/github/license/Vaskivskyi/ha-asusrouter.svg?style=for-the-badge&color=yellow)](LICENSE) [![HACS Default](https://img.shields.io/badge/HACS-default-blue.svg?style=for-the-badge)](https://hacs.xyz) [![Community forum discussion](https://img.shields.io/badge/COMMUNITY-FORUM-success?style=for-the-badge&color=yellow)](https://community.home-assistant.io/t/custom-component-asusrouter-integration/416111)<a href="https://github.com/Vaskivskyi/ha-asusrouter/actions/workflows/build.yaml"><img src="https://img.shields.io/github/workflow/status/Vaskivskyi/ha-asusrouter/Build?style=for-the-badge" alt="Build Status" align="right" /></a>

## AsusRouter - Monitor and control your Asus router from Home Assistant

AsusRouter is a custom integration for Home Assistant to monitor and control your Asus router using [AsusRouter](https://github.com/Vaskivskyi/asusrouter) python library.

The integration uses the native Asus HTTP(S) API - the same way the web panel or mobile app works and relies on direct communications with your device. Even though this is not the primary purpose of the integration, one can configure it for usage with the remote device over the global network as well.

You could always help with its development by providing your feedback.

## Installation

#### HACS

You can add this repository in your HACS:
`HACS -> Integrations -> Explore & Download Repositories -> AsusRouter`

#### Manual

Copy content of the [stable branch](https://github.com/Vaskivskyi/ha-asusrouter/tree/stable) `custom_components/asusrouter/` to `custom_components/asusrouter/` in your Home Assistant folder.

## Usage

After AsusRouter is installed, you can add your device from Home Assistant UI.

[![Open your Home Assistant instance and start setting up a new integration.](https://my.home-assistant.io/badges/config_flow_start.svg)](https://my.home-assistant.io/redirect/config_flow_start/?domain=asusrouter)

To connect to the device you need to provide the following data:
- IP address or hostname

- Username (the one you use for login in the Asus web panel)
- Password
- Whether to use an SSL connection

With version 0.3.0 of the integration, its setup for a new device has become simpler and cleaner. If your device uses default connection settings, you will probably never need to type in all the possible settings. Our [Setup documentation](https://github.com/Vaskivskyi/ha-asusrouter/blob/main/docs/setup.md) may provide you with detailed information on the configuration flow.

#### Reconfiguration

Almost all the integration settings can be reconfigured later via the `Configure` button on the Integrations page without the need to remove your device and add it again.

[![Open your Home Assistant instance and show your integrations.](https://my.home-assistant.io/badges/integrations.svg)](https://my.home-assistant.io/redirect/integrations/)

#### Selecting network interfaces for monitoring

Entities for the newly selected network interfaces will be created automatically. Removal of the entities which you don't need anymore should be done manually.

#### Lights

These entities require `Device control` to be enabled in the integration settings. Some of the entities may not be available for your device because of their firmware limitation.

<details>
<summary>LED</summary>

*(enabled by default)*
  - name: `led`
  - description: Light entity allows user to control LED state of the device
  - not available for: `RT-AC66U`
</details>

#### Sensors

<details>
<summary>Network traffic and speed</summary>

*(enabled by default)*

- **Traffic**:
  - names: `{}_download` / `{}_upload` [^traffic]
  - units: `GB` [^units]
  - attributes:
    - `Bytes` - raw data from device
- **Speed**:
  - names: `{}_download_speed` / `{}_upload_speed`
  - units: `Mb/s`
  - attributes:
    - `Bits/s` - raw data from device

Possible network interfaces (can be changed via the `Configure` button for the configuration):
- `WAN` - traffic to your ISP (*Some of the devices do not report WAN data, refer to the [issue](https://github.com/Vaskivskyi/ha-asusrouter/issues/30). If your device also doesn't show such sensors, please add your information to this issue*)
- `USB` - traffic to the USB modem / mobile phone connected via USB
- `LAN` - local wired traffic
- `WLANx` - wireless traffic: `0` - 2.4 GHz WiFi, `1` and `2` - 5 GHz WiFi
- and more
</details>

<details>
<summary>CPU usage sensor</summary>

*(disabled by default)*
  - name: `cpu`
  - units: `%`
  - attributes:
    - `Core X` - usage by corer `x`
  - description: Sensor shows average CPU usage.
</details>

<details>
<summary>RAM usage sensor</summary>

*(disabled by default)*
  - name: `ram`
  - units: `%`
  - attributes (all in `KB`, as device reports):
    - `Total`
    - `Free`
    - `Used`
  - description: Sensor represents RAM usage of the device. In most cases, it slowly increases with time. On reboot, RAM usage drops. 
</details>

<details>
<summary>Connected devices sensor</summary>

*(enabled by default)*
  - name: `connected_devices`
  - units: ` `
  - description: Sensor shows the total number of devices connected.
</details>

<details>
<summary>Boot time sensor</summary>

*(disabled by default)*
  - name: `boot_time`
  - units: ` `
  - description: Sensor represents the last time the device was rebooted.
</details>

<details>
<summary>Ports connection sensors</summary>

*(disabled by default)*

  - names: `lan_speed` / `wan_speed`
  - units: `Mb/s`
  - attributes:
    - `LAN x` / `WAN x` - represents speed of each port `x` in `Mb/s`
  - description: Sensor value represents the total speed on all the connected LAN / WAN ports. E.g. if 2 ports arer connected in `1 Gb/s` mode and 1 - in `100 Mb/s` mode, this value will be `2100 Mb/s`.
</details>

<details>
<summary>WAN IP sensor</summary>

*(disabled by default)*

  - name: `wan_ip`
  - attributes:
    - `Type` - type of the IP (may be `static`, `dhcp` and more)
    - `Gateway`
    - `Mask`
    - `DNS`
    - `Private subnet`
  - description: Sensor value represents the current external IP address of the device
</details>

[^traffic]: Asus routers calculate traffic as an 8-digit HEX, so it overflows each 4 294 967 296 bytes (4 GB). One can either use [Long-term statistics](#long-term-statistics) or create a [`utility_meter`](https://www.home-assistant.io/integrations/utility_meter/) for more control and the possibility to monitor traffic above the limitation.
[^units]: The integration uses 2^10 conversion factors in calculations - the same way as native Asus software. If you would like to use 10^3 factors, all the sensors provide raw data in bytes or bits/s.

#### Binary sensors

<details>
<summary>WAN status</summary>

*(disabled by default)*

  - name: `wan`
  - attributes:
    - `IP`
    - `Type` - type of the IP (may be `static`, `dhcp` and more)
    - `Gateway`
    - `Mask`
    - `DNS`
    - `Private subnet`
  - description: Sensor value represents the internet connection of the device
</details>

#### Device trackers

Device trackers are disabled by default if HA doesn't know the device. The generated `device_tracker.device_name` entities include `device_name` as reported by AsusRouter library according to the priority list:
- Friendly name of the device (if set in the router admin panel)
- Name as reported by the connected device
- MAC address if neither of the above is known

<details>
<summary>Attributes</summary>

- `connection_time` - time of connection to the router (only wireless devices)
- `connection_type` - type of connection (`Wired`, `2.4 GHz`, `5 GHz`). Implementation of `6 GHz` requires a test device.
- `host_name` - name of the connected device as stated before
- `internet` - internet connection of the device (`connected`, `disconnected` or `blocked` - if internet access is restricted by router)
- `ip`
- `ip_method` - (`Manual`, `DHCP` and other)
- `last_activity` - last time device was seen online in HA
- `mac`
- `rssi` - (only wireless devices)
- `rx_speed` - (only wireless devices)
- `tx_speed` - (only wireless devices)
</details>

**Important**

If the connected device's MAC address is known to Home Assistant (e.g. it was set up by another integration `ABC`), its `device_tracker` will be added to that device in the enabled state. Moreover, this device will be showing for both integrations: `AsusRouter` and `ABC`

#### Long-term statistics

The long-term statistics is supported for all the sensors, but `boot_time`.

## More features

#### Secondary WAN (USB)

If your device supports the dual-WAN feature, the [corresponding sensors](#sensors) can be created. They will be showing the correct values when USB WAN is connected and `unknown`, when a phone / USB modem is disconnected.

This is expected behaviour and should be handled by integration automatically (without the need for a user to restart the integration).

#### Reboot handle

Integration does support automatic handling of the device reboots. Most of the routers reboot in approximately 1-1.5 min. After that time, a new connection will be done automatically.

There will be an error in your log in this case.

If you are experiencing multiple errors in your log (more than 5 in a row) or integration does not provide any data 5 minutes after a reboot, please report it to the Issues. The more data you provide, the easier it will be to fix the problem.

## Supported devices

The list of supported devices includes (but is not limited to):

802.11 AC models:
- `RT-AC66U` (without LED control, firmware limitation)
- `RT-AC86U` (a known issue https://github.com/Vaskivskyi/ha-asusrouter/issues/29, produces warnings in the log)
- `RT-ACRH13`

802.11 AX models:
- `RT-AX58U`
- `RT-AX68U`
- `RT-AX86U`
- `RT-AX88U`
- `RT-AX89X`
- `RT-AX92U`
- `ZenWiFi AX (XT8)`

If your device is not listed but is confirmed to work, you are welcome to open PR with a proposed change to this list. Alternatively, you can open an issue with the device model and it will be added to the list.

## Support the integration

### Issues and Pull requests

If you have found an issue working with the integration or just want to ask for a new feature, please fill in a new [issue](https://github.com/Vaskivskyi/ha-asusrouter/issues).

You are also welcome to submit [pull requests](https://github.com/Vaskivskyi/ha-asusrouter/pulls) to the repository!

### Check it with your device

Testing the integration with different devices would help a lot in the development process. Unfortunately, currently, I have only one device available, so your help would be much appreciated.

### Other support

This integration is a free-time project. If you like it, you can support me by buying a coffee.

<a href="https://www.buymeacoffee.com/vaskivskyi" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-blue.png" alt="Buy Me A Coffee" style="height: 40px !important;"></a>

## Thanks to

The initial codebase for this integration is highly based on Home Assistant core integration [AsusWRT](https://www.home-assistant.io/integrations/asuswrt/) and [ollo69/ha_asuswrt_custom](https://github.com/ollo69/ha_asuswrt_custom).


