[![OpenCore Version](https://img.shields.io/badge/OpenCore_Version:-0.9.4+-success.svg)](https://github.com/acidanthera/OpenCorePkg) ![macOS](https://img.shields.io/badge/Supported_macOS:-≤26.x-white.svg)

# Installing macOS Tahoe on Kaby Lake and newer

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

---

[← **HOME**](/OCLP4Hackintosh/README.md) | [**NEXT:  Preparations →**](Prep.md)
