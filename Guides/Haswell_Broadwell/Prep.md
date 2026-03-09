[![OpenCore Version](https://img.shields.io/badge/OpenCore_Version:-0.9.4+-success.svg)](https://github.com/acidanthera/OpenCorePkg) ![macOS](https://img.shields.io/badge/Supported_macOS:-≤15.x-white.svg)

# Installing macOS 13+ on Haswell/Broadwell systems

## Preparations

### Backup your working EFI

**Backup** your working EFI folder on a FAT32 formatted USB Flash Drive just in case something goes wrong because we have to modify the config and content of the EFI folder.

### Update OpenCore and kexts
Update OpenCore and kexts to the latest version to maximize compatibility with macOS. To check which version of OpenCore you're currently using, run the following commands in Terminal:

```shell
nvram 4D1FDA02-38C7-4A6A-9CC6-4BCCA8B30102:opencore-version
```

For updating OpenCore, Drivers and kexts easily, you can use OpenCore Auxiliary Tools (OCAT) &rarr; [Instructions](https://github.com/5T33Z0/OC-Little-Translated/blob/main/Content/D_Updating_OpenCore/Updating_OC.md)

## Additional Checks
This is what you need to know before attempting to install macOS Ventura on unsupported systems:

- **iGPU/GPU**: Check if your iGPU/GPU is supported by OCLP. Although Drivers for Intel, NVIDIA and AMD cards can be added in Post-Install, the [list is limited](https://dortania.github.io/OpenCore-Legacy-Patcher/PATCHEXPLAIN.html#on-disk-patches) 
- Check if any peripherals you are using are compatible with macOS 12+ (Printers, WiFi and Bluetooth deviced come to mind).
- **Networking**:
	- For **Ethernet**, there are kexts for legacy LAN controllers [available here](https://github.com/dortania/OpenCore-Legacy-Patcher/tree/main/payloads/Kexts/Ethernet)
	- **Wifi and Bluetooth**:
		- For enabling Broadcom Wifi/BT Cards, you will need a different [set of kexts](https://github.com/5T33Z0/OC-Little-Translated/tree/main/10_Kexts_Loading_Sequence_Examples#example-7-broadcom-wifi-and-bluetooth) to load which need to be controlled via `MinKernel` and `MaxKernel` settings. On macOS 12.4 and newer, a new address check has been introduced in `bluetoothd`, which will trigger an error if two Bluetooth devices have the same address. This can be circumvented by adding boot-arg `-btlfxallowanyaddr` (provided by [BrcmPatchRAM](https://github.com/acidanthera/BrcmPatchRAM) kext).
		- Same applies to [Intel WiFi/BT](https://github.com/5T33Z0/OC-Little-Translated/tree/main/10_Kexts_Loading_Sequence_Examples#example-8a-intel-wifi-airportitlwm-and-bluetooth-intelbluetoothfirmware) cards using [OpenIntelWirless](https://github.com/OpenIntelWireless) kexts
		- [Enabling Wifi in macOS Sonoma](/Enable_Features/WiFi_Sonoma.md) requires additional kext and also applying root patches in Post-Install!
- **Security**: Modifying the system with OCLP Requires SIP, Apple Secure Boot and AMFI to be disabled so there are some compromises in terms of security.
- **System Updates**: Incremental (or delta) updates won't be available after applying root patches with OCLP. Instead, the whole macOS Installer will be downloaded every time (approx. 15 GB for the latest OS), since root patching breaks the security seal of the volume! :bulb: In Haswell and newer, you can actually workaround this issue by reverting the root patches *prior* to checking for updates. Then, a regular incremental update will be installed which is much smaller. Afterwards you just have to re-apply the root patches again.

---

[← **Previous: Introduction**](README.md) | [**Next: Config Asjustments →**](Haswell_Config.md)
