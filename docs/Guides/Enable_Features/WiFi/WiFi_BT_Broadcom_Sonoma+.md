# Re-Enabling Broadcom Wi-Fi and Bluetooth in macOS Sonoma and newer

## Overview

During the early stages of macOS Sonoma development, Apple removed several kexts and frameworks required by older Broadcom Wi-Fi cards. As a result, Wi-Fi functionality stopped working on many commonly used Broadcom cards.

There are two approaches to restoring Broadcom Wi-Fi functionality in macOS Sonoma (14), Sequoia (15) and Tahoe (26):

1. **SimpleBrcm Configuration (No Wi-Fi Root Patches Required):** Uses [SimpleBrcm](https://github.com/laobamac/SimpleBCM), a Lilu plugin that restores Broadcom Wi-Fi support through OpenCore injection. It uses the native wireless frameworks provided by macOS and does not require OCLP Wi-Fi root patches.

2. **OCLP Configuration (Root Patches Required):** Uses legacy Apple wireless drivers, additional kexts and OpenCore configuration changes, followed by OCLP Post-Install Root Patches.

Start with the SimpleBrcm method. If your card or configuration is not supported by SimpleBrcm and requires the traditional OCLP approach, follow the second method instead.

Bluetooth is covered separately because its configuration is independent of the Wi-Fi restoration method.

## 1. SimpleBrcm Configuration (No Wi-Fi Root Patches Required)

[**SimpleBrcm**](https://github.com/laobamac/SimpleBCM) by laobamac is a Lilu plugin that restores Broadcom Wi-Fi support in macOS Sonoma, Sequoia and Tahoe through OpenCore injection.

Unlike the traditional OCLP method, SimpleBrcm uses the native `IOSkywalkFamily` framework of each macOS version. It does not require modifying the system volume or applying Wi-Fi root patches.

### Supported macOS Versions

- macOS Sonoma 14
- macOS Sequoia 15
- macOS Tahoe 26

### Installation

1. Download the latest release of [SimpleBrcm](https://github.com/laobamac/SimpleBCM/releases).
2. Copy `SimpleBrcm.kext` to `EFI/OC/Kexts`.
3. Add the kext to `Kernel -> Add` using the supplied `config.fragment.plist`.
4. Set `MinKernel` to `23.0.0`
5. Delete the following kexts from `EFI/OC/Kexts` and their corresponding entries from `config.plist`:
   - `IOSkywalkFamily.kext`
   - `IO80211FamilyLegacy.kext`
   - `AirPortBrcmNIC.kext`
6. Remove or disable any existing `Kernel -> Block` entry for `com.apple.iokit.IOSkywalkFamily`.
7. Validate your configuration using `ocvalidate` and reboot.

### Optional: AirportBrcmFixup

When using Broadcom cards that require `AirportBrcmFixup.kext`, you can place `SimplBrcm.kext` before AirportBrcmFixup and any Plugin kexts after (mind the `MinKernel`/`MaxKernel` settings):

![](/Users/5t33z0/Desktop/simplbrcm.png)

## 2. OCLP Configuration (Root Patches Required)

This method restores Broadcom Wi-Fi by injecting legacy Apple wireless drivers and applying OCLP Post-Install Root Patches.

### Supported Chipsets

#### Modern Broadcom cards

Affected chipsets include:

- `BCM94350` (including `BCM94352`)
- `BCM94360`
- `BCM43602`
- `BCM94331`
- `BCM943224`

Required kexts:

- `IOSkywalkFamily.kext`
- `IO80211FamilyLegacy.kext`
- `AirPortBrcmNIC.kext`
- `AirportBrcmFixup.kext`
- `AirPortBrcmNIC_Injector.kext`

#### Legacy Broadcom cards

Affected chipsets include:

- `BCM94322`
- `BCM94328`

Required kexts:

- `corecaptureElCap.kext`
- `IO80211ElCap.kext`

### Cards That Do Not Require OCLP Root Patches

Some Broadcom cards can work in macOS Sonoma and later without applying OCLP root patches through an alternative approach.

| Chipset | Device ID | Card names |
|---|---|---|
| BCM43602 | `0x14E4, 0x43BA` | BCM943602BAED, BCM943602CDP, BCM943602CS, DW1830 |
| BCM4350 | `0x14E4, 0x43A3` | BCM94350ZAE, DW1820A |

For these cards, you can also use [AppleBCMWLANCompanion](https://github.com/0xFireWolf/AppleBCMWLANCompanion) by 0xFireWolf. Follow its [Getting Started guide](https://github.com/0xFireWolf/AppleBCMWLANCompanion/blob/main/Documentation/GettingStarted.md) for configuration instructions.

### Third-Party Broadcom Cards

OCLP also supports certain third-party Broadcom chipsets through `AirportBrcmFixup.kext`:

- Device ID `pci12e4,4357` → BCM43225
- Device ID `pci12e4,43B1` → BCM4352
- Device ID `pci12e4,43B2` → BCM4352 (2.4 GHz)

### Configuration and EFI Adjustments

Add the required kexts to `EFI/OC/Kexts` and configure them under `Kernel -> Add`.

#### Modern Broadcom Cards

Obtain the following kexts:

- `IOSkywalkFamily.kext` and `IO80211FamilyLegacy.kext` from the [OCLP Wi-Fi payloads](https://github.com/dortania/OpenCore-Legacy-Patcher/tree/main/payloads/Kexts/Wifi).
- `AirportBrcmFixup.kext` from the [Acidanthera builds](https://dortania.github.io/builds/?product=AirportBrcmFixup&viewall=true).
- `AMFIPass.kext` from the [OCLP Acidanthera payloads](https://github.com/dortania/OpenCore-Legacy-Patcher/tree/main/payloads/Kexts/Acidanthera), if it is not already present.

`IO80211FamilyLegacy.kext` must include its `AirPortBrcmNIC.kext` plugin. `AirportBrcmFixup.kext` must include its `AirPortBrcmNIC_Injector.kext` plugin.

Use the following configuration as a reference:

![Modern Broadcom Wi-Fi configuration](https://github.com/5T33Z0/OC-Little-Translated/assets/76865553/ca04bf6f-71ee-47ed-b7b4-15359b88c17e)

#### Kernel/Block

Block `IOSkywalkFamily` for modern Broadcom cards:

![IOSkywalkFamily block configuration](https://github.com/5T33Z0/OC-Little-Translated/assets/76865553/54079541-ee2e-4848-bb80-9ba062363210)

**Note:** If you use a USB Ethernet dongle with the ECM protocol, also add [`ECM-Override.kext`](https://github.com/dortania/OpenCore-Legacy-Patcher/blob/main/payloads/Kexts/Misc/ECM-Override-v1.0.0.zip). This prevents a potential kernel panic caused by Apple's DriverKit stack using `IOSkywalk` for ECM dongles.

#### Kernel/Quirks

Enable the `DisableIoMapper` quirk. Once Wi-Fi is working, you can check whether it also works without this quirk.

#### Misc/Security

Set `SecureBootModel` to `Disabled` initially to rule out early kernel panics caused by security conflicts. You can re-enable it after confirming that everything works.

#### NVRAM

Under `NVRAM -> Add -> 7C436110-AB2A-4BBB-A880-FE41995C9F82`:

- Set `csr-active-config` to `03080000`.
- Optionally add `amfi=0x80` to `boot-args` if root patches cannot be applied. Remove it afterwards.
- Optionally add `-brcmfxbeta` if Wi-Fi is available but you cannot connect to access points after applying root patches.
- Optionally add `-amfipassbeta` if `AMFIPass.kext` does not load in the latest Sonoma beta.

### Verifying the Kexts

Before applying root patches, verify that the required kexts are loaded:

1. Save your configuration and reboot.
2. Reset NVRAM.
3. Boot into macOS Sonoma or later.
4. Open Terminal and run `kextstat`.
5. Search the output for the relevant kexts.

**Modern Broadcom:**

- `IOSkywalkFamily`
- `IO80211FamilyLegacy`
- `AirportBrcmFixup`
- `AirPort.BrcmNIC`
- `AMFIPass`

**Legacy Broadcom:**

- `corecaptureElCap`
- `IO80211ElCap`
- `AMFIPass`

If the kexts are missing, check your OpenCore configuration and EFI folder.

- If `AMFIPass` is not loaded, try adding `-amfipassbeta` to `boot-args` and reboot.
- If Wi-Fi is available but you cannot connect to access points, try `-brcmfxbeta` and reboot.

### Applying OCLP Root Patches

1. Download and install the [latest OpenCore Legacy Patcher release](https://github.com/dortania/OpenCore-Legacy-Patcher/releases).
2. Run OCLP and select **Post-Install Root Patch**.
3. If **Networking: Modern Wireless** or **Networking: Legacy Wireless** is available, select **Start Root Patching**.
4. Reboot once patching is complete.

If the expected patch option is unavailable, check the OCLP troubleshooting documentation.

## 3. Broadcom Bluetooth Configuration

Wi-Fi and Bluetooth are handled separately. Restoring Wi-Fi does not automatically restore Bluetooth functionality.

### NVRAM Configuration

Under `NVRAM -> Add -> 7C436110-AB2A-4BBB-A880-FE41995C9F82`, add the following entries. These are mandatory when using BrcmPatchRAM 2.7.0 and newer:

| Key | Type | Value |
|---|---|---|
| `bluetoothInternalControllerInfo` | Data | `00000000 00000000 00000000 0000` |
| `bluetoothExternalDongleFailed` | Data | `00` |

For macOS 14 and later, add `-btlfxboardid` to `boot-args` if Broadcom Bluetooth does not work despite the NVRAM entries being present when using BrcmPatchRAM 2.7.0 or newer.

Under `NVRAM -> Delete -> 7C436110-AB2A-4BBB-A880-FE41995C9F82`, add:

- `bluetoothInternalControllerInfo`
- `bluetoothExternalDongleFailed`

### Bluetooth Kext

For macOS 12 and later, add [`BlueToolFixup.kext`](https://github.com/acidanthera/BrcmPatchRAM) and set its `MinKernel` to `21.0.0`.

Adjust the `MinKernel` and `MaxKernel` settings of previously installed Bluetooth kexts accordingly.

![Bluetooth kext configuration](https://github.com/user-attachments/assets/6f3a6349-ade7-41a3-b7b1-0370401b5279)

### Troubleshooting Bluetooth

If Bluetooth stops working after updating kexts from [BrcmPatchRAM](https://github.com/acidanthera/BrcmPatchRAM) to version 2.7.0 or newer, try adding `-btlfxboardid` to `boot-args` on macOS 14 and later.

If `bluetoothExternalDongleFailed` changes from `00` to `01` after rebooting, you can use this [LaunchDaemon plist](https://gist.github.com/LBates2000/32be3c79dd987e72b98bb669c43fccbf) to reset the value to `00`.

## References

- [**SimpleBrcm**](https://github.com/laobamac/SimpleBCM)
- [**OpenCore Legacy Patcher**](https://github.com/dortania/OpenCore-Legacy-Patcher)
- [**BrcmPatchRAM**](https://github.com/acidanthera/BrcmPatchRAM)
- [**AirportBrcmFixup**](https://github.com/acidanthera/AirportBrcmFixup)
- [**AppleBCMWLANCompanion**](https://github.com/0xFireWolf/AppleBCMWLANCompanion)
- [**Wi-Fi patching discussion on InsanelyMac**](https://www.insanelymac.com/forum/topic/357087-macos-sonoma-wireless-issues-discussion/?do=findComment&comment=2809940)

## Credits

- Acidanthera for OpenCore and its kexts
- Dortania for OpenCore Legacy Patcher
- laobamac for SimpleBrcm
- 0xFireWolf for AppleBCMWLANCompanion
- Acquarius13 for investigating the required OCLP source changes
- deeveedee for pointing out the `brcmfxbeta` boot argument
