[![OpenCore Version](https://img.shields.io/badge/OpenCore_Version:-0.9.4+-success.svg)](https://github.com/acidanthera/OpenCorePkg) ![macOS](https://img.shields.io/badge/Supported_macOS:-≤15.x-white.svg)

# Installing macOS 13+ on Haswell/Broadwell systems

:construction: **OVERHAUL in PROGRESS**

## Introduction
Although installing and running macOS Ventura and newer on Wintel machines with Haswell/Broadwell CPUs is possible with OpenCore and the [**OpenCore Legacy Patcher**](https://github.com/dortania/OpenCore-Legacy-Patcher/tree/main) (OCLP), it’s not officially supported by Dortania – their support is limited to Apple Macs. Since no Hackintosh guide exists, I created this one in order to bridge the gap. I wrote it based on my experiences, analyzing the config, EFI folder, and logs after building OpenCore with OCLP for a Haswell system.

## Scope and limitations
This guide is intended to provide general information for adjusting your EFI and `config.plist` to install and run macOS Ventura and newer on unsupported Wintel systems. It is not a comprehensive configuration guide. Please refrain from using the "report issue" function to seek individualized assistance for fixing your config. Such issue reports will be closed immediately.

## Current status

| ⚠️ Current Status |
|:----------------------------|
| No iGPU patches for macOS Tahoe available. |

Check the links below for in-depth documentation about components/features that have been removed from macOS 12 and newer and the impact this has on systems prior to Kaby Lake. Keep in mind that this documentation targets real Macs, so certain issues may not apply to Wintel systems:

- [Status of OpenCore Legacy Patcher Support for macOS Tahoe](https://github.com/dortania/OpenCore-Legacy-Patcher/issues/1167)  
- [Status of OpenCore Legacy Patcher Support for macOS Sequoia](https://github.com/dortania/OpenCore-Legacy-Patcher/issues/1136)  
- [Status of OpenCore Legacy Patcher Support for macOS Sonoma](https://github.com/dortania/OpenCore-Legacy-Patcher/issues/1076)  
- [Status of OpenCore Legacy Patcher Support for macOS Ventura](https://github.com/dortania/OpenCore-Legacy-Patcher/issues/998)  
- [Legacy Metal Support and macOS Ventura–Sequoia](https://github.com/dortania/OpenCore-Legacy-Patcher/issues/1008)  
- [Legacy Non-Metal Support and macOS Big Sur–Sequoia](https://github.com/dortania/OpenCore-Legacy-Patcher/issues/108)

## Technical background

### How Haswell/Broadwell systems are affected
In macOS Ventura, support for CPU families prior to Kaby Lake was dropped. For Haswell/Broadwell CPUs this mainly affects integrated graphics and Metal support.

The approach described in this guide prepares the OpenCore configuration with the required patches, settings, and kexts for installing and running macOS Ventura+ and then adds iGPU/GPU drivers post-install using OpenCore Legacy Patcher.

[← **HOME**](/OCLP4Hackintosh/README.md) | [**Next: Preparations →**](Prep.md)
