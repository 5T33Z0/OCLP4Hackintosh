# Using `AirportItlwm.kext` for Intel Wi-Fi/BT Cards in macOS Sequoia and Tahoe

![intel_spoof07](https://github.com/user-attachments/assets/255406e1-554e-4b6c-9b67-5d24b9fcb962)

> [!IMPORTANT]
> 
> [**laobamac's itlwm fork**](https://github.com/laobamac/itlwm/releases) restores native `AirportItlwm.kext` support in macOS Sequoia and Tahoe — **without requiring OCLP root patches!** It also supports **AppleVTD**.

## About

Apple removed support for several system kexts and frameworks required by Broadcom Wi-Fi cards in macOS Sequoia. Although Apple never shipped Macs with Intel Wi-Fi/BT cards, `AirportItlwm.kext` also relied on some of these system components to function properly. Consequently, users with Intel Wi-Fi/BT cards had to resort to `itlwm.kext` instead.

Since `itlwm.kext` exposes the Wi-Fi card as a regular network interface in macOS rather than integrating it with Apple's native Wi-Fi stack, this comes with some limitations. For example, Airport Utility can no longer be used to connect to Wi-Fi hotspots, and a separate application ([HeliPort](https://github.com/OpenIntelWireless/HeliPort/releases)) is required to manage Wi-Fi connections. Some Apple services, such as Find My Mac, may also be affected.

> [!TIP]
> 
> If you also need an AppleVTD-compatible build of `itlwm.kext`, consider [jzrodriguez98's fork](https://github.com/jzrodriguez98/itlwm/releases), which incorporates laobamac's AirportItlwm fixes.

---

## Instructions

Choose the appropriate installation path depending on your current configuration.

### 1. Coming from a root-patched installation

If you previously applied OCLP root patches to enable Intel Wi-Fi, revert the old Wi-Fi patching changes first.

**Step 1: Revert OCLP root patches**

- Open OCLP-Mod.
- Select **Post-Install Root Patch → Revert Root Patches**.
- Reboot into macOS.

> [!CAUTION]
> 
> Only revert root patches if they were applied exclusively for enabling Wi-Fi. If your system requires other root patches, for example for graphics acceleration, do not revert. Keep the patches required by your other hardware.

**Step 2: Clean up the old EFI configuration**

Mount your EFI System Partition and make a backup of your working EFI folder.

Open `config.plist` and remove the old Wi-Fi patching configuration:

- **Kernel → Block**
  - Remove the `com.apple.iokit.IOSkywalkFamily` blocking entry.
- **Kernel → Add**
  - Remove `IOSkywalkFamily.kext`.
  - Remove `IO8021FamilyLegacy.kext`.
  - Remove the old Ventura `AirportItlwm.kext` if present.
  - Disable `AMFIPass.kext` if it was only added for the old Wi-Fi patching method and is not needed elsewhere.
- **DeviceProperties**
  - Delete `IOName` proerty for your WiFi Card, if present.

Keep unrelated kexts, quirks and NVRAM settings required by your system.

**Step 3: Install the new AirportItlwm**

Continue with section 2 below.

### 2. Starting from a vanilla installation

If you are starting with an unpatched installation of macOS Sequoia or Tahoe, you can install the new kext directly.

**Step 1: Prepare your EFI**

- Mount your EFI System Partition.
- Back up your existing EFI folder to a FAT32-formatted USB flash drive.
- Open your `config.plist`.

**Step 2: Configure OpenCore**

Configure the appropriate `DisableIoMapper`/`DisabledIoMapper` quirk if required by your hardware and OpenCore version. For Tahoe, check the fork's release notes for the recommended setting.

**Step 3: Download AirportItlwm**

- Visit [laobamac's itlwm releases](https://github.com/laobamac/itlwm/releases).
- Download the appropriate `AirportItlwm.kext` build for your macOS version:
  - macOS Sequoia
  - macOS Tahoe

Make sure you use the correct build for your installed macOS version.

**Step 4: Install the kext**

- Copy `AirportItlwm.kext` into `EFI/OC/Kexts`.
- Add it to **Kernel → Add** in your `config.plist`.
- Make sure the entry is enabled.
- Save your `config.plist`.
- Adjust MinKernel and MaxKernel Settings:
	- macOS Sequoia: 24.0.0 (Min), 24.99.99 (Max)
	- macOS Tahoe: 25.0.0 (Min)

**Important:** Do not install the old OCLP Wi-Fi dependencies. The new fork does not require the old `IOSkywalk.kext` and `IO8021FamilyLegacy.kext` injection method.

You should not need the old Ventura AirportItlwm build, OCLP root patches, or the associated AMFIPass configuration for this method.

### 3. Final checks

Before rebooting, verify the following:

- [ ] Correct AirportItlwm build for Sequoia or Tahoe.
- [ ] `AirportItlwm.kext` is present in `EFI/OC/Kexts`.
- [ ] `AirportItlwm.kext` is enabled in `Kernel → Add`.
- [ ] Old Wi-Fi patching entries have been removed.
- [ ] No conflicting `Itlwm.kext` is enabled.
- [ ] Your EFI has been backed up.

Reboot into macOS and check whether Wi-Fi is working.

If you previously used `Itlwm.kext`, you can also remove HeliPort from your login items if you no longer need it.

---

## Building itlwm and AirportItlwm from Source Code (Requires Xcode)

- Download the itlwm source code and extract it to your Downloads folder.
- Open Terminal.
- Navigate to the `itlwm-main` folder:

  ```bash
  cd ~/Downloads/itlwm-main
  ```

- Add the MacKernelSDK:

  ```bash
  git clone https://github.com/acidanthera/MacKernelSDK
  ```

- Delete the existing firmware binary (if present):

  ```bash
  rm include/FwBinary.cpp
  ```

- Build the firmware:

  ```bash
  xcodebuild -project itlwm.xcodeproj -target fw_gen -configuration Release -sdk macosx
  ```

- Build the kexts:

  ```bash
  xcodebuild -alltargets -configuration Release
  ```

This will compile `itlwm.kext` and the available `AirportItlwm.kext` variants for supported macOS versions, including Sequoia and Tahoe. The compiled kexts will be located in `~/Downloads/itlwm-main/build/Release`.

- If you only want to compile AirportItlwm for Sequoia:

  ```bash
  xcodebuild -project itlwm.xcodeproj -scheme AirportItlwm-Sequoia \
    -configuration Release ARCHS=x86_64 CODE_SIGNING_ALLOWED=NO build
  ```

- If you only want to compile AirportItlwm for Tahoe:

  ```bash
  xcodebuild -project itlwm.xcodeproj -scheme AirportItlwm-Tahoe \
    -configuration Release ARCHS=x86_64 CODE_SIGNING_ALLOWED=NO build
  ```

> [!TIP]
> If you want to include AirportItlwm variants for multiple macOS versions in your OpenCore EFI, rename the kexts accordingly (e.g. `AirportItlwm_Sequoia.kext` or `AirportItlwm_Tahoe.kext`), then add them to `EFI/OC/Kexts` and `config.plist`.
>
> Adjust the `MinKernel` and `MaxKernel` settings so that only the appropriate kext is loaded for each macOS version:
>
> <img width="1489" height="124" alt="AirportItlwm Settings" src="https://github.com/user-attachments/assets/a5cf38f2-2072-479d-b41d-fe6a375917c2" />

---

## Credits and Thank Yous

- laobamac for the OCLP-Mod project and itlwm fork.
- lifeknife10A for the original [workaround](https://github.com/OpenIntelWireless/itlwm/issues/1009#issuecomment-2370919270).
- sughero for additional information about kext loading order.
- stefanalmare for pointing me to this solution.
- lzhoang2801 for OpCore Simplify and further explanations about the effect on iServices.
- Mirone for [Wi-Fi Patcher Pro](https://github.com/Mirone/WiFiPatcherPro).
