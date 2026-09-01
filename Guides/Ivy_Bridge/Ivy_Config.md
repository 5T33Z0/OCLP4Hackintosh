[![OpenCore Version](https://img.shields.io/badge/OpenCore_Version:-0.9.4+-success.svg)](https://github.com/acidanthera/OpenCorePkg) ![macOS](https://img.shields.io/badge/Supported_macOS:-≤15.x-white.svg)

# Installing macOS Ventura and newer on Ivy Bridge systems

## Config Edits

Listed below, you find the required modifications to prepare your config.plist and EFI folder for installing macOS Monterey or newer on Ivy Bridge systems. If this is over your head, there's an [accompanying plist](/plist/Ivy_Bridge_OCLP_Wintel_Patches.plist) that contains the necessary settings that you can use for cross-referencing. 

:bulb: If your system (or components thereof) doesn't work afterwards, please refer to OCLP's [patch documentation](https://github.com/dortania/OpenCore-Legacy-Patcher/blob/main/docs/PATCHEXPLAIN.md) and see if need additional settings or kexts.

Config Section | Action | Description
:-------------:|------- | ---------
**`ACPI/Add`** | Disable/Delete **`SSDT-PLUG`** or **`SSDT-XCPM`** if present. | We don't want the system to use XCPM on Ivy Bridge CPUs! 
**`Booter/Patch`**| **Add** and **enable** the following Booter patch from OCLP's config: <ul> <li> [**"Skip Board ID check"**](https://github.com/dortania/OpenCore-Legacy-Patcher/blob/main/payloads/Config/config.plist#L220-L243) | <ul><li> Skips board-id check. <li> In combination with ResterictEvents kext, this allows: <ul> <li> Booting macOS with unsupported, native SMBIOS best suited for your CPU <li> Installing Sytsem Updates on unsupported systems </ul> <li> More [Details](https://github.com/5T33Z0/OC-Little-Translated/tree/main/09_Board-ID_VMM-Spoof#booter-patches) <li> ⚠️ Clover users need to add `-no_compat_check` boot-arg instead!
**`DeviceProperties/Add`**| **PciRoot(0x0)/Pci(0x2,0x0)** – Verify/Adjust `AAPL,ig-platform-id` (optional, relevant for CPUs with integrated graphics only)<ul><li> **[Desktops](https://dortania.github.io/OpenCore-Install-Guide/config.plist/ivy-bridge.html#add-2)**: <ul><li> **07006201** (Empty Framebuffer) <li> **0A006601** (Default) <li> **05006201** (Alternative, if default causes issues) </ul></ul><ul><li> **[Laptops](https://dortania.github.io/OpenCore-Install-Guide/config-laptop.plist/ivy-bridge.html#add-2)**: <ul><li> **04006601** (1600x900 px or more)<li> **03006601** (1366x769 px or less) <li> **09006601** (Alternative if the other 2 don't work) </ul></ul><ul><li> **Intel NUCs** (or other USDTs): <ul><li> **0B006601** </ul></ul>**PciRoot(0x0)/Pci(0x16,0x0)** – Check requirement for spoofed **IMEI** device <ul> <li> **device-id**: 3A1C0000 | <ul> <li> **Empty Framebuffer**: For Desktops. Use this if (a) your CPU has an iGPU, (b) if a dedicated GPU is used for graphics and (c) if you are using an iMac SMBIOS. <li> **Default**: Use this if you have a PC and the iGPU is used for driving a display <li> **Alternative**: use if the default framebuffer patch causes graphical glitches. <li> **NUCs**: For Intel NUCs and other Ultra Slim Desktops (USDT), such as: HP 6300 Pro, HP 8300 Elite, etc. <li> **IMEI**: Only needed when using an Ivy Bridge CPU with a 6-series mainboard (ie. H61, B65, Q65, P67, H67, Q67, Z68) </ul> Refer to [**Intel HD FAQ**](https://github.com/acidanthera/WhateverGreen/blob/master/Manual/FAQ.IntelHD.en.md#intel-hd-graphics-25004000-ivy-bridge-processors) for more details. Remember: the FAQ displays the ig-platform-ids in Big Endian but for the config you need Little Endian!
**`Kernel/Add`** and <br>**`EFI/OC/Kexts`** |**Add the following Kexts**:<ul><li>[**AutoPkgInstaller**](https://github.com/dortania/OpenCore-Legacy-Patcher/tree/main/payloads/Kexts/Acidanthera)<li>[**AMFIPass**](https://github.com/dortania/OpenCore-Legacy-Patcher/tree/main/payloads/Kexts/Acidanthera) (`MinKernel`: `21.0.0`) <li>[**CryptexFixup**](https://github.com/acidanthera/CryptexFixup) (`MinKernel`: `22.0.0`)<li> [**RestrictEvents**](https://github.com/acidanthera/RestrictEvents) (`MinKernel`: `20.4.0`) <li>  [**AppleIntelCPUPowerManagement**](https://github.com/dortania/OpenCore-Legacy-Patcher/raw/main/payloads/Kexts/Misc/AppleIntelCPUPowerManagement-v1.0.0.zip) (`MinKernel`: `22.0.0`)<li> [**AppleIntelCPUPowerManagementClient**](https://github.com/dortania/OpenCore-Legacy-Patcher/raw/main/payloads/Kexts/Misc/AppleIntelCPUPowerManagementClient-v1.0.0.zip) (`MinKernel`: `22.0.0`) <li> [**FeatureUnlock**](https://github.com/acidanthera/FeatureUnlock) (optional) <li> [**CSVLFixup**](https://github.com/dortania/OpenCore-Legacy-Patcher/blob/main/payloads/Kexts/Acidanthera/CSLVFixup-v2.6.1.zip) (optional)</ul> **WiFi** (optional) <ul><li>[**IOSkywalk.kext**](https://github.com/dortania/OpenCore-Legacy-Patcher/blob/e21efa975c0cf228cb36e81a974bc6b4c27c7807/payloads/Kexts/Wifi/IOSkywalkFamily-v1.0.0.zip) (`MinKernel`: `23.0.0`)  <li>[**IO80211FamilyLegacy.kext**](https://github.com/dortania/OpenCore-Legacy-Patcher/blob/e21efa975c0cf228cb36e81a974bc6b4c27c7807/payloads/Kexts/Wifi/IO80211FamilyLegacy-v1.0.0.zip) (contains `AirPortBrcmNIC.kext`, ensure this is injected as well) (`MinKernel`: `23.0.0`) </ul> **Delete the following Kexts** from EFI/OC/Kexts and config (if present): <ul><li> **CPUFriend** <li> **CPUFriendDataProvider** </ul> | <ul><li>**AutoPkgInstaller**: For applying root-patches during macOS istallation automatically. Requires preparation of the installer ([**Details**](/Guides/Auto-Patching.md))<li> **AMFIPass**: Beta kext from OCLP 0.6.7. Allows booting macOS 12+ without disabling AMFI. <li> **Cryptexfixup**: Required for installing and booting macOS Ventura on systems without AVX 2.0 support (see [OCLP Support Issue #998](https://github.com/dortania/OpenCore-Legacy-Patcher/issues/998)) <li> **CSVLFixup**: Fixes issues with Music.app if Library Validation is disabled ([Details](https://github.com/dortania/OpenCore-Legacy-Patcher/issues/497))  <li> **RestrictEvents**: Forces VMM SB model, allowing OTA updates for unsupported models on macOS 11.3 or newer. Requires additional NVRAM parameters. <li> **AppleIntelCPUPowerManagement** kexts: Required for re-enabling SMC CPU Power Management ([more details](https://github.com/5T33Z0/OC-Little-Translated/tree/main/01_Adding_missing_Devices_and_enabling_Features/CPU_Power_Management/CPU_Power_Management_(Legacy)#re-enabling-acpi-power-management-in-macos-ventura))<li> **FeatureUnlock**: Enables Sidecar, AirPlay to Mac <li> **WiFi Kexts**: For macOS Sonoma. Re-Enable modern WiFi: BCM94350, BCM94360, BCM43602, BCM94331 and BCM943224. Legacy WiFi: Atheros chipsets, Broadcom BCM94322, BCM94328.
**`Kernel/Block`**| Block `com.apple.iokit.IOSkywalkFamily`: <br> ![](https://user-images.githubusercontent.com/76865553/256150446-54079541-ee2e-4848-bb80-9ba062363210.png)| Blocks macOS'es IOSkywalk kext, so the injected one will be used instead. Only required for "Modern" Wifi Cards (&rarr; [Wifi Patching Guide](/Enable_Features/WiFi_Sonoma.md)). 
**`Kernel/Emulate`** | Disable `DummyPowermanagement` (if enabled) | If you imject the Kexts to re-instatate ACPI CPU Power Management on macOS13+ while this setting is still enabled, the system will freeze 10 to 15 seconds after booting.
**`Kernel/Patch`** | Add and enable the following Kernel Patches from [**OCLP**](https://github.com/dortania/OpenCore-Legacy-Patcher/blob/main/payloads/Config/config.plist) (apply `MinKernel` and `MaxKernel` settings as well): <ul> <li> **Force FileVault on Broken Seal** (optional) <li> **"Disable Library Validation Enforcement"**<li>**"Disable _csr_check() in _vnode_check_signature"** <li> **Fix PCI bus enumeration (Ventura)** <li> **Fix PCI bus enumeration (Sonoma)** </ul> **NOTE**: VMM board-id kernel patches are no longer required since `RestrictEvents` kext can enable the VMM board-id via `sbvmm` NVRAM entry!| <ul> <li> **Force FileVault on Broken Seal** is only required when using FileVault <li> **"Disable _csr_check() in _vnode_check_signature"** is not required on my Laptop, but on some Desktops it's needed. Try for yourself. <li> **Fix PCI bus enumeration**: enable if internal PCIe devices are showing up as express cards in the menu bar: ![Screenshot](https://github.com/user-attachments/assets/d362d81c-01f7-491e-98c9-cd9372f30eb1) 
**`Kernel/Quirks`** | <ul><li> Enable **`AppleCpuPmCfgLock`**. Not required if you can disable CFG Lock in BIOS. <li> Disable **`AppleXcmpCfgLock`** (if enabled) <li> Disable **`AppleXcpmExtraMsrs`** (leave enabled when using an Ivy Bridge E CPU) | Again, we want to make sure, XCPM is not utilized for Ivy Bridge CPUs!
**`Misc/Security`**| <ul> <li>**SecureBootMbodel**: `Disabled` <li> **Vault**: `Optional`| Required when patching in graphics drivers for AMD and NVIDIA GPUs. Intel HD4000 drivers work with `SecureBootModel` set to `Default`. Try for yourself.
**`NVRAM/Add/...-4BCCA8B30102`** | **Add the following Keys:** <ul> <li> **Key**: `OCLP-Settings`<br>**Type**: String <br>**Value**: `-allow_amfi` <li> **Key**: `revblock` <br> **Type**: String <br> **Value**: `media`<li>  **Key**: `revpatch` <br> **Type:** String <br> **Value**: `sbvmm,f16c`| <ul> <li> Settings for OCLP and RestrictEvents. <li> `media`: Blocks `mediabranalysisd` service on Ventura+ (for Metal 1 GPUs) <li>`sbvmm,f16c` &rarr; Enables OTA updates and addresses graphics issues in macOS 13 (check RestrictEvents documentation for details)|
**`NVRAM/Delete/...-4BCCA8B30102`** (Array) | **Add the following Strings:** <ul> <li>  `OCLP-Settings` <li> `revblock` <li> `revpatch` | Deletes NVRAM for these parameters before writing them. Otherwise you would need to perform an NVRAM reset every time you change any of them in the corresponding `Add` section.  
**`NVRAM/Add/...-FE41995C9F82`** | **Change** **`csr-active-config`** to: <ul><li>**`03080000`** <li> When using an NVIDIA GPU, set it to: **`030A0000`**</ul> **Add the following**`boot-args`: <ul><li> ~**`amfi_get_out_of_my_way=0x1`** or **`amfi=0x80`** (both do the same)~ <li>`-amfipassbeta` <li> **`ipc_control_port_options=0`** </ul>**Optional boot-args for GPUs** (Select based on GPU Vendor): <ul><li> ~~**`-radvesa`**~~ <li> **`nv_disable=1`** <li> **`ngfxcompat=1`**<li>**`ngfxgl=1`**<li> **`nvda_drv_vrl=1`** <li> **`agdpmod=vit9696`** </ul> |<ul> <li>**`amfi=0x80`**: Disables Apple Mobile File Integrity validation. Required for applying Root Patches with OCLP ~~and booting macOS 12+~~. :bulb: No longer needed for booting thanks to AMFIPass.kext – only for installing Root Patches with OCLP. Disabling AMFI causes issues with [3rd party apps' access to Mics and Cameras](https://github.com/5T33Z0/OC-Little-Translated/blob/main/13_Peripherals/Fixing_Webcams.md).<li> **`ipc_control_port_options=0`**: Required for Intel HD 4000. Fixes issues with Firefox and electron-based apps like Discord. <li> **`-radvesa`** (AMD only): Disables hardware acceleration and puts the card in VESA mode. Only required if your screen turns off after installing macOS 12+. Once you've installed the GPU drivers with OCLP, **disable it** so graphics acceleration works! Reported as not working in macOS Sonoma. In this case, use `-amd_no_dgpu_accel` instead.<li> **`nv_disable=1`** (NVIDIA only): Disables hardware acceleration and puts the card in VESA mode. Only required if your screen turns off after installing macOS Ventura. Kepler Cards switch into VESA mode automatically without it. Once you've installed the GPU drivers with OCLP, **disable it** so graphics acceleration works! <li>**`ngfxcompat=1`** (NVIDIA only): Ignores compatibility check in `NVDAStartupWeb`. Not required for Kepler GPUs <li>**`ngfxgl=1`** (NVIDIA only): Disables Metal Spport so OpenGL is used for rendering instead. Not required for Kepler GPUs. <li> **`nvda_drv_vrl=1`** (NVIDIA only): Enables Web Drivers. Not required for Kepler GPUs. <li> **`agdpmod=vit9696`** &rarr; Disables board-id check. Useful if screen turns black after booting macOS which can happen after installing NVIDIA Webdrivers. <li> **`-wegnoigpu`** &rarr; Optional. Disables the iGPU in macOS. **ONLY** required when using an AMD GPU and an SMBIOS for a CPU without on-board graphics (i.e. `iMacPro1,1` or `MacPro7,1`) to let the GPU handle background rendering and other tasks. Requires Polaris or Vega cards to work properly (Navi is not supported by OCLP). Combine with `unfairgva=x` bitmask (x= 1 to 7) to [address DRM issues](https://github.com/5T33Z0/OC-Little-Translated/tree/main/H_Boot-args#unfairgva-overrides)
`UEFI/Drivers` and <br> `EFI/OC/Drivers`| <ul> <li> Add `ResetNvramEntry.efi` to `EFI/OC/Drivers` <li> And to your config:<br> ![resetnvram](https://github.com/5T33Z0/OC-Little-Translated/assets/76865553/8d955605-fb27-401f-abdd-2c616b233418) | Adds a boot menu entry to perform an NVRAM reset but without resetting the order of the boot drives. Requires a BIOS with UEFI support.

> [!CAUTION]
> 
> Don't add the NVRAM parameter `OCLP-Version` to your config – it's meant for real Macs only! It checks if your `config.plist` is up to par with the one provided by OCLP. If the version in your config is lower, a pop-up will appear asking you if you would like to update OpenCore:
>
> ![oclp-version](https://github.com/user-attachments/assets/3376afa3-da56-4311-9960-a9ec90e6010f)
>
> If you would press "OK" in this scenario, your `OC` folder would be replaced by the one created for the corresponding Mac model leaving your macOS installation in an unbootable state!

## Testing the changes
Once you've added the required kexts and made the necessary changes to your config.plist, save, reboot and perform an NVRAM Reset. If your system still boots fine after that, you can now prepare the system for installing macOS 13.

### Adjusting the SMBIOS
If your system reboots successfully, we need to edit the config one more time and adjust the SMBIOS depending on the macOS Version *currently* installed.

#### When Upgrading from macOS Big Sur 11.3+
When upgrading from macOS 11.3 or newer, we can use macOSes virtualization capabilities to trick it into thinking that it is running in a VM so spoofing a compatible SMBIOS is no longer a requirement.

Based on your system, use one of the following SMBIOSes for Ivy Bridge CPUs. Open your config.plist and change the `SystemProductName` in the `PlatformInfo/Generic` section. The processor numbers mentioned in brackets are CPUs used in actual Mac machines. 

- **For Desktops**: 
	- **`iMac13,1`** (i5-3330S, i5-3470S, i7-3770S)
	- **`iMac13,2`** (i5-3470S, i5-3470, i7-3770)
	- **`iMac13,3`** (i3-3225)
	- **`MacPro6,1`** (E5-1620v2, E5-1650v2, E5-1680v2, E5-2697v2)
- **For Laptops**:
	- **`MacBookPro10,1`** (i7-3615QM, i7-3635QM, i7-3720QM, i7-3740QM, i7-3820QM, i7-3840QM)
	- **`MacBookPro10,2`** (i5-3210M, i7-3520M, i5-3230M, i7-3540M)
	- **`MacBookPro9,1`** (i7-3615QM, i7-3720QM, i7-3820QM)
	- **`MacBookPro9,2`** (i5-3210M, i7-3520M)
	- **`MacBookAir5,1`** (i5-3317U, i7-3667U)
	- **`MacBookAir5,2`** (i5-3317U, i5-3427U or i7-3667U)
- **For NUCs**:
	- **`Macmini6,1`** (i5-3210M)
	- **`Macmini6,2`** (i7-3615QM, i7-3720QM)
- Generate new Serials using [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS)

> [!NOTE]
> 
> Once macOS 12 or newer is installed, you can disable the "Reroute kern.hv" and "IOGetVMMPresent" Kernel Patches. RestrictEvents will handle the VMM-Board-id spoof. **Only Exception**: Before running the "Install macOS" App, you have to re-enable the kernel patches again. Otherwise the installer will say the system is incompatible because of the unsupported SMBIOS it detects.
 	
#### When Upgrading from macOS Catalina or older
Since macOS Catalina and older lack the virtualization capabilities required to apply the VMM Board-ID spoof, switching to a supported SMBIOS temporarily is mandatory in order to be able to install macOS Ventura. Otherwise you will be greeted by the crossed-out circle instead of the Apple logo when trying to boot.

**Supported SMBIOSes**

- **Desktop**: 
	- **`iMac18,1`** or newer
	- **`MacPro7,1`** (High End Desktops)
- **Laptop**: 
	- **`MacBookPro14,1`** or 
	- **`MacBookAir8,1`**
- **NUC**: 
	- **`Macmini8,1`**

Generate new Serials using [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS)

> [!NOTE]
> 
> - Once macOS 12 or newer is installed, you can switch to an SMBIOS best suited for your Ivy Bridge CPU and reboot to enjoy all the benefits of a proper SMBIOS.
> - You may want to generate a new [**SSDT-PM**](https://github.com/5T33Z0/OC-Little-Translated/content/tree/main/01_Adding_missing_Devices_and_enabling_Features/CPU_Power_Management/CPU_Power_Management_(Legacy)) in Post-Install to optimize CPU Power Management.
> - You can also disable the "Reroute kern.hv" and "IOGetVMMPresent" Kernel Patches. RestrictEvents will handle the VMM-Board-id spoof from now on. **Only Exception**: Before running the "Install macOS" App, you have to re-enable the kernel patches again. Otherwise the installer will say the system is incompatible because of the unsupported SMBIOS it detects. 

---

[← **Previous: Introduction**](README.md) | [**Next: macOS Installation →**](Ivy_Install.md)