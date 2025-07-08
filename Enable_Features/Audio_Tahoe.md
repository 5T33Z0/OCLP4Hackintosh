[![OpenCore Version](https://img.shields.io/badge/OpenCore_Version:-1.0.5+-success.svg)](https://github.com/acidanthera/OpenCorePkg) ![macOS](https://img.shields.io/badge/Supported_macOS:-≤26b2-white.svg)

# Re-enabling Audio in macOS Tahoe beta 2+

**TABLE of CONTENTS**

- [Introduction](#introduction)
- [Re-enabling Audio-support in macOS Tahoe](#re-enabling-audio-support-in-macos-tahoe)
  - [Option 1: Using OCLP-Mod](#option-1-using-oclp-mod)
    - [Pre-Requisites](#pre-requisites)
    - [Instructions](#instructions)
  - [Option 2: Using VoodooHDA](#option-2-using-voodoohda)
    - [Pre-Requisites](#pre-requisites-1)
    - [Instructions](#instructions-1)
- [Troubleshooting](#troubleshooting)
- [Credits and Thank Yous](#credits-and-thank-yous)

---

## Introduction
In macOS Tahoe beta 2, analog audio stopped working. Since `AppleALC.kext` was updated for macOS 26 compatibility, the culprit was found quick: `AppleHDA` has been removed from Apple's latest and last x86-compatible iteration of macOS.

***So what does AppleHDA do?***

`AppleHDA` is a core audio driver framework used in macOS to manage audio hardware functionality. It acts as a bridge between the operating system’s audio stack and the hardware, enabling audio input and output for various devices like speakers, microphones, and headsets. Below is an explanation of how it works and what it does, tailored to be concise yet comprehensive.

***How does AppleHDA work?***

> AppleHDA is a fairly simple kernel extension for setting up audio support for devices. It does this by parsing the Layout ID detected in the firmware (via DeviceProperties) and loads the associated audio map off of disk.
>
> - These maps [used to be] located at `/System/Library/Extensions/AppleHDA.kext/Contents/Resources/`
> - Layout IDs are present on the HDEF device in IOService as layout-id, data is generally presented in Hexadecimal form but supports Integer and ASCII
> - Layout data is generally presented as 2 files: `layout{x}.xml` and `Platforms.xml`. `Platforms.xml` represents chipset definitions while `layout.xml` is reserved for model-specific data.
>
> […]

**Source**: [Mykola's Blog](https://khronokernel.com/macos/2021/10/24/OCLP-AUDIO.html)

***So why is there no sound?***

Not only AppleHDA has been removed, but the associated dylib has also been deleted from the `dyld_shared_cache`[^1]. To restore functionality, the Kernel Development Kit (KDK) from Tahoe beta 1 is required to overwrite the root directory and reinstall the missing files. Since `AppleHDA` operates at the system user level and cannot be injected via OpenCore (otherwise the system crashes during boot), both the kext and dylib need to be reinstated through root patches.

[^1]: **Source**: [laobamac_yyds](https://www.insanelymac.com/forum/topic/361249-oclp-mod-releaseissuediscussion/page/2/#findComment-2835718)

## Re-enabling Audio-support in macOS Tahoe

### Option 1: Using OCLP-Mod

Since there's no official OCLP version available for macOS Tahoe yet, you can use [**OCLP Mod**](https://github.com/laobamac/OCLP-Mod/) to apply root patches – which will also install AppleHDA, thereby re-enabling audio. To download the latest build, you need a Github account.

> [!CAUTION]
>
> Don't use OCLP-Mod if your system requires root patches for enabling graphics output (either iGPU or GPU). In this case, you should wait for an official OCLP release by Dortania! Don't blame me if your system doesn't boot into macOS after applying root patches with OCLP Mod!

#### Pre-Requisites

- In general: an OpenCore EFI and `config.plist` prepared to run macOS 13 and newer is required (&rarr; see [Configuration Guides](https://github.com/5T33Z0/OCLP4Hackintosh/tree/main?tab=readme-ov-file#configuration-guides)).
- If macOS Tahoe is up and running already, the following changes are required so that root-patches can be applied:
  - In your `config.plist`, change `csr-active-config` to `03080000`
  - Add [`AMFIPass.kext`](https://github.com/bluppus20/AMFIPass/releases) to `EFI/OC/Kexts` and your `config.plist`. 
  - Update `AppleACL.kext` to the [**latest nightly version**](https://dortania.github.io/builds/?product=AppleALC&viewall=true)
  - Add the following NVRAM Settings to your config:
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
  - Save your `config.plist` and reboot

#### Instructions
- Ensure that your system is connected to the internet. This is mandatory because OCLP Mod needs to download additional files required for root patching!
- Download [**OCLP-Mod.pkg**](https://github.com/laobamac/OCLP-Mod/releases) and install it
- Run it. The GUI is in Chinese, unfortunately
- Open the settings:<br>![OCLP_mod01](https://github.com/user-attachments/assets/275e1bea-617b-4196-8c40-bb2c24eb73f0)
- Click on the shown Tab and enable the options you need (here AppleHDA) and press "Ok" at the bottom:<br>![Applehda](https://github.com/user-attachments/assets/73a5738c-aceb-42c2-908a-a0118b04dc9a)
- Press the upper right button for the root patching:<br>![oclp_mod01](https://github.com/user-attachments/assets/ad42427a-3726-480e-89a3-d2bd98754c3c)
- Next, press the upper button to install patches and wait until patching is completed:<br>![oclp_mod02](https://github.com/user-attachments/assets/25e5fc28-05de-4cdd-ac3d-d5a28d06d1db)
- If required, it will automatically download KDK or Metalibs
- Restart macOS when prompted to
- Once macOS is up and running again, the audio device will be present and driven by "AppleHDADriver" as shown here (example):<br>![after_patching](https://github.com/user-attachments/assets/99c7eb4d-6306-485f-820e-33619b19d239)

### Option 2: Using VoodooHDA
As a fallback, you can use `VoodooHDA.kext`, an alternative audio driver that doesn’t rely on `AppleHDA`. Luckily for us, Chris1111 has updated his [**VoodooHDA Installer for macOS Tahoe**](https://github.com/chris1111/VoodooHDA-Tahoe).

#### Pre-Requisites
- [Disable Gatekeeper](/Guides/Disable_Gatekeeper.md)
- Mount your EFI and open your `config.plist`
- Change `csr-active-config` to `03080000` or `850A0000`
- Disable `AppleALC.kext` if present
- Save your `config.plist` and reboot
- Continue with the instructions
 
#### Instructions
- Open Terminal
- Run the following command:
    ```bash
    git clone https://github.com/chris1111/VoodooHDA-Tahoe.git && cd $HOME/VoodooHDA-Tahoe && ./Package.command && open -R $HOME/VoodooHDA-Tahoe/VoodooHDA-Tahoe.pkg
    ```
- This will build the VoodooHDA Installer package:<br>![vodoohda_build](https://github.com/user-attachments/assets/25444a0b-5bb3-4c10-a14e-48f456398000)
- Once the build process has finished, Open Finder
- Navigate to your home folder (or press <kbd>CMD</kbd> + <kbd>Shift</kbd> + <kbd>H</kbd>)
- Open the "VoodooHDA-Tahoe folder"
- Double-click the `VoodooHDA-Tahoe.pkg` to install the driver and PrefPane
- Once that's done, reboot your system
- Open System Preferences and find the VoodooHDA PrefPane to select your audio source.
- Enjoy!

> [!NOTE]
> 
> - Be aware that `VoodooHDA` may have lower audio quality or compatibility issues compared to `AppleALC`.
> - Unlike AppleALC which can detect plugged in devices and switch audio sources automatically, you have to do it manually when using VoodooHDA!

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

## Credits and Thank Yous
- Dortania for OCLP
- laobamac for OCLP Mod
- bluppus20 for AMFIPass kext
- Slice for VoodooHDA kext
- Chris1111 for VoodooHDA-Tahoe
