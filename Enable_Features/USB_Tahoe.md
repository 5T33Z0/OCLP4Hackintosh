# Fixing `USBMap.Kext` for use with macOS 26 "Tahoe"

Apple introduced significant changes to the USB stack in macOS 26 “Tahoe”, causing the standard `USBMap.kext` to stop functioning as expected. This guide explains how to adapt an existing `USBMap.kext` to work reliably under macOS Tahoe without needing to map ports via an SSDT.

## Steps to Adapt USBMap.kext for macOS 26

### 1. Duplicate the Kext
- Create a copy of your existing `USBMap.kext`.
- Rename it to, e.g., `USBMap_Tahoe.kext`.

### 2. Modify the Info.plist in `USBMap_Tahoe.kext`
Update the following entries in the `Info.plist` file of `USBMap_Tahoe.kext`:
- `CFBundleIdentifier`: Set to `com.apple.driver.AppleUSBHostMergeProperties`
- Under `IOKitPersonalities/XHC/`: `IOClass`: Set to `AppleUSBHostMergeProperties` (if it isn't already)
- `IOProviderClass`: Set to `AppleUSBXHCIPCI`
- Under `IOKitPersonalities/XHC/IOProviderMergeProperties/ports`:
  - Change every instance of `port` to `usb-port-number`. This is critical due to Apple's updated USB architecture in macOS 26.

### 3. Update `config.plist`
Include both kexts in your OpenCore `config.plist` with version-specific kernel settings:
- For `USBMap.kext`:
  - Set `MaxKernel` to `24.9.9` (for macOS 15 Sequoia and earlier).
- For `USBMap_Tahoe.kext`:
  - Set `MinKernel` to `25.0.0` (for macOS 26 Tahoe and later).

This ensures the appropriate kext loads based on the used macOS version.

## Screenshots

Listed below are screenshots of the `info.plist` in the USBMap.kext. On the left is the original file and on the right the modified plist with the changes required for macOS Tahoe for direct comparison:

Previous macOS | macOS Tahoe
---------------|--------------
![old](https://github.com/user-attachments/assets/dcea4dc7-37bb-4fa0-acff-474710ea96a7) | ![new](https://github.com/user-attachments/assets/d89219c1-2ed5-4989-b211-ed173b1b12ca)

## Credits
**JustFun** from [Hackintosh-Forum.de](https://www.hackintosh-forum.de/forum/thread/60350-wwdc-2025-macos-26-hackintosh/?postID=802582#post802582)
