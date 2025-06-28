[![OpenCore Version](https://img.shields.io/badge/OpenCore_Version:-0.9.4+-success.svg)](https://github.com/acidanthera/OpenCorePkg) ![macOS](https://img.shields.io/badge/Supported_macOS:-≤26b2-white.svg)

# Re-enabling Audio in macOS Tahoe

**TABLE of CONTENTS**

- [Introduction](#introduction)
- [Re-enabling Audio-support in macOS Tahoe](#re-enabling-audio-support-in-macos-tahoe)
  - [Option 1: Using OCLP-Mod](#option-1-using-oclp-mod)
    - [Pre-Requisites](#pre-requisites)
    - [Instructions](#instructions)
  - [Option 2: Using VoodooHDA](#option-2-using-voodoohda)
- [Troubleshooting](#troubleshooting)

---

## Introduction
In macOS Tahoe beta 2, on-board audio stopped working. Since `AppleALC.kext` was updated for macOS 26 compatibility, the culprit was found quick: `AppleHDA` has been removed from Apple's latest and last x86-compatible iteration of macOS.

***So what does AppleHDA do?***

`AppleHDA` is a core audio driver framework used in macOS to manage audio hardware functionality. It acts as a bridge between the operating system’s audio stack and the hardware, enabling audio input and output for various devices like speakers, microphones, and headsets. Below is an explanation of how it works and what it does, tailored to be concise yet comprehensive.

***How does AppleHDA work?***

> AppleHDA is a fairly simple kernel extension for setting up audio support for devices. It does this by parsing the Layout ID detected in the firmware (via DeviceProperties) and loads the associated audio map off of disk.
>
> - These maps [used to] be located at `/System/Library/Extensions/AppleHDA.kext/Contents/Resources/`
> - Layout IDs are present on the HDEF device in IOService as layout-id, data is generally presented in Hexadecimal form but supports Integer and ASCII
>- Layout data is generally presented as 2 files: `layout{x}.xml` and `Platforms.xml`. `Platforms.xml` represents chipset definitions while `layout.xml` is reserved for model-specific data.
>
> […]
**Source**: [Mykola's Blog](https://khronokernel.com/macos/2021/10/24/OCLP-AUDIO.html)

***So why is there no sound?***

With `AppleHDA` gone from S/L/E, `AppleALC` — which is designed to inject these codec-specific files into the macOS HDEF device to enable audio — lacks a target framework to work with. As a result, onboard audio fails to function in macOS Tahoe.

## Re-enabling Audio-support in macOS Tahoe

### Option 1: Using OCLP-Mod

Since there's no official OCLP version available for macOS Tahoe yet, you can use [**OCLP Mod**](https://github.com/laobamac/OCLP-Mod/) to apply root patches – which will also install AppleHDA, thereby re-enabling audio. To download the latest build, you need a Github account.

> [!WARNING]
>
> Don't use OCLP-Mod if your system requires root patches for enabling graphics output (either iGPU or GPU). In this case, you should wait for an official OCLP release by Dortania! Don't blame me if your system doesn't boot into macOS after applying root patches with OCLP Mod!

#### Pre-Requisites

The following changes are required so that root-patches can be applied

1. In your `config.plist`, change `csr-active-config` to `03080000`
2. Add [`AMFIPass.kext`](https://github.com/bluppus20/AMFIPass/releases) to `EFI/OC/Kexts` and your `config.plist`. 
3. Update `AppleACL.kext` to the [**latest nightly version**](https://dortania.github.io/builds/?product=AppleALC&viewall=true)
4. Add the following NVRAM Settings to your config:
    ```xml
    […]
    <dict>
        <key>NVRAM</key>
        <dict>
            <key>Add</key>
            <dict>
                <key>4D1FDA02-38C7-4A6A-9CC6-4BCCA8B30102</key>
                <dict>
                    <key>OCLP-Settings</key>
                    <string>-allow_fv -allow_amfi</string>
                </dict>
            <key>Delete</key>
            <dict>
                <key>4D1FDA02-38C7-4A6A-9CC6-4BCCA8B30102</key>
                <array>
                    <string>OCLP-Settings</string>
                </array>
            </dict>
        </dict>
    </dict>
    […]
    ```
5. Save your `config.plist` and reboot

#### Instructions

- Ensure that your system is connected to the internet
- Download [**OCLP-Mod.pkg**](https://github.com/laobamac/OCLP-Mod/actions) and install it
- Run it. The GUI is in Chinese, unfortunately
- Press the upper right button for the root patching:<br>![oclp_mod01](https://github.com/user-attachments/assets/ad42427a-3726-480e-89a3-d2bd98754c3c)
- Next, press the upper button to install patches and wait until patching is completed:<br>![oclp_mod02](https://github.com/user-attachments/assets/25e5fc28-05de-4cdd-ac3d-d5a28d06d1db)
- If required, it will automatically download KDK or Metalibs
- Restart macOS when prompted to
- Once macOS is up and running again, the audio device will be present and working as shown in this example:<br>![Hackintool](https://github.com/user-attachments/assets/eae77186-9515-4f3f-8a2b-1309b7d769f7)

### Option 2: Using VoodooHDA

- As a fallback, install `VoodooHDA.kext`, an alternative audio driver that doesn’t rely on `AppleHDA`.  
- Download from [VoodooHDA GitHub](https://github.com/acidanthera/VoodooHDA).  
- Add to your OpenCore EFI and `config.plist`.  
- Be aware that `VoodooHDA` may have lower audio quality or compatibility issues compared to `AppleALC`.

## Troubleshooting

- **No audio devices detected**: Verify `AppleALC.kext` or `VoodooHDA.kext` is loading properly using:
    ```bash
    kextstat | grep -i applealc
    kextstat | grep -i voodoohda
    ```
- **Incorrect Layout ID**: Double-check your codec and Layout ID in `config.plist`.  
- **Kernel cache issues**: Rebuild the kernel cache with:
    ```bash
    sudo kextcache -i /
    ```
 - **SIP conflicts**: Ensure that SIP is disabled