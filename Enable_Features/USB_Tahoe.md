# Fixing `USBMap.Kext` for use with macOS 26 "Tahoe"

Apple introduced some changes to the USB stack in macOS 26 “Tahoe”, causing the standard `USBMap.kext` to stop functioning as expected. This guide explains how to adapt an existing `USBMap.kext` to work reliably under macOS Tahoe without needing to map ports via an SSDT.

## Option 1: Automated fix using USBMap Script

CorpNewt recently updated his [**USBMap**](https://github.com/corpnewt/USBMap) port mapping Python script. It also contains a script called `USBMapInjectorEdit`. It can now update the info.plist of the kext for macOS 26 compatibility automatically:

- Run `USBMapInjectorEdit` script (.command in macOS or .bat in Windows)
- Drag the USBMap.kext into the Terminal window and press Enter 
- In the Main windwow you now have the following options: <br>![](/Users/5t33z0/Desktop/Update_keys.png)
- You only have to press "U" and the relevant keys will be changed.
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
- Open it with ProperTree or the plist Editor of your choice
- In the plist, navigate to: `IOKitPersonalities` &rarr; XHC/EHC/etc. (Name depends on type of controller) &rarr; `IOProviderMergeProperties` &rarr; `ports` 
- There you will find the entries for all the ports you mapped (`HS` and `SS`)
- Open each one of them and every instance of `port` to `usb-port-number`. This is critical due to Apple's updated USB architecture in macOS 26.
- In the config, change `MinKernel` to `19.0.0`
- Save changes.

### Screenshots

Listed below are screenshots of the `info.plist` in the USBMap.kext. On the left is the original file and on the right the modified plist with the changes required for macOS Tahoe for direct comparison:

Previous macOS | macOS Tahoe
---------------|--------------
![old](https://github.com/user-attachments/assets/dcea4dc7-37bb-4fa0-acff-474710ea96a7) | ![new](https://github.com/user-attachments/assets/d89219c1-2ed5-4989-b211-ed173b1b12ca)

## Credits
- **JustFun** from for his original instructions [fix](https://www.hackintosh-forum.de/forum/thread/60350-wwdc-2025-macos-26-hackintosh/?postID=802582#post802582)
- **CorpNewt** for [USBMap](https://github.com/corpnewt/USBMap)
