[![OpenCore Version](https://img.shields.io/badge/OpenCore_Version:-0.9.4+-success.svg)](https://github.com/acidanthera/OpenCorePkg) ![macOS](https://img.shields.io/badge/Supported_macOS:-≤26b2-white.svg)

# Installing macOS Tahoe on Kaby Lake and newer

:construction: WORK in PROGRESS

<details>
<summary><b>TABLE of CONTENTS</b> (Click to reveal)</summary><br>

**TABLE of CONTENTS**

- [Overview](#overview)
- [How Kaby Lake systems and newer are affected](#how-kaby-lake-systems-and-newer-are-affected)
	- [Supported SMBIOSes](#supported-smbioses)
	- [Workarounds](#workarounds)
	- [Audio Issues](#audio-issues)
	- [GPU issues](#gpu-issues)
	- [Networking](#networking)
- [Config Adjustments](#config-adjustments)
- [Kext Adjustments](#kext-adjustments)
- [Testing the changes](#testing-the-changes)
- [Getting and installing macOS Tahoe](#getting-and-installing-macos-tahoe)
- [Credits](#credits)

</details>

## Overview
Although it is possible to install and run macOS Tahoe and newer on machines with 8th/9th Gen Intel Core CPUs (Coffee Lake), some config adjustments have to be implemented in order to install Apple's last OS for Intel-based systems. Since macOS Tahoe is not supported by OCLP yet. In consequence, any system that previously required root patches for re-enabling Graphics or Wi-Fi won't work.

| ⚠️ Important Status Updates |
|:----------------------------|
| Don't install macOS Tahoe if you don't have a compatible iGPU/GPU in your system!
| No official OCLP Support for Kaby Lake or newer is available yet. There's an [OCLP Mod](https://github.com/laobamac/OCLP-Mod) which can reinstall AppleHDA so on-board audio works again.

- **Further Info**:
	- [Status of OpenCore Legacy Patcher Support for macOS Tahoe](https://github.com/dortania/OpenCore-Legacy-Patcher/issues/1167)
 	- [Status of OpenCore Legacy Patcher Supoort for macOS Sequoia](https://github.com/dortania/OpenCore-Legacy-Patcher/issues/1136) 
	- [Status of OpenCore Legacy Patcher Support for macOS Sonoma](https://github.com/dortania/OpenCore-Legacy-Patcher/issues/1076)
	- [Status of OpenCore Legacy Patcher Support for macOS Ventura](https://github.com/dortania/OpenCore-Legacy-Patcher/issues/998)
	- [Legacy Metal Support and macOS Ventura - Sequoia](https://github.com/dortania/OpenCore-Legacy-Patcher/issues/1008)
	- [Legacy Non-Metal Support and macOS Big Sur - Sequoia](https://github.com/dortania/OpenCore-Legacy-Patcher/issues/108)

## How Kaby Lake systems and newer are affected

### Supported SMBIOSes
With the beta release of macOS 26, Apple dropped support for most of the remaining Intel-based Macs. The only remaining officially supported SMBIOSes are: `MacBookPro16,1`/`MacBookPro16,4` (I've noticed that MacBookPro16,2 works as well) and `iMac20,1`/`iMac20,2`.

### Workarounds
Luckily for us, the board-id check skip booter patch still works so you can still and run macOS Tahoe on Kaby Lake and newer. Since `RestrictEvets.kext` and sbvmm patch also still works, System Updates are possible.

### Audio Issues
`AppleHDA` was removed from Tahoe beta 2, so AppleALC is useless any on-board audio CODEC for now. So Sound won't work unless it's coming from the audio device of your GPU via HDMI or DP. As a workaround you could try VoodooHDA.

### GPU issues
If your system is using the iGPU for driving the graphics output, the changes that have to be made to your existing `config.plist` are rather small. On the other hand, if you are using a GPU, it seems that only Navi GPUs are currently working OOB. Since no OCLP update for Tahoe is available yet, everyone using any legacy GPUs should wait before installing macOS Tahoe.

### Networking
It seems that Apple has made changes to AppleVTD so that any NIC requiring AppleVTD might or might not work based on chipset and used kext (the kext has to support AppleVTD as well).

## Config Adjustments

Listed in the table below are the required config changes for installing/booting macOS Tahoe on systems with Intel Kaby Lake or newer.

Config Section | Action | Description
:-------------:| ------ | ------------
**`Booter/Patch`**| **Add** and **enable** the following Booter patch from OCLP's config: <ul> <li> [**"Skip Board ID check"**](https://github.com/dortania/OpenCore-Legacy-Patcher/blob/main/payloads/Config/config.plist#L220-L243) | <ul><li> Skips board-id check. <li> In combination with ResterictEvents kext, this allows: <ul> <li> Booting macOS with unsupported, native SMBIOS best suited for your CPU <li> Installing Sytsem Updates on unsupported systems </ul> <li> More [Details](https://github.com/5T33Z0/OC-Little-Translated/tree/main/09_Board-ID_VMM-Spoof#booter-patches)
**`DeviceProperties/Add`**|**PciRoot(0x0)/Pci(0x2,0x0)** – Todo…<br>**PciRoot(0x0)/Pci(0x1c,0x0)/Pci(0x0,0x0)**<ul><li>**Key**: `IOName`<li>**Type**: String <li>**Value**: pci14e4,43a0 | <ul><li> The IOName key is only required for triggering OCLP modern Wi-Fi patches when using Intel cards. Since OCLP for Tahoe is not available yet, it's uncertain if this method will work in the future.
**`Kernel/Add`** and <br>**`EFI/OC/Kexts`** |**Add the following Kexts**:<ul><li>[**AMFIPass**](https://github.com/dortania/OpenCore-Legacy-Patcher/tree/main/payloads/Kexts/Acidanthera) (`MinKernel`: `21.0.0`)<li> [**RestrictEvents**](https://github.com/acidanthera/RestrictEvents) (`MinKernel`: `20.4.0`) <li> [**FeatureUnlock**](https://github.com/acidanthera/FeatureUnlock) (optional) </ul> </ul> **WiFi** (optional) <ul><li>[**IOSkywalk.kext**](https://github.com/dortania/OpenCore-Legacy-Patcher/blob/e21efa975c0cf228cb36e81a974bc6b4c27c7807/payloads/Kexts/Wifi/IOSkywalkFamily-v1.0.0.zip) (`MinKernel`: `23.0.0`, `MaxKernel`: `24.9.9`) <li>[**IO80211FamilyLegacy.kext**](https://github.com/dortania/OpenCore-Legacy-Patcher/blob/e21efa975c0cf228cb36e81a974bc6b4c27c7807/payloads/Kexts/Wifi/IO80211FamilyLegacy-v1.0.0.zip) (contains `AirPortBrcmNIC.kext`, ensure this is injected as well) (`MinKernel`: `23.0.0`, `MaxKernel`: `24.9.9`) </ul> **Disable the following Kexts** (if present): <ul><li> **CPUFriend** <li> **CPUFriendDataProvider**| <ul><li>**AutoPkgInstaller**: For applying root-patches during macOS installation automatically. Requires preparation of the installer ([**Details**](/Guides/Auto-Patching.md)). <li> **AMFIPass**: Beta kext from OCLP 0.6.7. Allows booting macOS 12+ without disabling AMFI. <li> **RestrictEvents**: Forces VMM SB model, allowing OTA updates for unsupported models on macOS 11.3 or newer. Requires additional NVRAM parameters. <li> **FeatureUnlock**: Unlocks AirPlay to Mac. <li> **WiFi Kexts**: For macOS Sonoma/Sequoia. Re-Enable modern WiFi: BCM94350, BCM94360, BCM43602, BCM94331 and BCM943224. Legacy WiFi: Atheros chipsets, Broadcom BCM94322, BCM94328.
**`Kernel/Block`**| Block `com.apple.iokit.IOSkywalkFamily`: <br> ![](https://user-images.githubusercontent.com/76865553/256150446-54079541-ee2e-4848-bb80-9ba062363210.png) <br> Set `MaxKernel` to `24.9.9`| Blocks macOS'es IOSkywalk kext, so the injected one will be used instead. Only required for "Modern" Wifi Cards (&rarr; [Wifi Patching Guide](/Enable_Features/WiFi_Sonoma.md)). 
**`Kernel/Patch`** | Add and enable the following Kernel Patches from [**OCLP**](https://github.com/dortania/OpenCore-Legacy-Patcher/blob/main/payloads/Config/config.plist) (apply `MinKernel` and `MaxKernel` settings as well): <ul> <li> **Force FileVault on Broken Seal** (optional) <li> **"Disable Library Validation Enforcement"**<li>**"Disable _csr_check() in _vnode_check_signature"** <li> **Fix PCI bus enumeration (Ventura)** <li> **Fix PCI bus enumeration (Sonoma)** </ul> **NOTE**: VMM board-id kernel patches are no longer required since `RestrictEvents` kext can enable the VMM board-id via `sbvmm` NVRAM entry! | <ul> <li> **Force FileVault on Broken Seal** is only required when using FileVault <li> **"Disable _csr_check() in _vnode_check_signature"** is not required on my Laptop, but on some Desktops it's needed. Try for yourself. <li> The "Fix PCI bus enumeration" patches fix internal PCIe devices showing up as express cards in the menu bar: ![Screenshot](https://github.com/user-attachments/assets/d362d81c-01f7-491e-98c9-cd9372f30eb1)
**`Misc/Security`**| <ul> <li>**SecureBootModel**: `Disabled` <li> **Vault**: `Optional`| Required when patching in graphics drivers for AMD and NVIDIA GPUs. Intel HD graphics might work with SecureBootModel set to `Default`. Try for yourself.
**`NVRAM/Add/...-4BCCA8B30102`** | **Add the following Keys**: <ul> <li> **Key**: `OCLP-Settings`<br>**Type**: String <br> **Value**: `-allow_amfi` <br> In preparation for when OCLP is available<li> **Key**: `revpatch` <br> **Type:** String <br> **Value**: `sbvmm,asset`| <ul> <li> Settings for OCLP and RestrictEvents.  <li>`sbvmm,asset` &rarr; Enables OTA updates and content caching (&rarr; Check RestrictEvents documentation for details)|
**`NVRAM/Delete/...-4BCCA8B30102`** (Array) | **Add the following Strings**: <ul> <li>  `OCLP-Settings` <li> `revblock` <li> `revpatch` | Deletes NVRAM for these parameters before writing them. Otherwise you would need to perform an NVRAM reset every time you change any of them in the corresponding `Add` section.  
**`NVRAM/Add/...-FE41995C9F82`** | **Change** **`csr-active-config`** to: <ul><li>**`03080000`** <li> When using an NVIDIA GPU, set it to: **`030A0000`** </ul> **Add the following**`boot-args`: <ul><li> **`amfi=0x80`** (only necessary if root patches can't be applied)<li> **`ipc_control_port_options=0`** <li> **`-disable_sidecar_mac`** </ul>**Optional boot-args for GPUs** (Select based on GPU Vendor): <ul> <li> **`-igfxvesa`** <li> **`-radvesa`** <li> **`nv_disable=1`** <li> **`ngfxcompat=1`**<li>**`ngfxgl=1`**<li> **`nvda_drv_vrl=1`** <li> **`agdpmod=vit9696`** | <ul> <li>**`amfi=0x80`**: Disables Apple Mobile File Integrity validation. Required for applying Root Patches with OCLP ~~and booting macOS 12+~~. :bulb: No longer needed for booting thanks to AMFIPass.kext – only for installing Root Patches with OCLP. Disabling AMFI causes issues with [3rd party apps' access to Mics and Cameras](https://github.com/5T33Z0/OC-Little-Translated/blob/main/13_Peripherals/Fixing_Webcams.md).<li> **`ipc_control_port_options=0`**: Required for Intel HD Graphics. Fixes issues with Firefox and electron-based apps like Discord. <li> **`-disable_sidecar_mac`**: For FeatureUnlock &rarr; Disables Sidecar/AirPlay/Universal Control patches. <li> **`-igfvesa`** (Intel iGPU): Disables Intel iGPU acceleration (optional). Might be required before re-installing Skylake iGPU drivers with OCLP <li> **`-radvesa`** (AMD only): Disables hardware acceleration and puts the card in VESA mode. Only required if your screen turns off after installing macOS 12+. Once you've installed the GPU drivers with OCLP, **disable it** so graphics acceleration works! <li> **`nv_disable=1`** (NVIDIA only): Disables hardware acceleration and puts the card in VESA mode. Only required if your screen turns off after installing macOS. Kepler Cards switch into VESA mode automatically without it. Once you've installed the GPU drivers with OCLP, **disable it** so graphics acceleration works! <li>**`ngfxcompat=1`** (NVIDIA only): Ignores compatibility check in `NVDAStartupWeb`. Not required for Kepler GPUs <li>**`ngfxgl=1`** (NVIDIA only): Disables Metal Spport so OpenGL is used for rendering instead. Not required for Kepler GPUs. <li> **`nvda_drv_vrl=1`** (NVIDIA only): Enables Web Drivers. Not required for Kepler GPUs. <li> **`agdpmod=vit9696`** &rarr; Disables board-id check. Useful if screen turns black after booting macOS which can happen after installing NVIDIA Webdrivers. <li> **`-wegnoigpu`** &rarr; Optional. Disables the iGPU in macOS. **ONLY** required when using an AMD GPU and an SMBIOS for a CPU without on-board graphics (i.e. `iMacPro1,1` or `MacPro7,1`) to let the GPU handle background rendering and other tasks. Requires Polaris or Vega cards to work properly (Navi is not supported by OCLP). Combine with `unfairgva=x` bitmask (x= 1 to 7) to [address DRM issues](https://github.com/5T33Z0/OC-Little-Translated/tree/main/H_Boot-args#unfairgva-overrides)
**`UEFI/Drivers`** and <br> **`EFI/OC/Drivers`**| <ul> <li> Add `ResetNvramEntry.efi` to `EFI/OC/Drivers` <li> And to your config:<br> ![resetnvram](https://github.com/5T33Z0/OC-Little-Translated/assets/76865553/8d955605-fb27-401f-abdd-2c616b233418) | Adds a boot menu entry to perform an NVRAM reset but without resetting the order of the boot drives. Requires a BIOS with UEFI support.
**`PlatformInfo/Generic`**|  **SMBIOS**: leave as is | Board-ID check skip will allow installing macOS on unsupported board-ids and RestrictEvents kext will enable receiving System Updates, so no need to change the SMBIOS!

## Kext Adjustments

In order for `USBMap.kext` to work in macOS Tahoe, some settings have to be adjusted. &rarr; [**Instructions**](/Enable_Features/USB_Tahoe.md) 

## Testing the changes
Once you've added the required kexts and made the necessary changes to your `config.plist`, save, reboot and perform an NVRAM Reset. If your system still boots fine after that, you can now prepare the system for installing macOS 13 and newer.

## Getting and installing macOS Tahoe
- For now, download the latest `InsatllAssistant.pkg` via the directlink [**listed here**](https://mrmacintosh.com/macos-tahoe-full-installer-database-download-directly-from-apple/)
- Once it's downloaded, run it
- It will place "Install macOS" app will be located in the "Programs" folder
- Create a new APFS Volume on your SSD/NVME
- Run the Tahoe beta installer App and install it on the newly created APFS volume
- This will prepare the installation (copy the files over to the newly created volume)
- Once that's done, the system will reboot and there will be a new entry in the boot menu "Install macOS" or similar
- Select it to continue with the install

## Credits
- Acidanthera for OpenCore, OCLP and numerous Kexts
- Corpnewt for MountEFI, GenSMBIOS and ProperTree
- dhinakg for AMFIPass
- Dortania for [OpenCore Legacy Patcher](https://github.com/dortania/OpenCore-Legacy-Patcher/releases) and [Guide](https://dortania.github.io/OpenCore-Legacy-Patcher/)
- Rehabman for Laptop framebuffer patches
