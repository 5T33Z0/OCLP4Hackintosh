![Last Update](https://img.shields.io/badge/Last_Update_(yy/mm/dd):-26.11.11-blueviolet.svg)

# OCLP-4-Hackintosh: Installing newer versions of macOS on legacy Wintel systems

## About

**OCLP4Hackintosh** is a streamlined guide for using OpenCore Legacy Patcher (OCLP) to apply macOS patches on Intel-based Hackintosh systems, enabling support for modern macOS versions via OpenCore. This content was originally part of my [OC-Little Translated](https://github.com/5T33Z0/OC-Little-Translated) repository, but due to its significant growth, I decided to move it to a dedicated repo for easier maintenance.

As you may know, Dortania developed the [**OpenCore Legacy Patcher**](https://github.com/dortania/OpenCore-Legacy-Patcher) (OCLP) to install and run macOS 12 and newer on end-of-life Macs with Intel Core CPUs from the 1st to 6th generation (Kaby Lake to Comet Lake CPUs are still supported by macOS 15). It achieves this by installing the OpenCore boot loader on the target system to inject settings and [additional kexts](https://github.com/dortania/OpenCore-Legacy-Patcher/tree/main/payloads/Kexts) required for:

- Booting newer versions of macOS on unsupported board IDs, utilizing the native SMBIOS best suited for the CPU ([more details](https://github.com/5T33Z0/OC-Little-Translated/tree/main/Content/09_Board-ID_VMM-Spoof)).
- Re-enabling legacy SMC CPU Power Management in macOS 13+ (1st to 3rd Gen Intel Core CPUs).
- Fixing issues with System Updates caused by disabling `SecureBootModel`, System Integrity Protection (`SIP`), and Apple Mobile File Integrity (`AMFI`).

Additionally, OCLP applies [*on-disk patches*](https://dortania.github.io/OpenCore-Legacy-Patcher/PATCHEXPLAIN.html#on-disk-patches) (“root patches”) during Post-Install to restore essential functionality such as hardware graphics acceleration (iGPU/GPU), Wi-Fi, and Bluetooth. These patches extend the lifecycle of machines that Apple has artificially blocked from upgrading through board-ID–based software lockouts.

What you may not know is that some of OCLP's configuration settings, kexts, and root patches can also be utilized on Wintel systems (aka Hackintoshes). However, the required modifications to `config.plist` and the selection of kexts to inject (as some are only needed for Apple systems) are not officially documented or supported by Dortania, and you won't find assistance for this on Discord. That's why I've created detailed configuration guides to prepare legacy systems to install and run macOS Ventura through Tahoe.

**Patches relevant to us are:**

- iGPU drivers (to [reinstate graphics acceleration and Metal Graphics API support](https://khronokernel.github.io/macos/2022/11/01/LEGACY-METAL-PART-1.html)).
- GPU drivers for legacy (non-Metal) AMD and NVIDIA Kepler cards.
- Frameworks for re-enabling USB, Wi-Fi, and Bluetooth.

## Latest updates

- [**OCLP Development status**](docs/OCLP-related/OCLP_status.md)
- 💥 [**Alternative Repositories**](docs/Forks.md) – A collection of actively maintained forks and alternative repos for kexts and tools that extend compatibility, fix long-standing issues, or add features missing from the original kexts, drivers, and tools.
- [**macOS Tahoe Updates**](docs/macOS-related/Tahoe_Notes.md) (Contains the latest findings, fixes, and information for Apple's latest OS).
- [**macOS Sequoia OCLP Notes**](docs/macOS-related/Sequoia_Notes.md)
- [**macOS Sonoma OCLP Notes**](docs/macOS-related/Sonoma_Notes.md)

## OpenCore Configuration Guides

Listed below are configuration guides to update your OpenCore EFI and `config.plist` with the required settings and kexts to run macOS 13 and newer:

- [**Installing macOS 26 on Kaby Lake or newer**](docs/Guides/Configuration_Guides/Kaby_Lake+/Kaby_Lake+.md)
- [**Installing macOS 13+ on Skylake systems**](docs/Guides/Configuration_Guides/Skylake/README.md)
- [**Installing macOS 13+ on Haswell/Broadwell systems**](docs/Guides/Configuration_Guides/Haswell_Broadwell/Haswell-Broadwell.md)
- [**Installing macOS 13+ on Ivy Bridge systems**](docs/Guides/Configuration_Guides/Ivy_Bridge/Ivy_Bridge.md)
- [**Installing macOS 13+ on Sandy Bridge systems**](docs/Guides/Configuration_Guides/Sandy_Bridge/Sandy_Bridge.md)
- [**Installing macOS 13+ on 1st Gen Intel Core systems**](docs/Guides/Configuration_Guides/Nehalem/README.md)
- [**General CPU and SMBIOS Guide**](docs/OCLP-related/CPU_to_SMBIOS.md)

> [!IMPORTANT]
>
> Updating from macOS 14.3.x to 14.4.x and newer might crash the installer early. This is related to `SecureBootModel`, so it should be set to `Disabled` during installation (→ see the [**Workarounds**](https://github.com/5T33Z0/OC-Little-Translated/blob/main/W_Workarounds/macOS14.4.md) section for details).

## (Re-)Enabling Features

### macOS Tahoe

- [**Fixing Audio in macOS Tahoe**](docs/Guides/Enable_Features/Audio/Audio_Tahoe.md)
- [**Fixing `USBMap.kext` for macOS Tahoe**](docs/Guides/Enable_Features/USB/USB_Tahoe.md)
- [**Re-Enabling Wi-Fi and Bluetooth**](docs/Guides/Enable_Features/WiFi/WiFi_BT_Sonoma+.md) (Broadcom/Atheros/Intel).
- [**Alternatives to OpenCore Legacy Patcher**](docs/OCLP-related/OCLP_Alternatives.md)

### macOS Sequoia

- [**Re-Enabling Wi-Fi and Bluetooth**](docs/Guides/Enable_Features/WiFi/WiFi_BT_Sonoma+.md) (Broadcom/Atheros/Intel).
- [**How to disable Gatekeeper in macOS Sequoia**](docs/macOS-related/Disable_Gatekeeper.md)

### macOS Sonoma

- [**Re-Enabling Wi-Fi and Bluetooth**](docs/Guides/Enable_Features/WiFi/WiFi_BT_Sonoma+.md) (Broadcom/Atheros/Intel).

## Misc

- [**How to enable auto-root-patching during macOS installation**](docs/OCLP-related/Auto-Patching.md)
- [**Force-enabling root patches in OCLP**](docs/OCLP-related/Force-enable_Root-Patches.md)

## OCLP-Mod

- [**How to build the multilingual version of OCLP-Mod**](docs/OCLP-related/OCLP-mod_Multilingual.md)

## Troubleshooting

- [**Dos and Don'ts of running macOS beta versions**](docs/macOS-related/Beta_dos_donts.md)
- [**Recovering from failed root patching attempts**](docs/OCLP-related/Failed_Root_Patches.md)
- [**OCLP and the macOS compatibility gap**](docs/macOS-related/Bridging_the_gap.md)
- [**Triggering macOS Updates via Terminal**](docs/macOS-related/macOS_Update_Terminal.md)
- [**Addressing sleep issues in macOS Sequoia**](https://www.insanelymac.com/forum/topic/360040-macos-15-sequoia-does-not-enter-sleep-mode-properly/#comment-2826474) (Thread on InsanelyMac).
- [**Fixing Preboot volumes modified by macOS High Sierra**](docs/macOS-related/Fix_Preboot_High_Sierra.md)

## Fetching macOS Installers

There are several options to fetch and download macOS installers directly from Apple. Here are some of them:

1. **OpenCore Legacy Patcher**. It can download macOS 11+ and create a USB installer as well.
2. [**Download Full Installer**](https://github.com/perez987/DownloadFullInstaller): A simple macOS app to download macOS Big Sur through Tahoe for Intel-based systems.
3. [**MIST**](https://github.com/ninxsoft/Mist): A GUI-based app to download macOS installers and Apple Silicon firmware.
4. **Terminal**. Open Terminal and enter the following commands:

   ```bash
   softwareupdate --fetch-full-installer --list-full-installers
   ```

   (To fetch the list of available installers.)

   ```bash
   softwareupdate --fetch-full-installer --full-installer-version xx.xx
   ```

   (Replace `xx.xx` with the version you want to download.)

For more options, check the [**Utilities**](https://github.com/5T33Z0/OC-Little-Translated/tree/main/C_Utilities_and_Resources#getting-macos) section.

## Miscellaneous

- [**OCLP FAQ**](https://dortania.github.io/OpenCore-Legacy-Patcher/FAQ.html#application-requirements)
- [**OCLP Changelog**](https://github.com/dortania/OpenCore-Legacy-Patcher/blob/main/CHANGELOG.md)
- [**OCLP Troubleshooting**](https://github.com/dortania/OpenCore-Legacy-Patcher/blob/main/docs/TROUBLESHOOTING.md)
- [**Installing Windows from within macOS without Boot Camp**](https://github.com/5T33Z0/OC-Little-Translated/blob/main/I_Windows/Install_Windows_NoBootcamp.md)
- [**Collection of Non-Metal Apple apps**](https://archive.org/details/apple-apps-for-non-metal-macs) (Archive.org).
- [**macOS Release Notes**](https://developer.apple.com/documentation/macos-release-notes)
- [**Evolution of macOS**](docs/macOS-related/macOS_Evolution.md)

## Resources

- **OpenCore Bootloader**: https://github.com/acidanthera/OpenCorePkg/releases
- **OpenCore Legacy Patcher**: https://github.com/dortania/OpenCore-Legacy-Patcher/releases
- **Kernel Debug Kit Support Package**: https://github.com/dortania/KdkSupportPkg/releases
- **Metallib Support Package**: https://github.com/dortania/MetallibSupportPkg/releases

## Contribute

Although I've created these guides with a lot of attention to detail, there's always room for improvement. As far as verifying the guides are concerned, I only have the following systems for testing: an iMac11,3 (Lynnfield), an iMac12,2 (Sandy Bridge), some notebooks (Ivy Bridge, Skylake, Whiskey Lake), and some Small Form Factor machines (Kaby Lake and Coffee Lake). So if you have any suggestions or updated instructions to improve the guides or workflows, feel free to create an issue and let me know!
