# Enabling Intel Wi-Fi and Bluetooth in macOS

**TABLE of CONTENTS**

- [Overview](#overview)
- [1. Intel Wi-Fi: macOS High Sierra – Sonoma](#1-intel-wi-fi-macos-high-sierra--sonoma)
  - [itlwm.kext vs. AirportItlwm.kext](#itlwmkext-vs-airportitlwmkext)
  - [Installation](#installation)
  - [Multiple macOS Versions](#multiple-macos-versions)
- [2. Intel Wi-Fi: macOS Sequoia – Tahoe](#2-intel-wi-fi-macos-sequoia--tahoe)
  - [Method 1: Native AirportItlwm (No Wi-Fi Root Patches Required)](#method-1-native-airportitlwm-no-wi-fi-root-patches-required)
    - [Supported macOS Versions](#supported-macos-versions)
    - [Installation](#installation-1)
    - [Migrating from a Root-Patched Installation](#migrating-from-a-root-patched-installation)
    - [Multiple macOS Versions](#multiple-macos-versions-1)
    - [Final Checks](#final-checks)
  - [Method 2: Traditional OCLP Root-Patch Method](#method-2-traditional-oclp-root-patch-method)
    - [Required Components](#required-components)
    - [OpenCore Configuration](#opencore-configuration)
    - [Applying OCLP Root Patches](#applying-oclp-root-patches)
- [Intel Bluetooth](#intel-bluetooth)
  - [Required Kexts](#required-kexts)
  - [Required NVRAM Settings](#required-nvram-settings)
- [Troubleshooting Bluetooth](#troubleshooting-bluetooth)
- [Links](#links)
- [References](#references)
- [Credits](#credits)

---

## Overview

Intel Wi-Fi support in macOS is provided by the [**OpenIntelWireless**](https://github.com/OpenIntelWireless) project, which offers two different kexts: `itlwm.kext` and `AirportItlwm.kext`.

The original OpenIntelWireless releases support macOS High Sierra through Sonoma. Starting with macOS Sequoia, changes to Apple's wireless frameworks require additional workarounds to restore native `AirportItlwm.kext` functionality.

There are two approaches for restoring native Intel Wi-Fi support in macOS Sequoia and Tahoe:

1. **Native AirportItlwm (No Wi-Fi Root Patches Required):** Uses [**laobamac's itlwm fork**](https://github.com/laobamac/itlwm), which restores native `AirportItlwm.kext` support in Sequoia and Tahoe without requiring OCLP Wi-Fi root patches.

2. **Traditional OCLP Root-Patch Method:** Downgrades `IOSkywalkFamily` to the Ventura version, injects the required legacy wireless components and uses the Ventura version of `AirportItlwm.kext`. OCLP Post-Install Root Patches are required.

For Bluetooth, the OpenIntelWireless project provides [**IntelBluetoothFirmware**](https://github.com/OpenIntelWireless/IntelBluetoothFirmware). Additional support for Apple's Bluetooth stack is provided by `BlueToolFixup.kext` included in the [**BrcmPatchRAM**](https://github.com/acidanthera/BrcmPatchRAM) package of kexts.

## 1. Intel Wi-Fi: macOS High Sierra – Sonoma

The [OpenIntelWireless](https://github.com/OpenIntelWireless/itlwm) project provides two different drivers for Intel wireless cards:

- `itlwm.kext`
- `AirportItlwm.kext`

Both drivers support the same range of Intel wireless adapters, but they integrate differently with macOS.

### itlwm.kext vs. AirportItlwm.kext

| Feature | `itlwm.kext` | `AirportItlwm.kext` |
|---|---|---|
| macOS integration | Ethernet interface | Native Wi-Fi interface |
| Wi-Fi management | HeliPort required | macOS Wi-Fi menu |
| Location Services | Limited | Supported |
| Apple Continuity | Limited | Handoff and Universal Clipboard |
| Airport Utility | Not supported | Supported |

**itlwm.kext**

`itlwm.kext` uses Apple's `IOEthernet` networking stack and exposes the wireless card as a regular network interface rather than a native Wi-Fi device.

It requires the companion application [**HeliPort**](https://github.com/joshcalvert47/HeliPort) to manage wireless connections.

**Advantages include**:

- Generally stable operation.
- Independent of Apple's native Wi-Fi framework.
- Useful when `AirportItlwm.kext` is incompatible with a particular macOS version.

Some Apple services that rely on native Wi-Fi integration, such as Location Services and Find My Mac, may be limited.

**AirportItlwm.kext**

`AirportItlwm.kext` integrates Intel wireless cards into Apple's native Wi-Fi stack.

**Advantages include**:

- Native Wi-Fi menu integration.
- Location Services support.
- Handoff and Universal Clipboard support.
- No separate Wi-Fi management application required.

However, `AirportItlwm.kext` is more sensitive to changes in Apple's wireless frameworks and requires a build matching the installed macOS version.

> [!IMPORTANT]
> 
> Never load `itlwm.kext` and `AirportItlwm.kext` simultaneously. Choose one driver.

### Installation

1. Visit the [OpenIntelWireless releases page](https://github.com/OpenIntelWireless/itlwm/releases).
2. Expand the **Assets** section of the relevant release.
3. Download the appropriate driver for your macOS version.
3. Extract the archive.
4. Copy the selected kext to `EFI/OC/Kexts`.
5. Add it to `Kernel -> Add` in your `config.plist`.
6. Make sure the entry is enabled.
7. If you use `itlwm.kext`, install JoshCalvert's Fork of [**HeliPort**](https://github.com/joshcalvert47/HeliPort).
8. Save your configuration and reboot.

For `AirportItlwm.kext`, make sure you download the build matching your macOS release.

### Multiple macOS Versions

If you run multiple macOS versions on the same machine, you can include several `AirportItlwm.kext` builds in your EFI folder by renaming them.

**For example**:

```text
EFI
└── OC
    └── Kexts
        ├── AirportItlwm_HighSierra.kext
        ├── AirportItlwm_Mojave.kext
        ├── AirportItlwm_Catalina.kext
        ├── AirportItlwm_BigSur.kext
        ├── AirportItlwm_Monterey.kext
        ├── AirportItlwm_Ventura.kext
        └── AirportItlwm_Sonoma.kext
```

Rename the kext bundles to distinguish between versions and add a separate entry for each one under `Kernel -> Add`.

Configure `MinKernel` and `MaxKernel` so that only the appropriate build loads for each macOS version.

| macOS Version | Kernel Version | Build |
|---|---|---|
| High Sierra | 17.x | High Sierra |
| Mojave | 18.x | Mojave |
| Catalina | 19.x | Catalina |
| Big Sur | 20.x | Big Sur |
| Monterey | 21.x | Monterey |
| Ventura | 22.x | Ventura |
| Sonoma 14.0–14.3 | 23.0.0–23.3.x | Sonoma 14.0 |
| Sonoma 14.4+ | 23.4.0–23.x | Sonoma |

Refer to the [OpenIntelWireless releases](https://github.com/OpenIntelWireless/itlwm/releases) for the available builds and version-specific requirements.

**Example**:

![Example configuration for Intel Wi-Fi kexts](https://github.com/user-attachments/assets/bf6ebd02-922b-42cf-9095-b103028ffa0d)

> [!NOTE]
> 
> Renaming kexts will affect update detection in tools like OCAT. You will have to download kext updates manually.

## 2. Intel Wi-Fi: macOS Sequoia – Tahoe

Apple's changes to the wireless framework in macOS Sequoia also affected `AirportItlwm.kext`, even though Apple never shipped Intel Wi-Fi cards in its Macs.

As a result, the original OpenIntelWireless builds cannot be used directly in Sequoia and Tahoe.

There are now two approaches to restoring native AirportItlwm support.

### Method 1: Native AirportItlwm (No Wi-Fi Root Patches Required)

[**laobamac's itlwm fork**](https://github.com/laobamac/itlwm) restores native `AirportItlwm.kext` support in macOS Sequoia and Tahoe without requiring OCLP Wi-Fi root patches.

It uses the native wireless frameworks provided by each macOS version rather than injecting the Ventura versions of `IOSkywalkFamily` and `IO80211FamilyLegacy`.

It also supports `AppleVTD`.

#### Supported macOS Versions

- macOS Sequoia 15
- macOS Tahoe 26

#### Installation

1. Visit [laobamac's itlwm releases](https://github.com/laobamac/itlwm/releases).
2. Download the appropriate `AirportItlwm.kext` build for your macOS version.
3. Copy the kext to `EFI/OC/Kexts`.
4. Add it to `Kernel -> Add` in your `config.plist`.
5. Make sure the entry is enabled.
6. Configure the kernel version settings.

**Kernel version settings:**

| macOS Version | MinKernel | MaxKernel |
|---|---|---|
| Sequoia 15 | `24.0.0` | `24.99.99` |
| Tahoe 26 | `25.0.0` | Leave empty |

For Tahoe, check the fork's release notes for any updated kernel compatibility requirements.

**Kernel -> Quirks**

Uncheck `DisableIoMapper`.

> [!IMPORTANT]
> 
> Do not install the old OCLP Wi-Fi dependencies when using this method. The new fork does not require:
>
> - `IOSkywalkFamily.kext`
> - `IO80211FamilyLegacy.kext`
> - The Ventura version of `AirportItlwm.kext`
> - OCLP Wi-Fi root patches
> - The associated `AMFIPass` configuration

#### Migrating from a Root-Patched Installation

If you previously used the traditional OCLP root-patch method, clean up the old configuration before switching.

**Step 1: Revert the old Wi-Fi root patches**

1. Open OpenCore Legacy Patcher.
2. Select **Post-Install Root Patch → Revert Root Patches**.
3. Reboot into macOS.

> [!CAUTION]
> 
> Only revert root patches if they were applied *exclusively* for Wi-Fi. If your system requires other root patches, for example for graphics acceleration, do not revert those patches. Keep the patches required by your other hardware.

**Step 2: Clean up the old EFI configuration**

Mount your EFI System Partition and make a backup of your working EFI folder.

Open `config.plist` and remove the old Wi-Fi patching configuration:

- `Kernel -> Block`
  - Remove the `com.apple.iokit.IOSkywalkFamily` blocking entry.
- `Kernel -> Add`
  - Remove `IOSkywalkFamily.kext`.
  - Remove `IO80211FamilyLegacy.kext`.
  - Remove the old Ventura `AirportItlwm.kext`.
  - Disable `AMFIPass.kext` if it was only added for the old Wi-Fi patching method and is not needed elsewhere.
- `DeviceProperties`
  - Remove the `IOName` property for your Wi-Fi card if present.

Keep unrelated kexts, quirks and NVRAM settings required by your system.

**Step 3: Install the new AirportItlwm**

Follow the installation instructions above and add the appropriate Sequoia or Tahoe build.

#### Multiple macOS Versions

If you want to include AirportItlwm variants for multiple macOS versions in the same EFI, rename the kexts accordingly:

```text
EFI
└── OC
    └── Kexts
        ├── AirportItlwm_Sequoia.kext
        └── AirportItlwm_Tahoe.kext
```

Add both kexts to `Kernel -> Add` and configure their kernel version ranges so that only the appropriate build loads.

#### Final Checks

Before rebooting, verify the following:

- Correct AirportItlwm build for Sequoia or Tahoe.
- `AirportItlwm.kext` is present in `EFI/OC/Kexts`.
- `AirportItlwm.kext` is enabled in `Kernel -> Add`.
- Old Wi-Fi patching entries have been removed.
- No conflicting `itlwm.kext` is enabled.
- Your EFI has been backed up.

Reboot into macOS and check whether Wi-Fi is working.

If you previously used `itlwm.kext`, you can also remove HeliPort from your login items if you no longer need it.

### Method 2: Traditional OCLP Root-Patch Method

This method restores compatibility by replacing Apple's newer wireless components with older versions from macOS Ventura.

The general patching procedure is:

1. Block the original `com.apple.iokit.IOSkywalkFamily`.
2. Inject the Ventura version of `IOSkywalkFamily.kext`.
3. Inject `IO80211FamilyLegacy.kext` and its required plugins.
4. Install the Ventura version of `AirportItlwm.kext`.
5. Apply the appropriate OCLP Post-Install Root Patches.
6. Reboot into macOS.

#### Required Components

| Component | Source |
|---|---|
| `IOSkywalkFamily.kext` | [OCLP Wi-Fi payloads](https://github.com/dortania/OpenCore-Legacy-Patcher/tree/main/payloads/Kexts/Wifi) |
| `IO80211FamilyLegacy.kext` | [OCLP Wi-Fi payloads](https://github.com/dortania/OpenCore-Legacy-Patcher/tree/main/payloads/Kexts/Wifi) |
| `AirportItlwm.kext` (Ventura build) | [OpenIntelWireless releases](https://github.com/OpenIntelWireless/itlwm/releases) |
| `AMFIPass.kext` | [OCLP Acidanthera payloads](https://github.com/dortania/OpenCore-Legacy-Patcher/tree/main/payloads/Kexts/Acidanthera) |

#### OpenCore Configuration

Add the required kexts to `EFI/OC/Kexts` and configure them under `Kernel -> Add`.

Use the following configuration:

**Kernel -> Block**

Block the native `IOSkywalkFamily` framework:

- `com.apple.iokit.IOSkywalkFamily`

Set the blocking entry to apply to macOS Sequoia and later.

**Kernel -> Add**

Inject:

- `IOSkywalkFamily.kext`
- `IO80211FamilyLegacy.kext`
- `AirportItlwm.kext` (Ventura build)
- `AMFIPass.kext`

Make sure the kext loading order and plugin dependencies are correct.

**Kernel -> Quirks**

Enable `DisableIoMapper` if required by your hardware and configuration.

**DeviceProperties**

Remove any `IOName` spoofing entry for the Intel Wi-Fi device if present, unless it is required for another purpose.

#### Applying OCLP Root Patches

1. Download and install [OCLP-Mod](https://github.com/laobamac/OCLP-Mod).
2. Boot into macOS Sequoia or Tahoe.
3. Open OCLP-Mod.
4. Select **Post-Install Root Patch**.
5. Apply the appropriate wireless root patches.
6. Reboot when prompted.

The root-patching process restores the Ventura wireless components required by the Ventura version of `AirportItlwm.kext`.

> [!NOTE]
> 
> This method uses a downgraded wireless stack. It is retained for compatibility with existing configurations, but the newer laobamac fork avoids this workaround.

## Intel Bluetooth

### Required Kexts

If your Intel Wi-FI/BT card is [supported](https://openintelwireless.github.io/itlwm/Compat.html), the following kexts are required for enabling Bluetooth in macOS 14+ (&rarr; [plist](/plist/Sonoma_WIFI_Intel.plist)):

Kext | Note | `MinKernel` | `MaxKernel`
-----|---------|:-----------:|:-----------:
[**`BlueToolFixup.kext`**](https://github.com/acidanthera/BrcmPatchRAM) | BT enabler for macOS 12 and newer | 21.0.0 | 
[**`IntelBluetoothFirmware.kext`**](https://github.com/lshbluesky/IntelBluetoothFirmware) | Intel BT FIrmware |  | 
**`IntelBTPatcher.kext`** | Included in IntelBTFirmware download
**`IntelBluetoothInjector.kext`** | Disable for macOS 12+ | |20.99.99

**Screenshot**:

<img width="1229" height="132" alt="btkexts" src="https://github.com/user-attachments/assets/c7c4ba75-df4d-4b9d-acc9-622eb899964b" />

### Required NVRAM Settings

Change the following Settings in **`NVRAM/Add/7C436110-AB2A-4BBB-A880-FE41995C9F82`**:

- Add the following entries:
	- **Key**: `bluetoothInternalControllerInfo`
	- **Type:** `Data`
	- **Value:** `00000000 00000000 00000000 0000`
	- **Key**: `bluetoothExternalDongleFailed`
  	- **Type:** `Data`
  	- **Value:** `00`
- **Optional**: Change `csr-active-config` to `03080000`, if your system requires Root Patches by OCLP, otherwise leave it at `00000000´.
- **Optional**: Add `-amfipassbeta` to `boot-args` if `AMFIPass.kext` is not loaded.

**Screenshot:**

<img width="813" height="270" alt="NVRAM_del" src="https://github.com/user-attachments/assets/47da0051-0342-4d0a-8011-2af5aecb796b" />

Add the following entries t **`NVRAM/Delete/7C436110-AB2A-4BBB-A880-FE41995C9F82`**:

- `csr-active-config`
- `boot-args`
- `bluetoothInternalControllerInfo`
- `bluetoothExternalDongleFailed`

**Screenshot:**

<img width="813" height="270" alt="NVRAM_del" src="https://github.com/user-attachments/assets/b2e2ca24-8792-4708-a65a-3ea3e56735d7" />

> [!NOTE]
> 
> - Make sure to perform an NVRAM reset after applying these settings and after switching macOS versions (if you have different versions of macOS installed)
> - I have noticed that it can take like 10 seconds or so until BT is available after applying these changes.
> - For using Intel Bluetooth in macOS Monterey and newer, [**read this**](https://openintelwireless.github.io/IntelBluetoothFirmware/FAQ.html#what-additional-steps-should-i-do-to-make-bluetooth-work-on-macos-monterey-and-newer).

## Troubleshooting Bluetooth

If Bluetooth is not working after adding the two kexts, do the following:

- Open Terminal and enter `NVRAM -p`
- If the output contains `bluetoothExternalDongleFailed=%01`, you need to change it to `%00` in NVRAM.
- Enter: `sudo NVRAM bluetoothExternalDongleFailed=%00`
- Followed by `sudo pkill bluetoothd` to kill and restart the Bluetooth stack

> [!TIP]
>
> In cases where `bluetoothExternalDongleFailed` changes from `00` to `01` after rebooting, you can use this [plist](https://gist.github.com/LBates2000/32be3c79dd987e72b98bb669c43fccbf). It acts as a LaunchDaemon to reset the value back to `00`. 

## Links
- [minipci.biz](https://minipci.biz/) – Find macOS-compatible MiniPCIe WiFi and BT cards.
- [Wireless USB Adapter](https://github.com/chris1111/Wireless-USB-OC-Big-Sur-Adapter) – Tool for enabling numerous Realtek-based WiFi Dongles in macOS 10.15 and newer.

## References

- [OpenIntelWireless/itlwm](https://github.com/OpenIntelWireless/itlwm)
- [OpenIntelWireless/IntelBluetoothFirmware](https://github.com/OpenIntelWireless/IntelBluetoothFirmware)
- [OpenIntelWireless/HeliPort](https://github.com/OpenIntelWireless/HeliPort)
- [laobamac/itlwm](https://github.com/laobamac/itlwm)
- [laobamac/OCLP-Mod](https://github.com/laobamac/OCLP-Mod)
- [Acidanthera/BrcmPatchRAM](https://github.com/acidanthera/BrcmPatchRAM)
- [OpenCore Legacy Patcher](https://github.com/dortania/OpenCore-Legacy-Patcher)

## Credits

- OpenIntelWireless for `itlwm`, `AirportItlwm` and `IntelBluetoothFirmware`
- laobamac for the Sequoia and Tahoe AirportItlwm implementation and OCLP-Mod
- Acidanthera for OpenCore and BlueToolFixup
- Dortania for OpenCore Legacy Patcher
- The Hackintosh community for testing and documenting compatibility workarounds
