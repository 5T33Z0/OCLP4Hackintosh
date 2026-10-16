[![OpenCore Version](https://img.shields.io/badge/OpenCore_Version:-1.0.5+-success.svg)](https://github.com/acidanthera/OpenCorePkg) ![macOS](https://img.shields.io/badge/Supported_macOS:26.x-white.svg)

# Re-enabling Audio in macOS Tahoe

## Introduction

In macOS Tahoe Developer Beta 2, analog onboard audio stopped working because Apple removed `AppleHDA.kext`, the underlying driver required by `AppleALC.kext`.

This change is related to Apple's transition to T2-managed audio, which began with the iMac Pro in 2017 and was later extended to other Intel Macs. Hackintosh systems lack a T2 chip and still rely on AppleHDA to enable analog onboard audio through AppleALC. With the legacy driver removed in Tahoe, an alternative solution is needed to restore audio functionality.

For more information on how AppleHDA works, see [Mykola's blog post on audio in macOS](https://khronokernel.com/macos/2021/10/24/OCLP-AUDIO.html).

This guide explains how to restore analog onboard audio using `SimpleHDA.kext`, without requiring root patches. **This method is intended for systems that do not require root patches for iGPU or dGPU support.**

## Fix: Adding `SimpleHDA.kext`

[SimpleHDA](https://github.com/laobamac/SimpleHDA) is a Lilu plugin developed by laobamac to restore native AppleHDA functionality in macOS Tahoe.

Instead of modifying the system volume, SimpleHDA restores the AppleHDA stack through OpenCore injection, allowing AppleALC to work with your existing codec configuration and `layout-id`.

This means you can continue using your existing AppleALC setup without reinstalling AppleHDA through root patches.

### Instructions

1. Download [SimpleHDA.kext](https://github.com/laobamac/SimpleHDA/releases) from the project's Releases page and extract it.
2. Mount your ESP and copy `SimpleHDA.kext` to `EFI/OC/Kexts/`.
3. Open `config.plist` with your preferred configuration editor and add `SimpleHDA.kext` under `Kernel > Add`.
4. Set `MinKernel` to `25.0.0` so that SimpleHDA loads only on macOS Tahoe and later.
5. Make sure the kext load order is correct: `Lilu.kext` → `AppleALC.kext` → `SimpleHDA.kext`
6. Add `keepsyms=1` to `boot-args`.
7. Save your configuration and reboot.

> [!IMPORTANT]
>
> Keep your existing AppleALC codec configuration and `layout-id`. SimpleHDA restores the missing driver functionality; it does not automatically correct an incompatible layout or an incorrectly configured audio codec.

### Volume slider language

Starting with SimpleHDA version 1.0.0, the `shda-lang=` boot argument is no longer required to set the language displayed in the volume slider. Instead, SimpleHDA uses the language specified by `prev-lang:kbd`.

If `prev-lang:kbd` is empty, the volume slider defaults to Chinese.

**Screenshot — `prev-lang:kbd` is empty:**

<img width="326" height="134" alt="Tahoe audio slider with an empty prev-lang:kbd property" src="https://github.com/user-attachments/assets/5153319f-a7b0-4ca0-aeea-8faf4d8e3474" />

**Screenshot — `prev-lang:kbd` is set to `en-US:0`:**

<img width="339" height="107" alt="Tahoe audio slider with prev-lang:kbd set to en-US:0" src="https://github.com/user-attachments/assets/fb98c4cc-0484-48ba-a7c2-ac9c10078aed" />

---

## Previous Fixes (Obsolete)

<details>
<summary><strong>Click to reveal</strong> (Obsolete)</summary><br>

### Using MyKextInstaller (Obsolete)

If your system doesn't require any root patches to run macOS, you can use **MyKextInstaller** to install an older version of the **AppleHDA** kext to re-enable analog audio. Follow the instrcutions on the [MyKextInstaller](https://github.com/Mirone/MyKextInstaller) repo to so.

> [!NOTE]
>
> There's also [**HDA Universal**](https://www.insanelymac.com/forum/topic/362932-hdauniversal-applehda-like-audio-kext-for-macos-tahoe-and-hackintosh-systems/), an alternative AppleHDA-like audio solution for macOS Tahoe and Hackintosh systems. However, since it is "developed" by MaLd0n/Olarila and the source code is not publicly available, it is not covered or recommended in this guide. 

### Using OCLP-Mod (Obsolete)

If your system does require root patches to run macOS Tahoe, you should use this option instead. Since there's no official OCLP version available for macOS Tahoe , you can use [**OCLP Mod**](https://github.com/laobamac/OCLP-Mod/) to apply root patches – it will also install an older version of AppleHDA, thereby re-enabling analog audio. To download the latest build, you need a Github account.

> [!CAUTION]
>
> Don't use OCLP-Mod if your system requires root patches for enabling graphics acceleration (either iGPU or GPU) in macOS Tahoe. In this case, you should wait for an official OCLP release by Dortania! Don't blame me if your system doesn't boot into macOS after applying root patches with OCLP-Mod!

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
- Run it. The GUI is in Chinese, unfortunately. For an english translation, you might want to look into [compiling a multilingual version](https://github.com/5T33Z0/OCLP4Hackintosh/blob/main/Guides/OCLP-mod_Multilingual.md)
- Open the settings:<br>![OCLP_mod01](https://github.com/user-attachments/assets/275e1bea-617b-4196-8c40-bb2c24eb73f0)
- Click on the shown Tab and enable the options you need (here AppleHDA) and press "Ok" at the bottom:<br>![Applehda](https://github.com/user-attachments/assets/73a5738c-aceb-42c2-908a-a0118b04dc9a)
- Press the upper right button for the root patching:<br>![oclp_mod01](https://github.com/user-attachments/assets/ad42427a-3726-480e-89a3-d2bd98754c3c)
- Next, press the upper button to install patches and wait until patching is completed:<br>![oclp_mod02](https://github.com/user-attachments/assets/25e5fc28-05de-4cdd-ac3d-d5a28d06d1db)
- If required, it will automatically download KDK or Metalibs
- Restart macOS when prompted to
- Once macOS is up and running again, the audio device will be present and driven by "AppleHDADriver" as shown here (example):<br>![after_patching](https://github.com/user-attachments/assets/99c7eb4d-6306-485f-820e-33619b19d239)

### Using VoodooHDA (Obsolete)
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

</details>

## Credits and Thank Yous
- Dortania for OCLP
- laobamac for SmpleHDA
- bluppus20 for AMFIPass kext
- Slice for VoodooHDA kext
- Chris1111 for VoodooHDA-Tahoe
