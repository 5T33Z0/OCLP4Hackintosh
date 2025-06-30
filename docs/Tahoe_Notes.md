# macOS Tahoe Notes

On June 9th 2025, macOS unveiled macOS 26, aka macOS Tahoe (Kernel version 25.0). Seemingly, Apple decided to drop the version-based naming scheme (macOS 16 would have been the next logical version) to a year-based scheme (26 as in 2026)… I am already not a fan. Apple also announced that "macOS Tahoe will be the final release for Intel Macs".

## (Officially) Supported Mac Models 

![macOS_talahon](https://github.com/user-attachments/assets/2e3c53c7-4b33-4968-8505-e15247619004)

**Source**: https://www.apple.com/os/macos/

It looks like Dortania has a lot of work to do to support all of these newly dropped Intel Macs via OCLP…

## Usable SMBIOSes

- **Noetbooks**: `MacBookPro16,1` and `MacBookPro16,4`
- **Desktops**: `iMac20,1` and `iMac20,2`
- The rest: dropped

Since the board-id check skip and `RestrictEvents.kext` still work, macOS Tahoe can be installed with a native SMBIOS for the used CPU family and system updates can be obtained as well.

## New features

- Nothing really notable besides an overhaul of Spotlight
- Horrible overhaul of the on-screen display

## Current OCLP Status

- Waiting for official statements by Dortania et. al.
- There's now a OCLP Mod available which can run in macOS Tahoe.

## Known Issues

- [x] **Audio**:
	- [x] **On-board audio**: `AppleHDA` has been removed from Tahoe beta 2 (not present at `System/Libray/Extensions`). Without it, AppleALC is useless and on-board audio CODECs won't work. Injectig AppleHDA via OpenCore is not an option (no boot). Audio over GPU (HDMI/DP) still works. But there's a [fix](/Enable_Features/Audio_Tahoe.md).
- [ ] **Video**:
	- [ ] **Metal 4 support**: Apple Silicon only.
	- [x] **AMD Polaris GPUs**: can be enabled in beta 2 by disabling WhateverGreen during install.
- **Connectivity**
	- [x] **Ethernet**: 
		- Intel NICs using **IntelMause** or **IntelMausiEthernet** (and derivates therof) won't work due to incomplete **AppleVTD** support. **Solution**: use Miezes updated version of [**IntelMausiEthernet**](https://github.com/Mieze/IntelMausiEthernet/releases) with AppleVTD support. If Ethernet still doesn't work afterwards, enable **`DisableIoMapper`** Quirk!
		- NICs requiring `AppleVTD` to be present *might* work based on mainboard and chipset ([Example](https://github.com/SongXiaoXi/AppleIGC/issues/23#issuecomment-3002214327))
	- [ ] **Bluetooth**: Intel Bluetooth doesn't work on my systems currently. Maybe BluetoolFixup needs an update.
- [x] **DRM issues**: Apple Music not playing back any music (online or local). **Fix**: add boot-arg `unfairvga=7` Thanks to [patriczeq](https://github.com/5T33Z0/Thinkpad-T490-Hackintosh-OpenCore/issues/57#issuecomment-2977474242) for the tip
- [x] **USB**: USB issues due to changed parameters ([**Fix**](/Enable_Features/USB_Tahoe.md))
- [x] **File Vault**: There seems to be a bug in the APFS driver of the Tahoe beta release that causes issues with File Vault 2. If the previous driver from macOS 15 is injected via OpenCore, File Vault 2 should works as expected ([**Source**](https://github.com/acidanthera/bugtracker/issues/2499))

### AppleVTD support

A word from [Mieze](https://www.hackintosh-forum.de/forum/thread/60350-wwdc-2025-macos-26-hackintosh/?postID=802677#post802677), developer of IntelMausiEthernet:

> Currently, the following Ethernet drivers support **AppleVTD** in the current version (provided the motherboard cooperates), but can also operate without it:
>
> - LucyRTL8125Ethernet
> - RealtekRTL8111
> - AtherosE2200Ethernet
> - IntelLucy
> - [IntelMausiEthernet](https://github.com/Mieze/IntelMausiEthernet/releases) (v2.5.5d0, recently updated to support AppleVTD!)
>
> The following drivers currently do _not_ work with **AppleVTD** because approximately 10 lines of code are missing: 
>
> - IntelMausi
>
> In contrast, all drivers provided by Apple _strictly require AppleVTD_ and cannot operate without it, as they no longer run in kernel-space but in user-space.
  
## Observations

- Board-ID Skip and RestrictEvents kext still work
- On my Ivy Bridge Notebook, installation worked without effort. But the system crashes after completing the first stage of the install assistant.
- Couldn't get it to work on my Z490 system with a Comet Lake CPU and a Polaris GPU (RX580). **Workaround**: disable GPU (`-wegnoegpu`) and use on-board graphics

## Recommendations

- You can install macOS Tahoe if your system has a Kaby Lake or newer CPU and does not require root patches by OCLP (board-id check skip is required when using an unsupported SMBIOS).
- Don't install macOS Tahoe if your GPU and/or iGPU is incompatible and requires root patches. So no 11. Gen CPUs or newer and no Legacy NVIDIA and AMD GPUs.

## Outlook

- khronokernel, one of the cornerstone developers of OpenCore and OCLP stated in his [OpenCore Legacy Patcher retrospective](https://khronokernel.com/macos/2025/06/20/OCLP-RETROSPECTIVE.html) blog post that he has quit working on OCLP.

## Further Resources
- [macOS 26 Beta Release Notes](https://developer.apple.com/documentation/macos-release-notes/macos-26-release-notes)
- [macOS Tahoe 26 on Unsupported Macs](https://forums.macrumors.com/threads/macos-tahoe-26-on-unsupported-macs-discussion.2458481/) – Thread on macrumors.com. A good source for current developments regarding OCLP
- [Supported/unsupported GPUs](https://osxlatitude.com/forums/topic/8238-supportedunsupported-gpus-graphics-cards/) - Thread on osxlatitude
