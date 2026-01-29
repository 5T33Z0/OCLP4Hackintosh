# Using `AirportItlwm.kext` for Intel WiFi/BT Cards in macOS Sequoia and Tahoe

![intel_spoof07](https://github.com/user-attachments/assets/255406e1-554e-4b6c-9b67-5d24b9fcb962)

## About

As you may know, Apple removed support for various system kexts and frameworks required by Broadcom WiFi cards from macOS Sequoia. Although Apple never shipped any systems with Intel WiFi/BT cards, `AirportItlwm.kext` also requires these system kexts and frameworks to function properly. So in macOS Sequoia, users with Intel WiFi/BT cards had to resort to the `Itlwm.kext` instead. 

Since `Itlwm.kext` injects the WiFi card as LAN adapter into macOS, this has some side-effects. For example, the Airport-Utility which lets you connect to WiFi hotspot can no longer be used – a separate app ([**Heliport**](https://github.com/OpenIntelWireless/HeliPort/releases)) has to be used to join WiFi APs. Another issue is that FindMyMac also requires WiFi.

Luckily for us, we can utilize [**OCLP-Mod**](https://github.com/laobamac/OCLP-Mod/releases) to make use of `AirportItlwm` in macOS Sequoia and Tahoe again!

## Patching principle

1. Block Original `com.apple.iokit.IOSkywalkFamily`
2. Inject the required kexts for re-enabling legacy WiFi cards
3. Add `AirportItlwm.kext` from macOS Ventura 
3. Apply root patches with OCLP-Mod
4. Reboot to macOS Sequoia/Tahoe and voilà: `AirportItlwm.kext` is working again 

> [!NOTE]
> 
> When applying root patches for Modern WiFi, OCLP basically rolles back components from macOS Ventura. That's why the AirportItlwm kext for macOS Ventura is required to make WiFi work in Sequoia.

## Instructions

We need to prepare the `config.plist` and EFI folder content to make `AirportItlwm.kext` work in macOS Sequoia and Tahoe. You can either follow the instructions below, or [copy the settings from this plist](/plist/AirportItlwm_Sequoia.plist).

⚠️ Make sure to adjust the PCI path of the WiFi card so that it matches the location of the WiFi card in your system!

### 1. Disable/Delete `IOName` spoof

If present, disable (#) or delete `IOName` spoof from your config's `DeviceProperties` to prevent OCLP-Mod to apply root patches for Broadcom Cards:

<img width="894" height="123" alt="01" src="https://github.com/user-attachments/assets/a75f8b8d-9b28-43d9-854c-375fb7498328" />

### 2. Block new `IOSkywalk` kext

Under `Kernel/Block`, add the following rule:

```xml
<dict>
	<key>Arch</key>
	<string>x86_64</string>
	<key>Comment</key>
	<string>Allow IOSkywalk Downgrade</string>
	<key>Enabled</key>
	<true/>
	<key>Identifier</key>
	<string>com.apple.iokit.IOSkywalkFamily</string>
	<key>MaxKernel</key>
	<string></string>
	<key>MinKernel</key>
	<string>24.0.0</string>
	<key>Strategy</key>
	<string>Exclude</string>
</dict>
```

**Screenshot**:<br> <img width="920" height="240" alt="03" src="https://github.com/user-attachments/assets/b9760a94-a449-45f1-8289-c542e1262e65" />

### 3. Add Kexts
- Disable `Itlwm.kext`, if present!
- Add the following [**Kexts from the OCLP repo**](https://github.com/dortania/OpenCore-Legacy-Patcher/tree/main/payloads/Kexts/Wifi) to `EFI/OC/Kexts` and your `config.plist`:
	- [`AMFIPasss.kext`](https://github.com/dortania/OpenCore-Legacy-Patcher/tree/main/payloads/Kexts/Acidanthera) 
	- `IOSkywalk.kext`
	- `IO8021FamilyLegacy.kext` (contains plugin `AirportBrcmNIC.kext` which you can disable since it is for Broadcom WiFi cards)
	- [**`AirportItlwm.kext`**](https://github.com/OpenIntelWireless/itlwm/releases) (inject the one for macOS Ventura! I have renamed it to `AirportItlwm_Sequoia.kext` since I also have macOS Sonoma installed and it requires a different variant of the kext). ⚠️ Make sure it is injected ***after*** `IOSkywalk` and `IO8021FamilyLegacy` kexts!
- Adjust `MinKernel` and `MaxKernel` settings as shown in the **Screenshot**: <br>![itlwfbt](https://github.com/user-attachments/assets/b3cb9e89-9d91-4eb7-87e3-6ff5516df386)

### 4. Enable Kernel Quirk
- Enable `DisableIoMapper`

> [!TIP]
>
> Once you've verified that WiFi is working, you can check if WiFi works without this quirk as well. But in my experience on the machines I have, Intel WiFi does not work without it.

### 5. Disable `SecureBootModel`

Under `Misc`, change `SecureBootModel` to `Disabled`

### 6. Add/Adjust NVRAM Entries
- Change `csr-active-config` to `03080000` (resp. `030A0000`, if you are using an NVIDIA GPU)
- Add `-amfipassbeta` boot-arg if WiFi/BT is not working in macOS Sequoia/Tahoe
- If Bluetooth stops working after root patching, add the following entries to the NVRAM section, so that Intel BlueTooth will work:

```xml
<key>7C436110-AB2A-4BBB-A880-FE41995C9F82</key>
<dict>
	<key>bluetoothExternalDongleFailed</key>
	<data>AA==</data>
	<key>bluetoothInternalControllerInfo</key>
	<data>AAAAAAAAAAAAAAAAAAA=</data>
	<key>boot-args</key>
	<string>-lilubetaall</string>
	<key>csr-active-config</key>
	<data>AwgAAA==</data>
</dict>
```
**Screenshot**:<br><img width="1139" height="395" alt="04" src="https://github.com/user-attachments/assets/d8c45364-066b-4f30-91ff-e7bd227ccf77" />

- Save your `config.plist`

### 7. Download OCLP-Mod
- Since you won't have internet Access in macOS Sequoia, [download the latest release of OCLP-Mod](https://github.com/laobamac/OCLP-Mod/releases) before rebooting into macOS
- Now reboot into macOS Sequoia

### 8. Apply root patches with OCLP

- Run OCLP-Mod
- Click the top right button:<br> <img width="600" height="331" alt="patch01" src="https://github.com/user-attachments/assets/19dc7610-829c-4bd5-9e99-a0938331b50e" />
- There should say "Intel" somewhere in the Chines text of the Patches that will be applied:<br><img width="603" height="332" alt="oclpmod_intel01" src="https://github.com/user-attachments/assets/f49ecc37-e644-4e67-851b-f54fce640935" />
- Click the first button from the top to start root patching.
- Additional files required for patching will be downloadded automatically:<br><img width="415" height="288" alt="oclpmod_intel02" src="https://github.com/user-attachments/assets/391d5440-d3f4-4629-9dbb-e13fe511cbdd" />
- Once that's done, patching will commence:<br> <img width="405" height="559" alt="oclpmodpatch" src="https://github.com/user-attachments/assets/21d95656-c186-4032-be13-665f09b01f6d" />

### 9. Reboot and enjoy!

- Reboot the system
- Perform an NVRAM reset
- Boot into macOS Sequoia/Tahoe
- You should now be able to use the Airport-Utility in macOS Sequoia again, to connect to WiFi APs

> [!IMPORTANT]
> 
> Once root patches are applied, the security seal of the volume will be broken. And once it is broken, the complete macOS version will be downloaded every time an OS update is available. The workaround would be to revert root patches before installing updates – but then you won't have WiFi (unless you enable `itlwm` beforehand).

### 10. Troubleshooting

On some systems excluding the IOSkywalkFamily kext may cause a Kernel panic. In this case the workaround is to add the following rule to the `Kernel/Force` section of your config.plist:

| Identifier | BundlePath | Enabled | ExecutablePath | MinKernel |
| ---------- | ---------- | :------: | -------------- | --------- | 
| com.apple.iokit.IOSkywalkFamily | System/Library/Extensions/IOSkywalkFamily.kext | true | Contents/MacOS/IOSkywalkFamily | 24.0.0 |

## Using this fix to enable iServices in macOS Sonoma

As it turns out, this fix is also required if you need working iServices in macOS Sonoma (Screenshot from OpCore Simplify):

![wifi](https://github.com/user-attachments/assets/af01d107-12b8-4441-b858-bc3720b2fe7a)

So, if you need iService in macOS Somona, do the following:
- [ ] Change `MinKenel` of the following Kexts back to `23.0.0`:
	- [ ] `AirportItlwm.kext` (use the version for macOS Ventura!)
	- [ ] `IOSkywalkFamily.kext` 
	- [ ] `IO80211FamilyLegacy.kext` 
	- [ ] `com.apple.iokit.IOSkywalkFamily` (under `Kernel/Block`) 
- [ ] Apply root patches in Post-Install! 

Once you apply root patches, incremental system updates are no longer an option – every time an updates ist available, the complete installer will be downloaded because root patching brakes the security seal of macOS. If you don't require iService you don't have to do this – WiFi will work in Sonoma without this patch.

## Credits and Thank Yous
- lifeknife10A who came up with this [workaround](https://github.com/OpenIntelWireless/itlwm/issues/1009#issuecomment-2370919270)
- sughero, for additional info about the order of the kexts
- stefanalmare for pointing me to this solution
- lzhoang2801 for OpCore Simplify and further explanations about the effect on iServices
