# Alternative Repositories

## Long Live the Fork!

With the release of macOS 27 (codenamed Golden Gate), Hackintoshing is slowly coming to an end, and several kext repositories have become dormant: development has stalled, pull requests go unmerged, and issues remain unresolved.

But not all is lost! While the main repository of a kext or app may be dormant, plenty of work continues behind the scenes—or, more precisely, in forks. Other community members are picking up development and, with the help of AI-assisted coding, not only keeping projects alive but also addressing long-standing issues that have remained unresolved for years.

Two prime examples are Intel Wi-Fi support in macOS Sequoia and Tahoe, as well as Metal support for modern NVIDIA graphics cards.

The purpose of this document is to provide a list of alternative repositories for existing kexts, drivers, and tools that are still actively maintained.

## Kext, Driver, and App Alternatives

Below is an incomplete list of actively maintained alternatives to existing kext, driver, and app repositories.

### 🔊 Audio
| Original Kext | Alternative | Improvements |
| --- | --- | --- |
| [**SimpleHDA**](https://github.com/laobamac/SimpleHDA) | N/A (new kext) | Enables analog Audio in macOS Tahoe without root patching |

---

### 🎨 Graphics

#### 🟩 NVIDIA

| Original Kext | Alternative | Improvements |
| --- | --- | --- |
| [**nvidia-macos-driver**](https://github.com/nullmoth/nvidia-macos-driver) | N/A (new kext) | Enables Metal 3 support for modern NVIDIA graphics cards in macOS |

#### 🟥 AMD

| Original Kext | Alternative | Improvements |
| --- | --- | --- |
| [**Cobalt64**](https://github.com/Almosst-DEV/Cobalt64) | N/A (new driver) | Experimental native macOS GPU acceleration for unsupported AMD Radeon GPUs, starting with RDNA 4 (RX 9070 XT / Navi 48). Uses a custom kernel driver and a port of Mesa's RADV Vulkan driver to enable Metal rendering. Currently in preview; not recommended for everyday use yet. |

---

### 🌐 Networking

#### Ethernet

| Original Kext | Alternative | Improvements |
| --- | --- | --- |
| [**AppleIGC**](https://github.com/SongXiaoXi/AppleIGC) | [dr0pmead/AppleIGC](https://github.com/dr0pmead/AppleIGC) | Focuses on stability with Intel I225/I226 2.5GbE controllers on macOS Sequoia and Tahoe. Includes fixes for kernel panics caused by malformed network packet buffers, improved sleep/wake handling, and driver optimizations to reduce link instability and buffer-related stalls. |
| [**IntelMausi**](https://github.com/acidanthera/IntelMausi) | [IntelMausiEthernet](https://github.com/Mieze/IntelMausiEthernet) | Added AppleVTD support in macOS Tahoe |

#### 🛜 Wi-Fi

##### Broadcom Wi-Fi

This is not a direct replacement, but an alternative for specific Broadcom Wi-Fi cards.

| Original Kext | Alternative | Improvements/Limitations |
| --- | --- | --- |
[**AirportBrcmFixup**](https://github.com/acidanthera/AirportBrcmFixup) | [AppleBCMWLANCompanion](https://github.com/0xFireWolf/AppleBCMWLANCompanion) | Enables the use of supported Broadcom Wi-Fi cards on macOS Sonoma, Sequoia, and Tahoe without root patches. However, only a limited selection of Broadcom FullMAC Wi-Fi cards is supported. |

##### Intel Wi-Fi

| Original Kext/App | Alternative | Improvements |
| --- | --- | --- |
| [**HeliPort**](https://github.com/OpenIntelWireless/HeliPort) | [joshcalvert47/HeliPort](https://github.com/joshcalvert47/HeliPort) | Adds a signal quality graph and speeds up automatic connection to saved networks at login |
| [**itlwm**](https://github.com/OpenIntelWireless/itlwm) | [laobamac/itlwm](https://github.com/laobamac/itlwm/releases) | Provides native AirportItlwm kexts for macOS Sequoia and Tahoe, eliminating the need for OCLP root patches ([More details](https://github.com/5T33Z0/OCLP4Hackintosh/blob/main/Enable_Features/AirportItlwm_Sequoia.md)) |
| [**itlwm**](https://github.com/OpenIntelWireless/itlwm) | [jzrodriguez98/itlwm](https://github.com/jzrodriguez98/itlwm/tree/laobamac-updates-integration) | Includes laobamacs changes for AirportItlwm in Sequoia/Tahoe but adds Apple-VTD compatibility for `Itlwm.kext`

---

### Bluetooth

#### BrcmPathRAM

| Original Kext | Alternative | Improvements/Limitations |
| --- | --- | --- |
[**BrcmPathRAM**](https://github.com/acidanthera/BrcmPatchRAM) | [hexxyan/BlueToolFixup-FixA2DP](https://github.com/hexxyan/BlueToolFixup-FixA2DP) | This is from an [open PR](https://github.com/acidanthera/BrcmPatchRAM/pull/38) which hasn't been merged for months. It adds an opt-in runtime patch for `bluetoothd` to **`Bluetoolfixup.kext`** that resolves the infamous "Connected but No Sound" issue when using third-party Bluetooth controllers (e.g. Intel) with modern Apple headphones equipped with W1/H1/H2 chips (AirPods Pro 2, AirPods Pro 3, etc.). Requires additional boot-arg `-btlfxa2dpcheck` to enable the patch, since it intercepts a core system daemon path at the binary level.

#### Intel Bluetooth Firmware

| Original Kext | Alternative | Improvements/Limitations |
| --- | --- | --- |
| [**IntelBluetoothFirmware**](https://github.com/OpenIntelWireless/IntelBluetoothFirmware) | [lshbluesky/IntelBluetoothFirmware](https://github.com/lshbluesky/IntelBluetoothFirmware/releases) | Improved BLE pairing; additional and updated firmware; macOS Tahoe compatibility. |

--- 

### SD Card Readers

| Original Kext | Alternative | Improvements |
| --- | --- | --- |
| [**RealtekCardReader**](https://github.com/0xFireWolf/RealtekCardReader) | [Baio1977/RealtekCardReader](https://github.com/Baio1977/RealtekCardReader) | Adds support for macOS Sonoma through Tahoe |
| [**Sinetek-rtsx**](https://github.com/cholonam/Sinetek-rtsx) | [enigma131/Sinetek-rtsx](https://github.com/enigma131/Sinetek-rtsx) | Adds legacy build support for OS X Tiger and Leopard (Xcode 2.5, 32-bit) and Snow Leopard (Xcode 4.2, 64-bit). Provides a universal binary intended to extend compatibility across older and newer macOS versions, including Tahoe. Focuses on build compatibility rather than new chipset support or driver performance improvements. |

### ⚙️ System Management

| Original Kext | Alternative | Improvements |
| --- | --- | --- |
[**YogaSMC**](https://github.com/zhen-zen/YogaSMC) | [hoaug-tran/ReYogaSMC](https://github.com/hoaug-tran/ReYogaSMC/) | Brings support for macOS 11 to 26.

---

### UEFI Drivers

| Original Driver | Alternative | Improvements |
| --- | --- | --- |
| [**DisablePROCHOT.efi**](https://github.com/arter97/DisablePROCHOT) | [Magniquick/DisablePROCHOT](https://github.com/Magniquick/DisablePROCHOT/releases) | The original dates back to 2018, while the fork was released in 2026 and includes bug fixes and other improvements. |

---

### 🧰 Apps/Tools

#### OpenCore Config Editors

| Original | Fork | Improvements |
| --- | --- | --- |
[**OCAuxiliaryTools**](https://github.com/ic005k/OCAuxiliaryTools) | [YBronst/OCAuxiliaryTools](https://github.com/YBronst/OCAuxiliaryTools/releases) | Supports drag and drop of list entries, added more kexts, general improvements

#### 📊 System Monitoring

| Original | Fork | Improvements |
| --- | --- | --- |
| [**MenuMeters**](https://github.com/yujitach/MenuMeters) | [leeliu/MenuMeters](https://github.com/leeliu/MenuMeters/releases) or as an alernative: [Stats](https://github.com/exelban/stats) | Compatible with macOS Tahoe 26.x

*To be continued…* 

**PS: Create an Issue/PR, if you find more up-to-date repos to add.**
