[![OpenCore Version](https://img.shields.io/badge/OpenCore_Version:-0.9.4+-success.svg)](https://github.com/acidanthera/OpenCorePkg) ![macOS](https://img.shields.io/badge/Supported_macOS:-≤15.x-white.svg)

# Installing macOS 13+ on Haswell/Broadwell systems

:construction: **OVERHAUL in PROGRESS**

## Introduction
Although installing and running macOS Ventura and newer on Wintel machines with Haswell/Broadwell CPUs is possible with OpenCore and the [**OpenCore Legacy Patcher**](https://github.com/dortania/OpenCore-Legacy-Patcher/tree/main) (OCLP), it’s not officially supported by Dortania – their support is limited to Apple Macs. Since no Hackintosh guide exists, I created this one in order to bridge the gap. I wrote it based on my experiences, analyzing the config, EFI folder, and logs after building OpenCore with OCLP for a Haswell system.

## Scope and limitations
This guide is intended to provide general information for adjusting your EFI and `config.plist` to install and run macOS Ventura and newer on unsupported Wintel systems. It is not a comprehensive configuration guide. Please refrain from using the "report issue" function to seek individualized assistance for fixing your config. Such issue reports will be closed immediately.

## Current status

| ⚠️ Current Status |
|:----------------------------|
| No iGPU patches for macOS Tahoe available. |

Check the links below for in-depth documentation about components/features that have been removed from macOS 12 and newer and the impact this has on systems prior to Kaby Lake. Keep in mind that this documentation targets real Macs, so certain issues may not apply to Wintel systems:

- [Status of OpenCore Legacy Patcher Support for macOS Tahoe](https://github.com/dortania/OpenCore-Legacy-Patcher/issues/1167)  
- [Status of OpenCore Legacy Patcher Support for macOS Sequoia](https://github.com/dortania/OpenCore-Legacy-Patcher/issues/1136)  
- [Status of OpenCore Legacy Patcher Support for macOS Sonoma](https://github.com/dortania/OpenCore-Legacy-Patcher/issues/1076)  
- [Status of OpenCore Legacy Patcher Support for macOS Ventura](https://github.com/dortania/OpenCore-Legacy-Patcher/issues/998)  
- [Legacy Metal Support and macOS Ventura–Sequoia](https://github.com/dortania/OpenCore-Legacy-Patcher/issues/1008)  
- [Legacy Non-Metal Support and macOS Big Sur–Sequoia](https://github.com/dortania/OpenCore-Legacy-Patcher/issues/108)

## Technical background

### How Haswell/Broadwell systems are affected
In macOS Ventura, support for CPU families prior to Kaby Lake was dropped. For Haswell/Broadwell CPUs this mainly affects integrated graphics and Metal support.

The approach described in this guide prepares the OpenCore configuration with the required patches, settings, and kexts for installing and running macOS Ventura+ and then adds iGPU/GPU drivers post-install using OpenCore Legacy Patcher.


## Preparations
This is what you need to know before attempting to install macOS Ventura on unsupported systems:

- :warning: **Backup** your working EFI folder on a FAT32 formatted USB Flash Drive just in case something goes wrong because we have to modify the config and content of the EFI folder.
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
- **Other**: Check the links below for in-depth documentation about components/features that have been removed from macOS 12 and newer and the impact this has on systems prior to Kaby Lake. But keep in mind that this was written for real Macs so certain issues don't apply to Wintel systems:
	- [Status of OpenCore Legacy Patcher Support for macOS Tahoe](https://github.com/dortania/OpenCore-Legacy-Patcher/issues/1167)
 	- [Status of OpenCore Legacy Patcher Supoort for macOS Sequoia](https://github.com/dortania/OpenCore-Legacy-Patcher/issues/1136) 
	- [Status of OpenCore Legacy Patcher Support for macOS Sonoma](https://github.com/dortania/OpenCore-Legacy-Patcher/issues/1076)
	- [Status of OpenCore Legacy Patcher Support for macOS Ventura](https://github.com/dortania/OpenCore-Legacy-Patcher/issues/998)
	- [Legacy Metal Support and macOS Ventura - Sequoia](https://github.com/dortania/OpenCore-Legacy-Patcher/issues/1008)
	- [Legacy Non-Metal Support and macOS Big Sur - Sequoia](https://github.com/dortania/OpenCore-Legacy-Patcher/issues/108)

### Update OpenCore and kexts
Update OpenCore and kexts to the latest version to maximize compatibility with macOS. To check which version of OpenCore you're currently using, run the following commands in Terminal:

```shell
nvram 4D1FDA02-38C7-4A6A-9CC6-4BCCA8B30102:opencore-version
```

For updating OpenCore and kext easily, you can use OpenCore Auxiliary Tools (OCAT) [Instructsions](https://github.com/5T33Z0/OC-Little-Translated/blob/main/Content/D_Updating_OpenCore/Updating_OC.md)

### EFI and Config Changes

Use this [config.plist](/Haswell-Broadwell_OCLP_Wintel_Patches.plist) which contains all the settings below. You can cross-reference or copy entries directly from it.

## 1. Booter Section – Add Board-Id check Skip

**Purpose**: Skips macOS board-id verification, allowing you to:
- Boot macOS with the native SMBIOS best suited for your CPU (instead of spoofed models)
- Receive system updates on unsupported hardware (when combined with RestrictEvents kext)

<details>
<summary><b>Instructions</b> (click to expand)</summary><br>

1. Open your config.plist in ProperTree
2. Copy the following code (click the copy button):
	```xml
	<dict>
		<key>Arch</key>
		<string>x86_64</string>
		<key>Comment</key>
		<string>Skip Board ID check</string>
		<key>Count</key>
		<integer>0</integer>
		<key>Enabled</key>
		<true/>
		<key>Find</key>
		<data>AFAAbABhAHQAZgBvAHIAbQBTAHUAcABwAG8AcgB0AC4AcABsAGkAcwB0</data>
		<key>Identifier</key>
		<string>Apple</string>
		<key>Limit</key>
		<integer>0</integer>
		<key>Mask</key>
		<data></data>
		<key>Replace</key>
		<data>AC4ALgAuAC4ALgAuAC4ALgAuAC4ALgAuAC4ALgAuAC4ALgAuAC4ALgAu</data>
		<key>ReplaceMask</key>
		<data></data>
		<key>Skip</key>
		<integer>0</integer>
	</dict>
	```
3. Paste it in the `Booter/Patch` Section (CMD+V):<br> <img width="1146" height="508" alt="Board-ID-Skip" src="https://github.com/user-attachments/assets/751068e7-0d67-4ff3-a6ce-46a2a596a734" />

Gif-Animation:<br> ![Booter_patch](https://github.com/user-attachments/assets/e00333d1-cc31-4f50-ada2-4d48c73d0eeb)

</details>

---

## 2. DeviceProperties – iGPU Configuration

**Location**: `DeviceProperties/Add/PciRoot(0x0)/Pci(0x2,0x0)`

**Supported iGPUs**: Intel HD 4200/4400/4600, HD 5000/5100/5200/5600, Iris Pro 6200

ℹ️ This section ensures you're using the correct `AAPL,ig-platform-id`. If using iGPU for display output, you may need additional framebuffer properties - consult the OpenCore Install Guide for complete patches.

<details>
<summary><b>Desktop - Haswell Systems (click to expand)</b></summary>

### Haswell with dedicated GPU (Headless configuration)
**When to use**: iMac SMBIOS + Haswell CPU + iGPU enabled + dedicated GPU handles display

```
AAPL,ig-platform-id: 04001204
device-id: 12040000
```
**Note**: `device-id` only required for HD 4400

### Haswell iGPU for display
**When to use**: Desktop PC where the iGPU directly drives your monitor

```
AAPL,ig-platform-id: 0300220D
device-id: 12040000
```
**Note**: `device-id` only required for HD 4400

</details>

<details>
<summary><b>Desktop - Broadwell Systems (click to expand)</b></summary>

### Broadwell iGPU for display
**When to use**: Desktop PC with Broadwell CPU where iGPU drives your monitor

```
AAPL,ig-platform-id: 07002216
device-id: 12040000
```
**Note**: `device-id` only required for HD 4400

</details>

<details>
<summary><b>Laptop/NUC Systems (click to expand)</b></summary>

Laptop and NUC configurations require specific framebuffer patches depending on your exact iGPU model. Refer to the appropriate guide:

- **Haswell Laptops/NUCs**: [OpenCore Install Guide - Haswell](https://dortania.github.io/OpenCore-Install-Guide/config-laptop.plist/haswell.html#add-2)
- **Broadwell Laptops/NUCs**: [OpenCore Install Guide - Broadwell](https://dortania.github.io/OpenCore-Install-Guide/config-laptop.plist/broadwell.html#add-2)

Different combinations of `AAPL,ig-platform-id` and `device-id` may be required based on your specific hardware.

</details>

---

## 3. Kernel - Kexts

**Location**: `Kernel/Add` in config.plist and `EFI/OC/Kexts` folder

### Required Kexts (All Users)

Add these kexts to both your config and EFI folder:

| Kext | MinKernel | Purpose |
|------|-----------|---------|
| [**AutoPkgInstaller**](https://github.com/dortania/OpenCore-Legacy-Patcher/tree/main/payloads/Kexts/Acidanthera) | - | Applies root patches automatically during macOS installation (requires [prepared installer](/Guides/Auto-Patching.md)) |
| [**AMFIPass**](https://github.com/dortania/OpenCore-Legacy-Patcher/tree/main/payloads/Kexts/Acidanthera) | `21.0.0` | Allows booting macOS 12+ without fully disabling AMFI |
| [**RestrictEvents**](https://github.com/acidanthera/RestrictEvents) | `20.4.0` | Enables OTA updates with VMM spoofing |

### Optional Kexts

<details>
<summary><b>FeatureUnlock - Unlock macOS features (click to expand)</b></summary>

[**FeatureUnlock**](https://github.com/acidanthera/FeatureUnlock) unlocks macOS features normally restricted to newer Mac models:
- Sidecar
- AirPlay to Mac
- Universal Control

**Note**: Available features depend on your chosen SMBIOS. Check the [FeatureUnlock documentation](https://github.com/acidanthera/FeatureUnlock) for compatibility. Use the `-disable_sidecar_mac` boot-arg to disable these patches if needed.

</details>

<details>
<summary><b>WiFi Support for macOS Sonoma (click to expand)</b></summary>

**Only needed if you have a Broadcom or Atheros WiFi card** and are running macOS Sonoma.

### Supported WiFi Cards

**Modern Broadcom**: BCM94350, BCM94360, BCM43602, BCM94331, BCM943224  
**Legacy**: Atheros chipsets, Broadcom BCM94322, BCM94328

### Required Kexts

Add both kexts with `MinKernel: 23.0.0`:

1. [**IOSkywalk.kext**](https://github.com/dortania/OpenCore-Legacy-Patcher/blob/e21efa975c0cf228cb36e81a974bc6b4c27c7807/payloads/Kexts/Wifi/IOSkywalkFamily-v1.0.0.zip)
2. [**IO80211FamilyLegacy.kext**](https://github.com/dortania/OpenCore-Legacy-Patcher/blob/e21efa975c0cf228cb36e81a974bc6b4c27c7807/payloads/Kexts/Wifi/IO80211FamilyLegacy-v1.0.0.zip)
   - **Important**: This contains `AirPortBrcmNIC.kext` as a plugin - ensure it's also enabled in your config

### Additional Required Step

You must also block macOS's native IOSkywalk kext. Add to `Kernel/Block`:

```
Identifier: com.apple.iokit.IOSkywalkFamily
Strategy: Exclude
MinKernel: 23.0.0
Enabled: true
```

**Reference**: [Complete WiFi Patching Guide](/Enable_Features/WiFi_Sonoma.md)

</details>

### Kexts to Disable

If present in your config, **disable** these kexts:
- **CPUFriend**
- **CPUFriendDataProvider**

ℹ️ When changing SMBIOS, it's recommended to regenerate a new CPUFriendDataProvider.kext for optimal power management.

---

## 4. Kernel - CPU Emulation

**⚠️ HEDT WORKSTATIONS ONLY** 

**Skip this entire section** unless you have a High-End Desktop Workstation with Haswell-E or Broadwell-E CPU. Do NOT add these settings to regular Desktop, Laptop, or NUC configs.

<details>
<summary><b>Haswell-E / Broadwell-E Users (click to expand)</b></summary>

**Location**: `Kernel/Emulate`

### Haswell-E CPUs
```
Cpuid1Data: C3060300 00000000 00000000 00000000
Cpuid1Mask: FFFFFFFF 00000000 00000000 00000000
```

### Broadwell-E CPUs
```
Cpuid1Data: D4060300 00000000 00000000 00000000
Cpuid1Mask: FFFFFFFF 00000000 00000000 00000000
```

</details>

---

## 5. Kernel - Patches

**Location**: `Kernel/Patch`

**How to add**: Copy complete patch entries from [OCLP's config.plist](https://github.com/dortania/OpenCore-Legacy-Patcher/blob/main/payloads/Config/config.plist) including all fields (Comment, Enabled, MinKernel, MaxKernel, Find, Replace, etc.)

### Required Patches (All Users)

**Add and enable**:
- "Disable Library Validation Enforcement"
- "Disable _csr_check() in _vnode_check_signature"

ℹ️ The second patch may not be needed on all systems. Try without it first - if you encounter issues, add it back.

ℹ️ VMM board-id kernel patches are no longer required - RestrictEvents handles this via NVRAM.

<details>
<summary><b>Conditional Patches (click to expand)</b></summary>

### Force FileVault on Broken Seal
**Only needed if**: You're using FileVault encryption

### Fix PCI bus enumeration (Ventura)
### Fix PCI bus enumeration (Sonoma)

**Only needed if**: Internal PCIe devices (like WiFi/Bluetooth cards) show up as removable express cards in the menu bar:

![PCIe express card issue](https://github.com/user-attachments/assets/d362d81c-01f7-491e-98c9-cd9372f30eb1)

Add the patch that matches your macOS version.

</details>

---

## 6. Misc - Security Settings

**Location**: `Misc/Security`

```
SecureBootModel: Disabled
Vault: Optional
```

**Why**: Required when patching in graphics drivers for AMD and NVIDIA GPUs. 

ℹ️ Intel iGPU-only users may be able to use `SecureBootModel: Default` - try for yourself if you want to maintain some security features.

---

## 7. NVRAM - Settings

### Part A: Boot Configuration

**Location**: `NVRAM/Add/7C436110-AB2A-4BBB-A880-FE41995C9F82`

#### System Integrity Protection

**Modify existing `csr-active-config` key**:
- **Standard setup**: `03080000`
- **If using NVIDIA GPU**: `030A0000`

#### Boot Arguments - Required for All Users

**Add these boot-args**:
```
amfi_get_out_of_my_way=0x1
ipc_control_port_options=0
```

**Explanations**:
- `amfi_get_out_of_my_way=0x1`: Disables Apple Mobile File Integrity validation. Required for installing root patches with OCLP. ⚠️ Thanks to AMFIPass.kext, this is no longer needed for booting - only for applying patches. Note: Disabling AMFI causes [issues with 3rd party app access to cameras/microphones](https://github.com/5T33Z0/OC-Little-Translated/blob/main/13_Peripherals/Fixing_Webcams.md).
- `ipc_control_port_options=0`: Required for Intel HD Graphics. Fixes issues with Firefox and Electron-based apps like Discord.

<details>
<summary><b>AMD GPU Users - Additional boot-args (click to expand)</b></summary>

### Temporary Boot-Arg (Installation Only)

```
-radvesa
```

**Purpose**: Disables hardware acceleration and puts the card in VESA mode.

**When to use**: Only if your screen turns black after installing macOS 12+

**⚠️ CRITICAL**: After installing GPU drivers with OCLP, you **MUST disable this boot-arg** or you won't get graphics acceleration!

---

### AMD GPU with Headless SMBIOS

**Only if using**:
- SMBIOS: `iMacPro1,1` or `MacPro7,1` (SMBIOSes for CPUs without iGPU)
- GPU: Polaris or Vega cards (Navi is not supported by OCLP)

```
-wegnoigpu
```

**Purpose**: Disables the iGPU in macOS, letting the AMD GPU handle background rendering and compute tasks.

**Additional step**: You may need to add `unfairgva=` bitmask (values 1-7) to [address DRM issues](https://github.com/5T33Z0/OC-Little-Translated/tree/main/H_Boot-args#unfairgva-overrides).

</details>

<details>
<summary><b>NVIDIA GPU Users - Additional boot-args (click to expand)</b></summary>

### Kepler GPU Users (GTX 600/700 series)

**Good news**: Kepler GPUs work out of the box on macOS Monterey through Ventura. You don't need any special boot-args - they switch to VESA mode automatically during installation if needed.

---

### Non-Kepler NVIDIA GPU Users

#### Temporary Boot-Arg (Installation Only)

```
nv_disable=1
```

**Purpose**: Disables hardware acceleration during installation.

**When to use**: Only if your screen turns black after installing macOS Ventura

**⚠️ CRITICAL**: After installing GPU drivers with OCLP, you **MUST disable this boot-arg** or you won't get graphics acceleration!

---

#### Required Boot-Args for Non-Kepler Cards

```
ngfxcompat=1
ngfxgl=1
nvda_drv_vrl=1
```

**Explanations**:
- `ngfxcompat=1`: Ignores compatibility check in `NVDAStartupWeb`
- `ngfxgl=1`: Disables Metal support, uses OpenGL for rendering instead
- `nvda_drv_vrl=1`: Enables NVIDIA Web Drivers

---

#### If Black Screen Occurs After Driver Installation

```
agdpmod=vit9696
```

**Purpose**: Disables board-id check. Fixes black screens that can occur after installing NVIDIA Web Drivers.

</details>

<details>
<summary><b>FeatureUnlock Users (click to expand)</b></summary>

```
-disable_sidecar_mac
```

**Purpose**: Disables Sidecar/AirPlay/Universal Control patches from FeatureUnlock.

**When to use**: If you installed FeatureUnlock but don't want these specific features enabled, or if they're causing issues.

</details>

---

### Part B: OCLP and RestrictEvents Configuration

**Location**: `NVRAM/Add/4D1FDA02-38C7-4BCCA8B30102`

**Add these three new keys**:

| Key | Type | Value | Purpose |
|-----|------|-------|---------|
| `OCLP-Settings` | String | `-allow_amfi` | OCLP configuration settings |
| `revblock` | String | `media` | Blocks mediaanalysisd service (fixes graphical issues with Metal 1 GPUs on Ventura+) |
| `revpatch` | String | `sbvmm,asset` | Enables OTA system updates and content caching on unsupported hardware |

**Reference**: Check [RestrictEvents documentation](https://github.com/acidanthera/RestrictEvents) for detailed explanations of `revblock` and `revpatch` parameters.

---

### Part C: NVRAM Delete Entries

**Location**: `NVRAM/Delete/4D1FDA02-38C7-4BCCA8B30102`

**Add these strings to the array**:
```
OCLP-Settings
revblock
revpatch
```

**Purpose**: Ensures old NVRAM values are cleared before writing new ones. This prevents you from needing to perform manual NVRAM resets every time you change these settings in the `Add` section.

---

## 8. UEFI - Drivers

**Location**: `UEFI/Drivers` in config.plist and `EFI/OC/Drivers` folder

**Add**:
- `ResetNvramEntry.efi`

**How to add in config**:
```
Path: ResetNvramEntry.efi
Enabled: true
```

**Purpose**: Adds a boot menu entry to perform NVRAM reset without changing the boot drive order.

**Requirement**: Your system must have a UEFI BIOS (not legacy BIOS).

---

## ✓ Verification Checklist

After making all changes:

- [ ] **Validate config**: Run `ocvalidate` or use OCAuxiliaryTools/ProperTree to check for errors
- [ ] **Verify MinKernel settings**: Ensure kexts like AMFIPass and RestrictEvents have correct `MinKernel` values
- [ ] **Check kext plugins**: If using IO80211FamilyLegacy, ensure `AirPortBrcmNIC.kext` is enabled
- [ ] **Backup working EFI**: Make a copy of your current working EFI before rebooting with changes
- [ ] **Review GPU boot-args**: Make sure you only added boot-args relevant to YOUR GPU (not all of them)
- [ ] **HEDT check**: If you're NOT using Haswell-E/Broadwell-E, ensure `Kernel/Emulate` is empty

**If components don't work after patching**: Consult [OCLP's patch documentation](https://github.com/dortania/OpenCore-Legacy-Patcher/blob/main/docs/PATCHEXPLAIN.md) for additional settings or troubleshooting steps.
