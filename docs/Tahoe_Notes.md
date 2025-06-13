# macOS Tahoe Notes

On June 9th 2025, macOS unveiled macOS 26, aka macOS Tahoe (Kernel version 25.0). Seemingly, Apple decided to drop the version-based naming scheme (macOS 16 would have been the next logical version) to a year-based scheme (26 as in 2026)… I am already not a fan. Apple also announced that "macOS Tahoe will be the final release for Intel Macs".

## (Officially) Supported Mac Models 

![macOS_talahon](https://github.com/user-attachments/assets/2e3c53c7-4b33-4968-8505-e15247619004)

**Source**: https://www.apple.com/os/macos/

## Usable SMBIOSes

- **Noetbooks**: `MacBookPro16,1` and `MacBookPro16,4`
- **Desktops**: `iMac20,1` and `iMac20,2`
- The rest: dropped

It looks like Dortania has a lot of work to do to include all of these newly dropped Intel Macs into OCLP…

## New features

- Nothing really noteable besided an overhaul of Spotlight

## OpenCore Patcher Support

- Unknown yet. Waiting for official statements by Dortania et. al.

## Status

- Don't install macOS Tahoe yet! Wait for updated kexts and OpenCore builds.

## Technical challenges

- Metal 4 support
- LAN – all my systems with an I219 NIC stopped working under macOS Tahoe
- [USB](/Enable_Features/USB_Tahoe.md) 

## Observations

- Board-ID Skip and RestrictEvents still seem to work
- On my Ivy Bridge Notebook installation works without effor. But the system crashes after completing the first stage of the install assistant before attempting to create the user account.
- Couldn't get it to work on my Z490 system with a Comet Lake CPU and a Polaris GPU. I've read that disabling it and using the iGPU works

## Further Resources

- [macOS Tahoe 26 on Unsupported Macs](https://forums.macrumors.com/threads/macos-tahoe-26-on-unsupported-macs-discussion.2458481/) – Thread on macrumors.com. A good source for current developments regarding OCLP
