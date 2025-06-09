# OCLP4Hackintosh – Installing newer versions of macOS on legacy Wintel systems

## About
**OCLP4Hackintosh** is a concise guide for using OpenCore Legacy Patcher (OCLP) to apply macOS patches on Intel-based Hackintosh systems, enabling modern macOS support with OpenCore. Originally, the content in this repo was a chapter in my [OC-Little Translated](https://github.com/5T33Z0/OC-Little-Translated) repo but I decided to outsource it to a dedicated repo to better maintain it. 

As you may know, Dortania developed the [**OpenCore Legacy Patcher**](https://github.com/dortania/OpenCore-Legacy-Patcher) (OCLP) to install and run macOS 12 and newer on end-of-life Macs with Intel Core CPUs from the 1st to 6th generation (Kaby Lake to Comet Lake CPUs are still supported by macOS 15). It achieves this by installing the OpenCore boot loader on the target system to inject settings and [additional kexts](https://github.com/dortania/OpenCore-Legacy-Patcher/tree/main/payloads/Kexts) required for:

- Booting newer versions of macOS on unsupported board-ids, utilizing the native SMBIOS best suited for the used CPU ([more details](/09_Board-ID_VMM-Spoof/README.md)).
- Re-enabling legacy SMC CPU Power Management in macOS 13+ (1st to 3rd Gen Intel Core CPUs)
- Fixing issues with System Updates caused by disabling `SecureBootModel`, System Integrity Protection (`SIP`) and Apple Mobile File Integrity (`AMFI`)

Additionally, OCLP applies [on-disk patches](https://dortania.github.io/OpenCore-Legacy-Patcher/PATCHEXPLAIN.html#on-disk-patches) ("root-patches") in Post-Install to re-enable crucial features like hardware graphics acceleration (iGPU/GPU) as well as WiFi/BT to extend the lifecycle of these expansive machines.

What you may not know is: some of the settings, kexts and root-patches can  be utilized on Wintel systems as well. But the necessary adjustments of the `config.plist` and which of the kexts to inject (some of them are only required on Apple systems) is not officially documented nor supported by Dortania, nor will you receive any help for this on Discord. That's why I created in-depth configuration guides so your old Hackintosh can run macOS 12 and newer.

**Patches relevant to us are**:

- iGPU drivers (to [reinstate graphics acceleration and Metal Graphics API support](https://khronokernel.github.io/macos/2022/11/01/LEGACY-METAL-PART-1.html)) 
- GPU drivers for legacy (non-metal) AMD and NVIDIA Kepler Cards 
- Frameworks for re-enabling previously supported Wi-Fi/Bluetooth cards
 
## Latest updates
- [**OCLP update status**](/docs/OCLP_status.md)
- [**macOS Sequoia OCLP Notes**](/docs/Sequoia_Notes.md)
- [**macOS Sonoma OCLP Notes**](/docs/Sonoma_Notes.md)

## Configuration Guides
Listed below are configuration guides to update your OpenCore EFI and `config.plist` with the required settings and kext to run macOS 13 and newer:

- [**Installing macOS 13+ on 1st Gen Intel Core systems**](/Guides/Nehalem-Westmere-Lynnfield.md)
- [**Installing macOS 13+ on Sandy Bridge systems**](/Guides/Sandy_Bridge.md)
- [**Installing macOS 13+ on Ivy Bridge systems**](/Guides/Ivy_Bridge.md)
- [**Installing macOS 13+ on Haswell/Broadwell systems**](/Guides/Haswell-Broadwell.md)
- [**Installing macOS 13+ on Skylake systems**](/Guides/Skylake.md)
- [**General CPU and SMBIOS Guide**](/Guides/CPU_to_SMBIOS.md)

> [!IMPORTANT]
>
> Updating from from macOS 14.3.x to 14.4.x and newer might crash the installer early. This is related to `SecureBootModel`, so it should be set to `Disabled` during installation (&rarr; see [**Workarounds**](https://github.com/5T33Z0/OC-Little-Translated/blob/main/W_Workarounds/macOS14.4.md) section for details).

## (Re-)Enabling Features
- [**Fixing WiFi and Bluetooth in macOS Sonoma+**](/Enable_Features/WiFi_Sonoma.md)
- [**Enabling `AirportItlwm.kext` in macOS Sequoia and fixing iServices**](/Enable_Features/AirportItllwm_Sequoia.md)
- [**How to disable Gatekeeper in macOS Sequoia**](/Guides/Disable_Gatekeeper.md)
- [**How to enable auto-root-patching during macOS installation**](/Guides/Auto-Patching.md)
- [**Force-enabling root-patches Patches in OCLP**](/Enable_Features/Force-enable_Root-Patches.md)

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
2. [**MIST**](https://github.com/ninxsoft/Mist): GUI-based app to download macOS Installers and Apple Silicon Firmwares
3. **Terminal**. Open Terminal and enter the following commands:<br>
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
