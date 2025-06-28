# Fixing `USBMap.Kext` for use with macOS 26 "Tahoe"

Apple introduced some changes to the USB stack in macOS 26 “Tahoe”, causing the standard `USBMap.kext` to stop functioning as expected. This guide explains how to adapt an existing `USBMap.kext` to work reliably under macOS Tahoe without needing to map ports via an SSDT.

## Option 1: Automated fix using USBMap Script

CorpNewt recently updated his [**USBMap**](https://github.com/corpnewt/USBMap) port mapping Python script. It also contains a script called `USBMapInjectorEdit`. It can now update the info.plist of the kext for macOS 26 compatibility automatically:

- Run `USBMapInjectorEdit` script (.command in macOS or .bat in Windows)
- Drag the USBMap.kext into the Terminal window and press Enter 
- In the Main windwow you now have the following options: <br>![Update_keys](https://github.com/user-attachments/assets/ab0a2a8c-7149-4c44-a379-93fcc67c55e5)
- You only have to press <kbd>u</kbd> and press <kbd>Enter</kbd> – That's it.
- In the `config.plist`, change the `MinKernel` setting for your USBMap.kext to `19.0.0`
- Save your config, boot into macOS Tahoe and test it.

## Option 2: Editing the `info.plist` of the `USBMap.kext` manually

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
> If you've changed the SMBIOS of your system, you also must adjust the `model` entry in the `info.plist` as well. Otherwise the USB port map won't work. Ideally, you don't change your SMBIOS at all. Instead, you add the [board-id check skip](https://github.com/dortania/OpenCore-Legacy-Patcher/blob/main/payloads/Config/config.plist#L220-L243) (Booter/Patch), [RestrictEvents.kext](https://github.com/acidanthera/RestrictEvents) and `revpatch=sbvmmm` boot-arg to your EFI and `config.plist`. This allows booting macOS with an unsupported SMBIOS/Board-id and receiving System Updates, which wouldn't be possible otherwise.

### Screenshots

Listed below are screenshots of the `info.plist` in the USBMap.kext. On the left is the original file and on the right the modified plist with the changes required for macOS Tahoe for direct comparison:

Previous macOS | macOS Tahoe
---------------|--------------
![old](https://github.com/user-attachments/assets/dcea4dc7-37bb-4fa0-acff-474710ea96a7) | ![ports_new](https://github.com/user-attachments/assets/5cfd37e3-5e7c-40a9-ae21-7fd0796f7881)

## Credits
- **JustFun** from for his original instructions [fix](https://www.hackintosh-forum.de/forum/thread/60350-wwdc-2025-macos-26-hackintosh/?postID=802582#post802582)
- **CorpNewt** for [USBMap](https://github.com/corpnewt/USBMap)
