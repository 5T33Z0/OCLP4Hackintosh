# Alternative Repositories

## Long Live the Fork!

With the release of macOS 27 (codenamed Golden Gate), Hackintoshing is slowly coming to an end, and several kext repositories have become dormant: development has stalled, pull requests go unmerged, and issues remain unresolved.

But not all is lost! While the main repository of a kext or app may be dormant, plenty of work continues behind the scenes—or, more precisely, in forks. Other community members are picking up development and, with the help of AI-assisted coding, not only keeping projects alive but also addressing long-standing issues that have remained unresolved for years.

Two prime examples are Intel Wi-Fi support in macOS Sequoia and Tahoe, as well as Metal support for modern NVIDIA graphics cards.

The purpose of this document is to provide a list of alternative repositories for existing kexts, drivers, and tools that are still actively maintained.

## Kext, Driver, and App Alternatives

Below is an incomplete list of actively maintained alternatives to existing kext, driver, and app repositories.

### Audio
| Original Kext | Alternative | Improvements |
| --- | --- | --- |
| [**SimpleHDA**](https://github.com/laobamac/SimpleHDA) | N/A (new kext) | Enables analog Audio in macOS Tahoe without root patching |

---

### Graphics

#### NVIDIA

| Original Kext | Alternative | Improvements |
| --- | --- | --- |
| [**nvidia-macos-driver**](https://github.com/nullmoth/nvidia-macos-driver) | N/A (new kext) | Enables Metal 3 support for modern NVIDIA graphics cards in macOS |

---

### Networking

#### Ethernet

| Original Kext | Alternative | Improvements |
| --- | --- | --- |
| [**AppleIGC**](https://github.com/SongXiaoXi/AppleIGC) | [dr0pmead/AppleIGC](https://github.com/dr0pmead/AppleIGC) | Focuses on stability with Intel I225/I226 2.5GbE controllers on macOS Sequoia and Tahoe. Includes fixes for kernel panics caused by malformed network packet buffers, improved sleep/wake handling, and driver optimizations to reduce link instability and buffer-related stalls. |
| [**IntelMausi**](https://github.com/acidanthera/IntelMausi) | [IntelMausiEthernet](https://github.com/Mieze/IntelMausiEthernet) | Added AppleVTD support in macOS Tahoe |

#### Wi-Fi

##### Broadcom Wi-Fi

This is not a direct replacement, but an alternative for specific Broadcom Wi-Fi cards.

| Original Kext | Alternative | Improvements/Limitations |
| --- | --- | --- |
[**AirportBrcmFixup**](https://github.com/acidanthera/AirportBrcmFixup) | [AppleBCMWLANCompanion](https://github.com/0xFireWolf/AppleBCMWLANCompanion) | Enables the use of supported Broadcom Wi-Fi cards on macOS Sonoma, Sequoia, and Tahoe without root patches. However, only a limited selection of Broadcom FullMAC Wi-Fi cards is supported. |

##### Intel Wi-Fi

| Original Kext/App | Alternative | Improvements |
| --- | --- | --- |
| [**HeliPort**](https://github.com/OpenIntelWireless/HeliPort) | [perez987/HeliPort](https://github.com/perez987/HeliPort) | Adds a signal quality graph and speeds up automatic connection to saved networks at login |
| [**itlwm**](https://github.com/OpenIntelWireless/itlwm) | [laobamac/itlwm](https://github.com/laobamac/itlwm/releases) | Provides native AirportItlwm kexts for macOS Sequoia and Tahoe, eliminating the need for OCLP root patches ([More details](https://github.com/5T33Z0/OCLP4Hackintosh/blob/main/Enable_Features/AirportItlwm_Sequoia.md)) |
| [**itlwm**](https://github.com/OpenIntelWireless/itlwm) | [jzrodriguez98/itlwm](https://github.com/jzrodriguez98/itlwm/tree/laobamac-updates-integration) | Includes laobamacs changes for AirportItlwm in Sequoia/Tahoe but adds Apple-VTD compatibility for `Itlwm.kext`

---

### Bluetooth

#### Intel Bluetooth Firmware

| Original Kext | Alternative | Improvements/Limitations |
| --- | --- | --- |
| [**IntelBluetoothFirmware**](https://github.com/OpenIntelWireless/IntelBluetoothFirmware) | [lshbluesky/IntelBluetoothFirmware](https://github.com/lshbluesky/IntelBluetoothFirmware/releases) | Improved BLE pairing; additional and updated firmware; macOS Tahoe compatibility. |

--- 

### SD Card Readers

| Original Kext | Alternative | Improvements |
| --- | --- | --- |
| [**RealtekCardReader**](https://github.com/0xFireWolf/RealtekCardReader) | [Baio1977/RealtekCardReader](https://github.com/Baio1977/RealtekCardReader) | Adds support for macOS Sonoma through Tahoe |

---

### UEFI Drivers

| Original Driver | Alternative | Improvements |
| --- | --- | --- |
| [**DisablePROCHOT.efi**](https://github.com/arter97/DisablePROCHOT) | [Magniquick/DisablePROCHOT](https://github.com/Magniquick/DisablePROCHOT/releases) | The original dates back to 2018, while the fork was released in 2026 and includes bug fixes and other improvements. |

---

### Apps/Tools

#### System Monitoring

| Original | Fork | Improvements |
| --- | --- | --- |
| [**MenuMeters**](https://github.com/yujitach/MenuMeters) | [leeliu/MenuMeters](https://github.com/leeliu/MenuMeters/releases) or as an alernative: [Stats](https://github.com/exelban/stats) | Compatible with macOS Tahoe 26.x

*To be continued…* 

**PS: Create an Issue/PR, if you find more up-to-date repos to add.**
