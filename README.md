# OCLP-4-Hackintosh: Installing newer versions of macOS on legacy Wintel systems

![Last Update](https://img.shields.io/badge/Last_Update_\(yy/mm/dd\):-26.02.16-blueviolet.svg)

---

**TABLE oF CONTENTS**

- [About](#about)
- [Latest updates](#latest-updates)
- [Configuration Guides](#configuration-guides)
- [(Re-)Enabling Features](#re-enabling-features)
- [Troubleshooting](#troubleshooting)
- [Fetching macOS Installers](#fetching-macos-installers)
- [Miscellaneous](#miscellaneous)
- [Resources](#resources)
- [Contribute](#contribute)

---

## About
**OCLP4Hackintosh** is a streamlined guide for using OpenCore Legacy Patcher (OCLP) to apply macOS patches on Intel-based Hackintosh systems, enabling support for modern macOS versions via OpenCore. This content was originally part of my [OC-Little Translated](https://github.com/5T33Z0/OC-Little-Translated) repository, but due to its significant growth, I decided to move it to a dedicated repo for easier maintenance.

As you may know, Dortania developed the [**OpenCore Legacy Patcher**](https://github.com/dortania/OpenCore-Legacy-Patcher) (OCLP) to install and run macOS 12 and newer on end-of-life Macs with Intel Core CPUs from the 1st to 6th generation (Kaby Lake to Comet Lake CPUs are still supported by macOS 15). It achieves this by installing the OpenCore boot loader on the target system to inject settings and [additional kexts](https://github.com/dortania/OpenCore-Legacy-Patcher/tree/main/payloads/Kexts) required for:

- Booting newer versions of macOS on unsupported board-ids, utilizing the native SMBIOS best suited for the used CPU ([more details](https://github.com/5T33Z0/OC-Little-Translated/tree/main/09_Board-ID_VMM-Spoof)).
- Re-enabling legacy SMC CPU Power Management in macOS 13+ (1st to 3rd Gen Intel Core CPUs)
- Fixing issues with System Updates caused by disabling `SecureBootModel`, System Integrity Protection (`SIP`) and Apple Mobile File Integrity (`AMFI`)

Aditionally, OCLP applies [*on-disk patches*](https://dortania.github.io/OpenCore-Legacy-Patcher/PATCHEXPLAIN.html#on-disk-patches) (“root patches”) during Post-Install to restore essential functionality such as hardware graphics acceleration (iGPU/GPU), Wi-Fi, and Bluetooth. These patches extend the lifecycle of machines that Apple has artificially blocked from upgrading through board-ID–based software lockouts.

What you may not know is: some of OCLPs config settings, kexts and root-patches can be utilized on Wintel systems (aka Hackintoshes) as well. However, the required modifications to the `config.plist` and the selection of kexts to inject (as some are only needed for Apple systems) are not officially documented nor supported by Dortania, and you won’t find assistance for this on Discord. That’s why I’ve created detailed configuration guides to prepare legacy systems to install and run macOS Ventura to Tahoe.

**Patches relevant to us are**:

- iGPU drivers (to [reinstate graphics acceleration and Metal Graphics API support](https://khronokernel.github.io/macos/2022/11/01/LEGACY-METAL-PART-1.html)) 
- GPU drivers for legacy (non-metal) AMD and NVIDIA Kepler Cards 
- Frameworks for re-enabling USB, Wi-Fi and Bluetooth
 
## Latest updates
- [**OCLP update status**](/docs/OCLP_status.md)
- [**macOS Tahoe Updates**](/docs/Tahoe_Notes.md) (Contains the latest findings, fixes and stuff for Apple's latest OS)
- [**macOS Sequoia OCLP Notes**](/docs/Sequoia_Notes.md)
- [**macOS Sonoma OCLP Notes**](/docs/Sonoma_Notes.md)

## Configuration Guides
Listed below are configuration guides to update your OpenCore EFI and `config.plist` with the required settings and kext to run macOS 13 and newer:

- [**Installing macOS 26 on Kaby Lake or newer**](/Guides/Kaby_Lake+.md)
- [**Installing macOS 13+ on Skylake systems**](/Guides/Skylake.md)
- [**Installing macOS 13+ on Haswell/Broadwell systems**](/Guides/Haswell-Broadwell.md)
- [**Installing macOS 13+ on Ivy Bridge systems**](/Guides/Ivy_Bridge.md)
- [**Installing macOS 13+ on Sandy Bridge systems**](/Guides/Sandy_Bridge.md)
- [**Installing macOS 13+ on 1st Gen Intel Core systems**](/Guides/Nehalem-Westmere-Lynnfield.md)
- [**General CPU and SMBIOS Guide**](/Guides/CPU_to_SMBIOS.md)

> [!IMPORTANT]
>
> Updating from from macOS 14.3.x to 14.4.x and newer might crash the installer early. This is related to `SecureBootModel`, so it should be set to `Disabled` during installation (&rarr; see [**Workarounds**](https://github.com/5T33Z0/OC-Little-Translated/blob/main/W_Workarounds/macOS14.4.md) section for details).

## (Re-)Enabling Features

### macOS Tahoe
- [**Fixing Audio in macOS Tahoe**](/Enable_Features/Audio_Tahoe.md)
- [**Fixing `USBMap.kext` for macOS Tahoe**](/Enable_Features/USB_Tahoe.md)
- [**Enabling `AirportItlwm.kext` in macOS Sequoia and Tahoe**](/Enable_Features/AirportItllwm_Sequoia.md)

### macOS Sequoia
- [**Enabling `AirportItlwm.kext` in macOS Sequoia and Tahoe**](/Enable_Features/AirportItllwm_Sequoia.md)
- [**How to disable Gatekeeper in macOS Sequoia**](/Guides/Disable_Gatekeeper.md)

### macOS Sonoma
- [**Fixing WiFi and Bluetooth in macOS Sonoma+**](/Enable_Features/WiFi_Sonoma.md)

### Misc
- [**How to enable auto-root-patching during macOS installation**](/Guides/Auto-Patching.md)
- [**Force-enabling root-patches Patches in OCLP**](/Enable_Features/Force-enable_Root-Patches.md)

## OCLP-Mod
- [How to build the multilingual version of OCLP-Mod](https://github.com/5T33Z0/OCLP4Hackintosh/blob/main/Guides/OCLP-mod_Multilingual.md)

> [!TIP]
> 
> There's a newer fork of OCLP-Mod called [**OCLP-Plus**](https://github.com/YBronst/OCLP-Plus) which has an english GUI. It's highly recommended to use it insead of OCLP-Mod!

## Troubleshooting
- [**Dos and Don'ts of running macOS beta versions**](/docs/Beta_dos_donts.md)
- [**Recovering from failed root patching attempts**](/Guides/Reverting_Root_Patches.md)
- [**OCLP and the macOS compatibility gap**](/docs/Bridging_the_gap.md)
- [**Triggering macOS Updates via Terminal**](/docs/macOS_Update_Terminal.md)
- [**Addressing sleep issues in macOS Sequoia**](https://www.insanelymac.com/forum/topic/360040-macos-15-sequoia-does-not-enter-sleep-mode-properly/#comment-2826474) (Thread on insanelymac)
- [**Fixing Preboot volumes modified by macOS High Sierra**](/Guides/Fix_Preboot_High_Sierra.md)

## Fetching macOS Installers

There are several options to fetch and download macOS installers directly from Apple. Here are some of them:

1. **OpenCore Legacy Patcher**. It can download macOS 11+ and create a USB Installer as well.
2. [**Download Full Installer**](https://github.com/perez987/DownloadFullInstaller): Simple macOS App to download macOS Big Sur to Tahoe for Intel-based system
3. [**MIST**](https://github.com/ninxsoft/Mist): GUI-based app to download macOS Installers and Apple Silicon Firmwares
4. **Terminal**. Open Terminal and enter the following commands:<br>
	`softwareupdate  --fetch-full-installer --list-full-installers` (to fetch the list of Installers)<br>
	`softwareupdate  --fetch-full-installer --list-full-installer-version xx.xx` (replace xx.xx by the version you want to download)

For more options, check the [**Utilities**](https://github.com/5T33Z0/OC-Little-Translated/tree/main/C_Utilities_and_Resources#getting-macos) section
 
## Miscellaneous
- [**OCLP FAQ**](https://dortania.github.io/OpenCore-Legacy-Patcher/FAQ.html#application-requirements)
- [**OCLP Changelog**](https://github.com/dortania/OpenCore-Legacy-Patcher/blob/main/CHANGELOG.md)
- [**OCLP Troubleshooting**](https://github.com/dortania/OpenCore-Legacy-Patcher/blob/main/docs/TROUBLESHOOTING.md)
- [**Installing Windows from within macOS without Bootcamp**](https://github.com/5T33Z0/OC-Little-Translated/blob/main/I_Windows/Install_Windows_NoBootcamp.md)
- [**Collection of Non-Metal Apple apps**](https://archive.org/details/apple-apps-for-non-metal-macs) (Archive.org)
- [**macOS Release Notes**](https://developer.apple.com/documentation/macos-release-notes)
- [**Evolution of macOS**](/docs/macOS_Evolution.md)

## Resources

- **OpenCore Bootloader**: https://github.com/acidanthera/OpenCorePkg/releases
- **OpenCore Legacy Patcher**: https://github.com/dortania/OpenCore-Legacy-Patcher/releases
- **MetallibSupportPkg**: https://github.com/dortania/MetallibSupportPkg/releases

## Contribute
Although I've created these guides with a lot of attention to detail, there's always room for improvement. As far as verifying the guides are concerned, I only have the following systems for testing: an iMac11,3 (Lynnfield), an iMac12,2 (Sandy Bridge), Some notebooks (Ivy Bridge, Skylake, Whiskey Lake) and some Small Form Factor machines (Kaby Lake and Coffee Lake). So if you have any suggestions or updated instructions to improve the guides or workflows, feel free to create an issue and let me know!
