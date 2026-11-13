# Re-Enabling Atheros Wi-Fi in macOS Sonoma and later

## Overview

During the early stages of macOS Sonoma development, Apple removed several kexts and frameworks required by older wireless cards. This also affected legacy Atheros Wi-Fi cards, which were never used in Apple Macs.

OpenCore Legacy Patcher (OCLP) can restore support by injecting the required legacy wireless kexts through OpenCore and applying post-install root patches.

## Supported chipsets

The following Atheros chipsets are covered by this guide:

- `AR928X`
- `AR93xx`
- `AR242x` / `AR542x`
- `AR5418`
- `AR5416`

## Required Kexts

**Legacy Broadcom and Atheros cards:**

Add the following kexts from the [OCLP Wi-Fi payloads](https://github.com/dortania/OpenCore-Legacy-Patcher/tree/main/payloads/Kexts/Wifi) to `EFI/OC/Kexts`:

- `corecaptureElCap.kext`
- `IO80211ElCap.kext` (&rarr; contains `AirPortAtheros40.kext` as a plugin. Enable it for Atheros cards. ⚠️ Currently only working up to macOS Sequoia.
- `AirPortAtheros40-Tahoe.kext` (for macOS Tahoe)
- `AMFIPass.kext` (if it is not already present)

**Configuration**:

![wifi_legacy](https://github.com/5T33Z0/OC-Little-Translated/assets/76865553/ab112e90-a528-4354-a5a7-729900ebacf6)

### Config adjustments

- **Kernel → Block:** Block `com.apple.iokit.IOSkywalkFamily` for modern Broadcom cards:<br>![Brcm_Sonoma1](https://github.com/5T33Z0/OC-Little-Translated/assets/76865553/54079541-ee2e-4848-bb80-9ba062363210)
- **Kernel → Quirks:** Enable `DisableIoMapper`. Once Wi-Fi works, test whether the quirk can be disabled on your system.
- **Misc → Security:** Temporarily set `SecureBootModel` to `Disabled` if necessary to troubleshoot kernel panics.
- **NVRAM → Add → `7C436110-AB2A-4BBB-A880-FE41995C9F82`:** 
	- Set `csr-active-config` to `03080000`.
	- Add `-amfipassbeta` to `boot-args` if `AMFIPass.kext` fails to load.

> [!NOTE]
> 
> If you use an ECM-based USB Ethernet adapter, also add [ECM-Override.kext](https://github.com/dortania/OpenCore-Legacy-Patcher/blob/main/payloads/Kexts/Misc/ECM-Override-v1.0.0.zip) to prevent potential kernel panics caused by blocking `IOSkywalkFamily`.

> [!TIP]
>
> On my real iMac12,2, Atheros WiFi was not working in macOS Tahoe after patching with OCLP 3.0.0 RC 3. I didn't invest the issue further but maybe this helps: [Restore Wi-Fi for Atheros on macOS Monterey to Tahoe](https://github.com/samuelnotfound/miscellaneous-hackintosh-guides/tree/main/Atheros_Wifi_Monterey_and_newer) 

### Verify kext loading

Save the configuration, reboot, reset NVRAM and boot macOS. In Terminal, inspect the loaded kexts using:

```bash
kextstat | grep -v com.apple
```

Verify that the required kexts are loaded before applying root patches.

## Applying OCLP root patches

1. Download and install the [latest OpenCore Legacy Patcher release](https://github.com/dortania/OpenCore-Legacy-Patcher/releases).
2. Run OCLP and select **Post-Install Root Patch**.
3. If **Networking: Legacy Wireless** is available, select **Start Root Patching**.
4. Reboot once patching is complete.

If the expected patch option is unavailable, check the OCLP troubleshooting documentation.

> [!IMPORTANT]
>
> Root patches break the macOS sealed system volume. System updates may consequently require downloading the full installer. Do not revert root patches if other components, such as graphics acceleration, depend on them.

## Notes

- Make sure to perform an NVRAM reset after applying these settings and after switching macOS versions if you have multiple macOS installations.
- Bluetooth functionality is not covered by this guide. Bluetooth support depends on the separate Bluetooth controller and its corresponding kexts.

## References

- [OpenCore Legacy Patcher](https://github.com/dortania/OpenCore-Legacy-Patcher)
- [OCLP Wi-Fi kexts](https://github.com/dortania/OpenCore-Legacy-Patcher/tree/main/payloads/Kexts/Wifi)
- [Wi-Fi patching discussion on InsanelyMac](https://www.insanelymac.com/forum/topic/357087-macos-sonoma-wireless-issues-discussion/?do=findComment&comment=2809940)

## Credits

- Acidanthera for OpenCore and its kexts
- Dortania for OpenCore Legacy Patcher
- Acquarius13 for investigating the required OCLP source changes
