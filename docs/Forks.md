# Alternative Repositories

## Long Live the Fork!

With the release of macOS 27 (codenamed Golden Gate), Hackintoshing is slowly coming to an end, and several kext repositories have become dormant: development has stalled, pull requests go unmerged, and issues remain unresolved.

But not all is lost! While the main repository of a kext or app may be dormant, plenty of work continues behind the scenes – or, more precisely, in forks. Other community members are picking up development and, with the help of AI-assisted coding, not only keeping projects alive but also addressing long-standing issues that have remained unresolved for years.

Two prime examples are Intel Wi-Fi support in macOS Sequoia and Tahoe, as well as Metal support for modern NVIDIA graphics cards.

The purpose of this document is to provide a list of alternative repositories for existing kexts, drivers, and tools that are still actively maintained.

## Kext, Driver, and App Alternatives

Below is an incomplete list of actively maintained alternatives to existing kext, driver, and app repositories.

### 🔊 Audio

| Original Kext | Alternative | Description |
| --- | --- | --- |
| [**SimpleHDA**](https://github.com/laobamac/SimpleHDA) | N/A (new kext) | Restores the native AppleHDA stack in macOS Tahoe without root patching, allowing AppleALC to use existing audio configurations. |
| [**CirrusAudioFixup**](https://github.com/hoaug-tran/CirrusAudioFixup)|  N/A (new kext) | Experimental support for laptops with CS35L41 smart amplifiers. Initializes the amplifier, loads DSP firmware, and enables internal speakers with volume control and sleep/wake support. |
| [**AMDMicrophone**](https://github.com/qhuyduong/AMDMicrophone) | N/A (new kext) | Enables digital microphone input on compatible AMD laptops using the Audio Co-Processor (ACP). Primarily targets Renoir-based systems. Requires installation in `/Library/Extensions` and reduced SIP security; AppleALC is still needed for audio output. |

---

### 🎨 Graphics

#### 🟥 AMD

| Original Kext | Alternative | GPU Type / Generation | Description |
| --- | --- | --- | --- |
| [**Cobalt64**](https://github.com/Almosst-DEV/Cobalt64) | N/A (new driver) | RDNA 4 (Navi 48) | Experimental native macOS GPU acceleration for unsupported AMD Radeon GPUs, starting with RDNA 4 (RX 9070 XT / Navi 48). Uses a custom kernel driver and a port of Mesa's RADV Vulkan driver to enable Metal rendering. Currently in preview; not recommended for everyday use yet. |
| [**RedNavi**](https://github.com/Helemen7/RedNavi) | N/A (new driver) | RDNA 3 (Navi 32) | Native macOS display driver for AMD Radeon RX 7800 XT. ⚠️ Research prototype, not a driver you can install and use.|
| [**AMD RDNA 3 macOS Driver**](https://github.com/luuumi-dev/amd-rdna3-macos-driver) | N/A (new driver) | RDNA 3 (Navi 3x) | Work-in-progress driver for AMD RDNA 3 GPUs, aiming to enable graphics acceleration in macOS. Experimental; full graphics acceleration and Metal support are not yet established. |
| [**NootedRed**](https://github.com/ChefKissInc/NootedRed) | N/A (new driver) | Vega-based APUs (GCN 5) | Patches Apple's AMD graphics drivers to enable graphics acceleration on unsupported AMD Vega integrated GPUs, including those found in Raven Ridge, Picasso, Renoir, Cezanne, and Barcelo APUs. Experimental and under active development. |
| [**CarrizoRed**](https://github.com/CL0utM4N/CarrizoRed) | N/A (new driver) | GCN 3 (GFX8) APUs | Experimental graphics support for older AMD GCN 3 (GFX8) APUs using Apple's Polaris drivers. Targets Carrizo, Bristol Ridge, and Stoney Ridge. Currently experimental, with demonstrated functionality limited to Stoney Ridge; Carrizo and Bristol Ridge remain untested. |
| [**KaberiRed**](https://github.com/CL0utM4N/KaberiRed) | N/A (new driver) | AMD GCN 2 (Bonaire) | Adapts Apple's GCN 2 drivers for AMD Kabini, Mullins and Kaveri APUs. |

#### 🟦 Intel

| Original Kext | Alternative | Description |
| --- | --- | --- 
| [**WhateverXe**](https://github.com/b00t0x/WhateverXe) +<br> [**WhateverXe-Graphics**](https://github.com/b00t0x/WhateverXe-Graphics)  | N/A (new driver) | Experimental graphics acceleration for selected Intel Xe-LP iGPUs, currently targeting Tiger Lake (IDs `9A40`, `9A49`, `9A78`) and Alder Lake-N / Twin Lake (IDs `46D0`–`46D4`). Requires Lilu, a modified OpenCore Legacy Patcher Xe root patch, a modified AMFIPass kext and specific system security settings. Current documented target: macOS Big Sur 11. Additional macOS versions and GPU families are planned. Check the repositories for supported GPU IDs, macOS compatibility, and installation instructions. 

#### 🟩 NVIDIA

| Original Kext | Alternative | GPU Type / Generation | Description |
| --- | --- | --- | --- |
| [**nvidia-macos-driver**](https://github.com/nullmoth/nvidia-macos-driver) | N/A (new kext) | Modern NVIDIA GPUs (Turing and newer) | Experimental driver aiming to enable Metal 3 support for modern NVIDIA graphics cards in macOS. 

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
| [**AirportBrcmFixup**](https://github.com/acidanthera/AirportBrcmFixup) | [laobamac/SimpleBCM](https://github.com/laobamac/SimpleBCM) | SimpleBCM aims to restore native Wi-Fi support for legacy Broadcom cards on macOS Sequoia and Tahoe without OCLP root patches. Experimental; check the repository for supported chipsets, macOS versions, and current limitations before use. |
[**AirportBrcmFixup**](https://github.com/acidanthera/AirportBrcmFixup) | [AppleBCMWLANCompanion](https://github.com/0xFireWolf/AppleBCMWLANCompanion) | Enables the use of supported Broadcom Wi-Fi cards on macOS Sonoma, Sequoia, and Tahoe without root patches. However, only a limited selection of Broadcom FullMAC Wi-Fi cards is supported. |

##### Intel Wi-Fi

| Original Kext/App | Alternative | Improvements |
| --- | --- | --- |
| [**HeliPort**](https://github.com/OpenIntelWireless/HeliPort) | [joshcalvert47/HeliPort](https://github.com/joshcalvert47/HeliPort) | Adds a signal quality graph and speeds up automatic connection to saved networks at login |
| [**itlwm**](https://github.com/OpenIntelWireless/itlwm) | [laobamac/itlwm](https://github.com/laobamac/itlwm/releases) | Provides native AirportItlwm kexts for macOS Sequoia and Tahoe, eliminating the need for OCLP root patches ([More details](https://github.com/5T33Z0/OCLP4Hackintosh/blob/main/Enable_Features/AirportItlwm_Sequoia.md)) |
| [**itlwm**](https://github.com/OpenIntelWireless/itlwm) | [jzrodriguez98/itlwm](https://github.com/jzrodriguez98/itlwm/tree/laobamac-updates-integration) | Includes laobamacs changes for AirportItlwm in Sequoia/Tahoe but adds Apple-VTD compatibility for `Itlwm.kext`

##### Realtek

| Original Kext | Alternative | Improvements |
| --- | --- | --- |
| [**Realtek AirPort Family**](https://github.com/xnoah222/Realtek-AirPort-Family) | N/A (new kext) | Collection of native-style AirPort Wi-Fi drivers for selected Realtek wireless chipsets, with support targeting macOS Mojave through Tahoe. Chipset and macOS compatibility vary by driver. |
| [**RTL8723BE-macOS-WiFi**](https://github.com/AA-EION/RTL8723BE-macOS-WiFi) | N/A (new kext) | Experimental Wi-Fi support for the [Realtek RTL8723BE](https://www.realtek.com/Product/Index?id=609&cate_id=194&menu_id=291) chipset. Provides network scanning and connection to 2.4 GHz WPA2 networks through a menu bar app. Performance and stability may be limited. |
| [**AirPort_RTW89**](https://github.com/FranciumOS-org/Airport_RTW89) | N/A (new kext) | Native macOS Wi-Fi support for selected Realtek RTL89xx PCIe adapters, based on Linux's `rtw89` driver. Targets Wi-Fi 6/6E/7 chipsets, including RTL8852BE, RTL8852CE, and RTL8922DE. Experimental; support varies by chipset. |
| [**AirPort_RTW88**](https://github.com/xnoah222/Airport_RTW88) | N/A (new kext) | Native macOS Wi-Fi support for selected Realtek RTL88xx PCIe adapters, based on Linux's `rtw88` driver. Supports RTL8822BE, RTL8822CE, and RTL8821CE. Provides Wi-Fi through macOS's native interface; full AirDrop and AWDL support is not yet available. |
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

### 🧩 Device Support

#### 💾 SD Card Readers

| Original Kext | Alternative | Improvements |
| --- | --- | --- |
| [**RealtekCardReader**](https://github.com/0xFireWolf/RealtekCardReader) | [Baio1977/RealtekCardReader](https://github.com/Baio1977/RealtekCardReader) | Adds support for macOS Sonoma through Tahoe |
| [**Sinetek-rtsx**](https://github.com/cholonam/Sinetek-rtsx) | [enigma131/Sinetek-rtsx](https://github.com/enigma131/Sinetek-rtsx) | Adds legacy build support for OS X Tiger and Leopard (Xcode 2.5, 32-bit) and Snow Leopard (Xcode 4.2, 64-bit). Provides a universal binary intended to extend compatibility across older and newer macOS versions, including Tahoe. Focuses on build compatibility rather than new chipset support or driver performance improvements. |

#### 📷 Webcams

| Kext | Alternative | Improvements |
| --- | --- | --- |
| [**UVCProbeFix**](https://www.insanelymac.com/forum/topic/363309-uvcprobefix-for-macos-tahoe-fix-broken-usb-webcam-video-and-black-screen-on-hackintosh/) | N/A (new kext) | Experimental fix for USB UVC webcams that are detected by macOS Tahoe but produce a black screen because of malformed UVC PROBE/COMMIT negotiation. Source code is not publicly provided. |

---

### 💽 File System Drivers

| Original Kext / Driver | Alternative | Improvements |
| --- | --- | --- |
| [**NTFS Oficial**](https://github.com/dmanske/ntfs-oficial-publico)|  N/A (new driver) | Free, open-source NTFS read/write support for Intel and Apple Silicon Macs without NTFS-3G or macFUSE. Includes automatic mounting, a menu bar app, and tools to verify, repair, and format NTFS volumes. Requires macOS 14 or newer. Experimental project; back up important data before use. |

---

### ⚙️ System Management

| Original Kext | Alternative | Improvements |
| --- | --- | --- |
| [**YogaSMC**](https://github.com/zhen-zen/YogaSMC) | [**ReYogaSMC**](https://github.com/hoaug-tran/ReYogaSMC/) | Community-maintained fork extending support for Lenovo laptops running macOS 11 Big Sur through macOS 26 Tahoe. Provides Lenovo-specific power management, hotkey handling, battery information, and hardware monitoring. |

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
