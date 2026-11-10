[![OpenCore Version](https://img.shields.io/badge/OpenCore_Version:-0.9.4+-success.svg)](https://github.com/acidanthera/OpenCorePkg) ![macOS](https://img.shields.io/badge/Supported_macOS:-≤15.x-white.svg)

# Installing macOS Ventura and newer on Ivy Bridge systems

## Introduction
Although installing macOS Ventura and newer on systems with Intel CPUs of the Ivy Bridge family can be achieved with OpenCore and the OpenCore Legacy Patcher (OCLP), it's not officially supported nor documented since Dortania only provides support for legacy Macs by Apple. I developed this guide based on my experiences trying to get macOS 13 and newer running on my Lenovo T530 Laptop but it is applicable to desktop systems as well since I factored in the necessary EFI and config adjustments.

| ⚠️ Important Status Updates |
|:----------------------------|
| Don't install macOS Tahoe yet if your system relies on iGPU support and does not have a supported discrete GPU! There are no root patches available yet to enable the Intel HD 4000 on-board graphics!

## How Ivy Bridge systems are affected
In macOS Ventura, support for CPU families prior to Kaby Lake was dropped. For Ivy Bridge systems this affects CPU Instructions (missing AVX 2.0 instructions), CPU Power Management (removed `ACPI_SMC_PlatformPlugin`), integrated Graphics and Metal support. So what we will do is prepare the config with the required patches, settings and kexts for installing and running macOS Ventura and then add iGPU/GPU drivers in Post-Install using OpenCore Legacy Patcher.

**This guide allows you to**: 

- Install or upgrade to macOS Ventura, Sonoma or Sequoia (OCLP doesn't support Tahoe yet)
- Re-Install iGPU/GPU drivers in Post-Install so hardware graphics acceleration is working
- Re-enable SMC CPU Power Management so you have proper CPU Power Management using `SSDT-PM`
- Use a native SMBIOS for Ivy Bridge CPUs for optimal performance (no more spoofing required)
- Install OTA updates [which wouldn't be possible otherwise](https://github.com/5T33Z0/OC-Little-Translated/tree/main/S_System_Updates#fixing-issues-with-system-update-notifications-in-macos-113-and-newer)

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

---

[← **HOME**](/OCLP4Hackintosh/README.md) | [**NEXT:  Preparations →**](Prep.md)
