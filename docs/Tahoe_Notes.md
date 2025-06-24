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

Since the board-id check skip still works, macOS Tahoe can be installed with a native SMBIOS for the used CPU family.

## New features

- Nothing really notable besides an overhaul of Spotlight
- Horrible overhaul of the on-screen display  

## Current OCLP Status

- Unknown yet. Waiting for official statements by Dortania et. al.

## Technical challenges/issues

- [ ] Metal 4 support
- [ ] **Audio issues**: On-board audio stopped working on macOS Tahoe beta 2. According to [this post](https://forums.macrumors.com/threads/macos-tahoe-26-on-unsupported-macs-discussion.2458481/page-9?post=33975897#post-33975897) on MacRumors Forums, AppleHDA has been removed from macOS Tahoe beta 2. Tha's bad news.
- [ ] **GPUs**: AMD Polaris GPUs: I couldn't get my AMD Radeon RX580 to work in macOS Tahoe although the required kext is still present
- [ ] **Bluetooth**: Intel Bluetooth doesn't work on my systems currently. Maybe BluetoolFixup needs an update.
- [x] **DRM issues**: Apple Music not playing back any music (online or local). **Fix**: add boot-arg `unfairvga=7` Thanks to [patriczeq](https://github.com/5T33Z0/Thinkpad-T490-Hackintosh-OpenCore/issues/57#issuecomment-2977474242) for the tip
- [x] **Ethernet**: Intel NICs requiring **IntelMausiEthernet** (and derivates therof) won't work due to incomplete **AppleVTD** support. **Workaround**: enable **`DisableIoMapper`** Quirk!

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
  
  > In contrast, all drivers provided by Apple _strictly require AppleVTD_ and cannot operate without it, as they no longer run in kernel-space but in user-space.
  
- [x] **USB**: USB issues due to changed parameters ([**Fix**](/Enable_Features/USB_Tahoe.md))
- [x] **File Vault**: According to OC dev M. Haeuser, there seems to be a bug in the APFS driver of the Tahoe beta release that causes issues with File Vault 2. If the previous driver from macOS 15 is injected via OpenCore (loacted at `usr/standalone/i386/apfs_aligned.efi`), File Vault 2 works as expected ([**Source**](https://www.hackintosh-forum.de/forum/thread/60350-wwdc-2025-macos-26-hackintosh/?postID=802857#post802857))

## Observations

- Board-ID Skip and RestrictEvents kext still work
- On my Ivy Bridge Notebook, installation worked without effort. But the system crashes after completing the first stage of the install assistant.
- Couldn't get it to work on my Z490 system with a Comet Lake CPU and a Polaris GPU (RX580). **Workaround**: disable GPU (`-wegnoegpu`) and use on-board graphics

## Recommendations

- You can install macOS Tahoe if your system has a Kabylake or newer CPU and does not require root patches by OCLP (board-id check skip is required when using an unsupported SMBIOS).
- Don't install macOS Tahoe if your GPU and/or iGPU is incompatible and requires root patches. So no 11. Gen CPUs or newer and no Legacy GPUs (including AMD Radeon RX5xx).

## Outlook

- khronokernel, one of the cornerstone developers of OpenCore and OCLP stated in his [OpenCore Legacy Patcher retrospective](https://khronokernel.com/macos/2025/06/20/OCLP-RETROSPECTIVE.html) blog post that he has quit working on OCLP.

## Further Resources
- [macOS 26 Beta Release Notes](https://developer.apple.com/documentation/macos-release-notes/macos-26-release-notes)
- [macOS Tahoe 26 on Unsupported Macs](https://forums.macrumors.com/threads/macos-tahoe-26-on-unsupported-macs-discussion.2458481/) – Thread on macrumors.com. A good source for current developments regarding OCLP
- [Supported/unsupported GPUs](https://osxlatitude.com/forums/topic/8238-supportedunsupported-gpus-graphics-cards/) - Thread on osxlatitude
