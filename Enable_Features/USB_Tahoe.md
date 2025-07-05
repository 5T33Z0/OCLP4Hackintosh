# Fixing USB issues in macOS Tahoe

**TABLE of CONTENTS**

- [Introduction](#introduction)
- [Fixing USB port mapping for compatibility with macOS Tahoe](#fixing-usb-port-mapping-for-compatibility-with-macos-tahoe)
	- [Option 1: Automated fix using USBMap Script](#option-1-automated-fix-using-usbmap-script)
	- [Option 2: Editing the `info.plist` of the `USBMap.kext` manually](#option-2-editing-the-infoplist-of-the-usbmapkext-manually)
	- [Screenshots](#screenshots)
- [Fixing USB Wi-Fi Dongles in macOS Tahoe](#fixing-usb-wi-fi-dongles-in-macos-tahoe)
- [Credits](#credits)

## Introduction
With the release of macOS Tahoe beta, new USB issues arose concerning USBMap kexts as well as the IOUSBFamily.kext, which is required in order for USB Wi-Fi dongles to work

## Fixing USB port mapping for compatibility with macOS Tahoe

Apple introduced some changes to the USB stack in macOS 26 “Tahoe”, causing the standard `USBMap.kext` to stop functioning as expected. This guide explains how to adapt an existing `USBMap.kext` to work reliably under macOS Tahoe without having to create a new USBmap kext or resorting to [mapping USB ports via ACPI](https://github.com/5T33Z0/OC-Little-Translated/tree/main/03_USB_Fixes/ACPI_Mapping_USB_Ports) which is a rather tedious process.

### Option 1: Automated fix using USBMap Script

CorpNewt recently updated his [**USBMap**](https://github.com/corpnewt/USBMap) port mapping Python script. It also contains a script called `USBMapInjectorEdit`. It can now update the info.plist of the kext for macOS 26 compatibility automatically:

- Run `USBMapInjectorEdit` script (.command in macOS or .bat in Windows)
- Drag the USBMap.kext into the Terminal window and press Enter 
- In the Main windwow you now have the following options: <br>![Update_keys](https://github.com/user-attachments/assets/ab0a2a8c-7149-4c44-a379-93fcc67c55e5)
- You only have to press <kbd>u</kbd> and press <kbd>Enter</kbd> – That's it.
- In the `config.plist`, change the `MinKernel` setting for your USBMap.kext to `19.0.0`
- Save your config, boot into macOS Tahoe and test it.

> [!TIP]
>
> If you've changed the SMBIOS of your system, you also must adjust the `model` entry in the `info.plist` as well. Otherwise the USB port map won't work. 
> 
> Ideally, you would not change the SMBIOS at all, or even better, use the SMBIOS that's native to your CPU family. Next, add the [board-id check skip](https://github.com/dortania/OpenCore-Legacy-Patcher/blob/main/payloads/Config/config.plist#L220-L243) (Booter/Patch), [RestrictEvents.kext](https://github.com/acidanthera/RestrictEvents) and `revpatch=sbvmmm` boot-arg to your EFI and `config.plist`. This allows booting macOS with an unsupported SMBIOS/Board-id as well as receiving System Updates, which wouldn't be possible otherwise.
>
> Alternatively, you could temporarily enable the `XhciPortlimit` quirk to get USB working (not recommended, since it's a hack that could lead to system instabilities).

### Option 2: Editing the `info.plist` of the `USBMap.kext` manually

- Mount your EFI folder
- Open Finder
- Press Shift+G
- Enter 
	```text
	/Volumes/EFI/EFI/oc/Kexts/USBMap.kext/Contents/Info.plist
	```
- Open the plist with ProperTree or the plist editor of your choice
- In the plist, navigate to: `IOKitPersonalities` &rarr; XHC/EHC/etc. (Name depends on type of controller) &rarr; `IOProviderMergeProperties` &rarr; `ports` 
- There you will find the entries for all the ports you mapped (`HS` and `SS`)
- Open each one of them and look for the entry `port`
- Copy and paste the line and rename it to `usb-port-number` so that it works in macOS 26. (see "Screenshots" for details)
- Repeat this procedure for the rest of the ports
- Once you are done with editing, save the file
- Next, open your config.plist and change `MinKernel` to `19.0.0` for `USBMap.kext`
- Save the config
- Boot into macoOS Test and it

> [!TIP]
>
> If you've changed the SMBIOS of your system, you also must adjust the `model` entry in the `info.plist` as well. Otherwise the USB port map won't work. Ideally, you would not change the SMBIOS at all, or even better, use the SMBIOS that's native to your CPU family. Next, add the [board-id check skip](https://github.com/dortania/OpenCore-Legacy-Patcher/blob/main/payloads/Config/config.plist#L220-L243) (Booter/Patch), [RestrictEvents.kext](https://github.com/acidanthera/RestrictEvents) and `revpatch=sbvmmm` boot-arg to your EFI and `config.plist`. This allows booting macOS with an unsupported SMBIOS/Board-id as well as receiving System Updates, which wouldn't be possible otherwise.
>
> Alternatively, you could temporarily enable the `XhciPortlimit` quirk to get USB working (not recommended, since it's a hack that could lead to system instabilities).

### Screenshots

Listed below are screenshots of the `info.plist` in the USBMap.kext. On the left is the original file and on the right the modified plist with the changes required for macOS Tahoe for direct comparison:

Previous macOS | macOS Tahoe
---------------|--------------
![old](https://github.com/user-attachments/assets/dcea4dc7-37bb-4fa0-acff-474710ea96a7) | ![ports_new](https://github.com/user-attachments/assets/5cfd37e3-5e7c-40a9-ae21-7fd0796f7881)

## Fixing USB Wi-Fi Dongles in macOS Tahoe

With the release of macOS Tahoe, Apple removed legacy kernel extensions like `IOUSBFamily.kext`, breaking compatibility with many third-party USB Wi-Fi dongles that depend on this component — particularly those using older USB 2.0 (EHCI) interfaces. If your USB Wi-Fi dongle is no longer recognized, restoring `IOUSBFamily.kext` via root patching is currently the only known workaround.

This guide walks you through using **OCLP Mod**, a modified version of OpenCore Legacy Patcher, to reintroduce this functionality on unsupported systems. Be aware that patching the USB stack can introduce side effects (e.g., HID receivers may stop working), so apply this only if you're experiencing USB-related issues with Wi-Fi adapters.

**Instructions**:

- Ensure that your system is connected to the internet. This is mandatory because OCLP Mod needs to download additional files required for root patching!
- Download [**OCLP-Mod.pkg**](https://github.com/laobamac/OCLP-Mod/releases) and install it
- Run it. The GUI is in Chinese, unfortunately
- Open the settings:<br>![OCLP_mod01](https://github.com/user-attachments/assets/275e1bea-617b-4196-8c40-bb2c24eb73f0)
- Click on the shown Tab and enable the options you need (here USB) and press "Ok" at the bottom:<br>![iousbfamily](https://github.com/user-attachments/assets/e4b219be-4e44-4042-bf1a-a6475b3875fc)
- Press the upper right button for the root patching:<br>![oclp_mod01](https://github.com/user-attachments/assets/ad42427a-3726-480e-89a3-d2bd98754c3c)
- Next, press the upper button to install patches and wait until patching is completed:<br>![oclp_mod02](https://github.com/user-attachments/assets/25e5fc28-05de-4cdd-ac3d-d5a28d06d1db)
- If required, it will automatically download KDK or Metalibs
- Restart macOS when prompted to
- Once macOS is up and running again, USB Wi-FI dongles should work again.

## Credits
- **JustFun** from hackintosh-forum.de for the [manual info-plist fix](https://www.hackintosh-forum.de/forum/thread/60350-wwdc-2025-macos-26-hackintosh/?postID=802582#post802582)
- **CorpNewt** for [USBMap](https://github.com/corpnewt/USBMap)
