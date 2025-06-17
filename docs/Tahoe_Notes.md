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

- Nothing really noteable besided an overhaul of Spotlight

## OpenCore Patcher Support

- Unknown yet. Waiting for official statements by Dortania et. al.

## Status

- Don't install macOS Tahoe yet! Wait for updated kexts and OpenCore builds.

## Technical challenges

- [ ] Metal 4 support
- [x] DRM issues. Apple Music not playing back any music (online or local). **Fix**: add boot-arg `unfairvga=7` Thanks to [patriczeq](https://github.com/5T33Z0/Thinkpad-T490-Hackintosh-OpenCore/issues/57#issuecomment-2977474242) for the tip
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
  > - IntelMausi4
  
  > In contrast, all drivers provided by Apple _strictly require AppleVTD_ and cannot operate without it, as they no longer run in kernel-space but in user-space.
  
- [x] USB issues due to changed parameters ([**Fix available**](/Enable_Features/USB_Tahoe.md))
- [ ] GPUs: AMD Polaris GPUs: I couldn't get my AMD Radeon RX580 to work in macOS Tahoe although the required kext is still present
- [ ] Bluetooth: Intel Bluetooth doesn't work on my systems currently. Maybe BluetoolFixup needs an update.

## Observations

- Board-ID Skip and RestrictEvents still seem to work
- On my Ivy Bridge Notebook installation worked without effort. But the system crashes after completing the first stage of the install assistant.
- Couldn't get it to work on my Z490 system with a Comet Lake CPU and a Polaris GPU. I've read that disabling it and using the iGPU works

## Further Resources

- [macOS Tahoe 26 on Unsupported Macs](https://forums.macrumors.com/threads/macos-tahoe-26-on-unsupported-macs-discussion.2458481/) – Thread on macrumors.com. A good source for current developments regarding OCLP
- [Supported/unsupported GPUs](https://osxlatitude.com/forums/topic/8238-supportedunsupported-gpus-graphics-cards/) - Thread on osxlatitude
