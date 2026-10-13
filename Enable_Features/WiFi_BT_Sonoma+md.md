# Restoring Wi-Fi and Bluetooth Support in macOS Sonoma+

**TABLE of CONTENTS**

- [About](#about)
- [Identify Your Wi-Fi Chipset](#identify-your-wi-fi-chipset)
- [Broadcom and Atheros Wi-Fi](#broadcom-and-atheros-wi-fi)
  - [Required Kexts](#required-kexts)
  - [Config adjustments](#config-adjustments)
  - [Verify kext loading](#verify-kext-loading)
  - [Apply OCLP root patches](#apply-oclp-root-patches)
- [Intel Wi-Fi](#intel-wi-fi)
  - [Configure Intel Wi-Fi (macOS High Sierra to Sonoma)](#configure-intel-wi-fi-macos-high-sierra-to-sonoma)
  - [Configure Intel Wi-Fi (macOS Sequoia and Tahoe)](#configure-intel-wi-fi-macos-sequoia-and-tahoe)
- [Bluetooth](#bluetooth)
  - [Broadcom Bluetooth](#broadcom-bluetooth)
  - [Intel Bluetooth](#intel-bluetooth)
  - [Troubleshooting Bluetooth](#troubleshooting-bluetooth)
- [References](#references)

---

## About

macOS Sonoma and newer removed or changed components required by some previously supported Wi-Fi chipsets. This guide covers restoring Broadcom and Atheros Wi-Fi support with OpenCore Legacy Patcher (OCLP), configuring Intel Wi-Fi with OpenIntelWireless, and enabling Bluetooth.

## Identify Your Wi-Fi Chipset

| Chipset | Recommended approach |
|---|---|
| Modern Broadcom (`BCM94350`, `BCM94360`, `BCM43602`, etc.) | Inject required kexts and apply OCLP root patches |
| Legacy Broadcom (`BCM94322`, `BCM94328`) | Inject legacy Wi-Fi kexts and apply OCLP root patches |
| Atheros (`AR928X`, `AR93xx`, etc.) | Inject legacy Wi-Fi kexts and apply OCLP root patches |
| Intel on High Sierra to Sonoma | Use `itlwm.kext` or `AirportItlwm.kext` |
| Intel on Sequoia/Tahoe | Use `itlwm.kext` or the [laobamac/itlwm fork](https://github.com/laobamac/itlwm) for native `AirportItlwm.kext` support without Wi-Fi-related root patches |

**Exceptions:** `BCM43602` and `BCM4350` cards can use [AppleBCMWLANCompanion](https://github.com/0xFireWolf/AppleBCMWLANCompanion) without root patches. Follow its [configuration guide](https://github.com/0xFireWolf/AppleBCMWLANCompanion/blob/main/Documentation/GettingStarted.md).

> [!CAUTION]
>
> Before making changes, back up your working EFI folder on a FAT32 formatted USB Flash Drive

---

## Broadcom and Atheros Wi-Fi

### Required Kexts

Add the required kexts to `EFI/OC/Kexts` and configure them under `Kernel → Add` in `config.plist`.

**Modern Broadcom cards:**

- `IOSkywalkFamily.kext`
- `IO80211FamilyLegacy.kext` and its `AirPortBrcmNIC.kext` plugin
- `AirportBrcmFixup.kext` and its `AirPortBrcmNIC_Injector.kext` plugin
- `AMFIPass.kext`, if not already present

Download the Wi-Fi kexts from the [OCLP repository](https://github.com/dortania/OpenCore-Legacy-Patcher/tree/main/payloads/Kexts/Wifi) and `AirportBrcmFixup.kext` from the [Acidanthera builds page](https://dortania.github.io/builds/?product=AirportBrcmFixup&viewall=true). Get AMFIPass from [here](https://github.com/bluppus20/AMFIPass/releases)

**Configuration**:

![wifi_modern](https://github.com/5T33Z0/OC-Little-Translated/assets/76865553/ca04bf6f-71ee-47ed-b7b4-15359b88c17e)

**Legacy Broadcom and Atheros cards:**

- `corecaptureElCap.kext`
- `IO80211ElCap.kext`
- `AirPortAtheros40.kext` for Atheros cards
- `AMFIPass.kext`, if not already present

Download the Wi-Fi kexts from the [OCLP repository](https://github.com/dortania/OpenCore-Legacy-Patcher/tree/main/payloads/Kexts/Wifi) and AMFIPass from [here](https://github.com/bluppus20/AMFIPass/releases)

Configure the `MinKernel` and `MaxKernel` values according to the macOS versions supported by each kext. 

**Configuration**:

![wifi_legacy](https://github.com/5T33Z0/OC-Little-Translated/assets/76865553/ab112e90-a528-4354-a5a7-729900ebacf6)

### Config adjustments

- **Kernel → Block:** Block `com.apple.iokit.IOSkywalkFamily` for modern Broadcom cards:<br>![Brcm_Sonoma1](https://github.com/5T33Z0/OC-Little-Translated/assets/76865553/54079541-ee2e-4848-bb80-9ba062363210)
- **Kernel → Quirks:** Enable `DisableIoMapper`. Once Wi-Fi works, test whether the quirk can be disabled on your system.
- **Misc → Security:** Temporarily set `SecureBootModel` to `Disabled` if necessary to troubleshoot kernel panics.
- **NVRAM → Add → `7C436110-AB2A-4BBB-A880-FE41995C9F82`:** 
	- Set `csr-active-config` to `03080000`.
	- Add `-amfipassbeta` to `boot-args` if `AMFIPass.kext` fails to load.
	- Add `-brcmfxbeta` if Broadcom Wi-Fi is available but cannot connect to access points.

If you use an ECM-based USB Ethernet adapter, also add [ECM-Override.kext](https://github.com/dortania/OpenCore-Legacy-Patcher/blob/main/payloads/Kexts/Misc/ECM-Override-v1.0.0.zip) to prevent potential kernel panics caused by blocking `IOSkywalkFamily`.

### Verify kext loading

Save the configuration, reboot, reset NVRAM and boot macOS. In Terminal, inspect the loaded kexts using:

```bash
kextstat | grep -v com.apple
```

Verify that the required kexts are loaded before applying root patches.

### Apply OCLP root patches

1. Download and install the latest [OpenCore Legacy Patcher release](https://github.com/dortania/OpenCore-Legacy-Patcher/releases).
2. Open OCLP and select **Post-Install Root Patch**. This shows the available root patches. 
3. Click on "Start Root Patching"
4. Once completet, reboot the system.

> [!IMPORTANT]
>
> Root patches break the macOS sealed system volume. System updates may consequently require downloading the full installer. Do not revert root patches if other components, such as graphics acceleration, depend on them.

---

## Intel Wi-Fi

Intel Wi-Fi is supported by the [OpenIntelWireless project](https://openintelwireless.github.io/General/Installation.html). Two drivers are available:

- **`itlwm.kext`**: Exposes Wi-Fi as a LAN interface and requires [HeliPort](https://github.com/perez987/HeliPort) to connect to access points.
- **`AirportItlwm.kext`**: Integrates with macOS natively and does not require HeliPort. Each build is specific to a macOS version.

> [!IMPORTANT]
>
> Use either `AirportItlwm.kext` or `Itlwm.kext` – never both!

### Configure Intel Wi-Fi (macOS High Sierra to Sonoma)

1. Download the required kexts from the [itlwm releases](https://github.com/OpenIntelWireless/itlwm/releases).
2. Extract the archive and copy the desired kexts to `EFI/OC/Kexts`.
3. Add each kext under `Kernel → Add` in `config.plist`.
4. Configure `MinKernel` and `MaxKernel` as required:
   - For `itlwm.kext`, set the kernel restrictions to cover the macOS versions you want to use it with.
   - For `AirportItlwm.kext`, restrict each build to its corresponding macOS version.
5. Save `config.plist` and reboot.

> [!TIP]
>
> If you want to include AirportItlwm variants for multiple macOS versions in your OpenCore EFI, rename the kexts accordingly (e.g., `AirportItlwm_Sequoia` or `AirportItlwm_Tahoe`), then add them to your `EFI/OC/Kexts` folder and `config.plist`. Finally, adjust the `MinKernel` and `MaxKernel` settings so that only the appropriate kext is loaded for each macOS version:
>
> <img width="1489" height="124" alt="AirportItlwm Settings" src="https://github.com/user-attachments/assets/a5cf38f2-2072-479d-b41d-fe6a375917c2" />
> Renamed kexts may no longer be detected automatically by tools such as OCAT. Manage their updates manually.

### Configure Intel Wi-Fi (macOS Sequoia and Tahoe)

Conventional `AirportItlwm.kext` builds do not work normally in macOS Sequoia and Tahoe because of changes to Apple's Wi-Fi frameworks. The [laobamac/itlwm fork](https://github.com/laobamac/itlwm) provides compatible builds without requiring Wi-Fi-related OCLP root patches.

1. Back up your working EFI folder to a FAT32-formatted USB drive.
2. Download the appropriate Sequoia or Tahoe build from the [fork's releases](https://github.com/laobamac/itlwm/releases).
3. Remove or disable `itlwm.kext` and any components left over from the legacy workaround:
   - `IOSkywalk.kext`
   - `IO80211FamilyLegacy.kext`
   - The `com.apple.iokit.IOSkywalkFamily` entry under `Kernel → Block`
4. Add the fork's `AirportItlwm.kext` to `EFI/OC/Kexts` and enable it under `Kernel → Add`.
5. Enable `DisableIoMapper` if required, particularly on Tahoe. If you need AppleVTD, test with this quirk disabled, provided VT-d is enabled and supported by your system.
6. Save the configuration and reboot.

If OCLP root patches were installed solely for the legacy Intel Wi-Fi workaround, revert them using OCLP-Mod's **Post-Install Root Patch → Revert Root Patches** and reboot. Do not revert patches required by other hardware.

If switching from `itlwm.kext` to `AirportItlwm.kext`, remove HeliPort from automatic startup if it is no longer needed.

---

## Bluetooth

Bluetooth support is separate from Wi-Fi. Correct USB port mapping and the appropriate Bluetooth kexts are essential.

### Broadcom Bluetooth

For macOS Monterey and newer, add `BlueToolFixup.kext` from [BrcmPatchRAM](https://github.com/acidanthera/BrcmPatchRAM) and set `MinKernel` to `21.0.0`. Adjust the kernel restrictions of existing Bluetooth kexts according to the macOS versions you support.

**Kext Order amd Min/Max Kernel Settings**:

![brcmbeetee](https://github.com/user-attachments/assets/6f3a6349-ade7-41a3-b7b1-0370401b5279)


**NVRAM Settings**

When using BrcmPatchRAM 2.7.0 or newer, add these entries under  `NVRAM → Add → 7C436110-AB2A-4BBB-A880-FE41995C9F82`:

| Key | Type | Value |
|---|---|---|
| `bluetoothInternalControllerInfo` | Data | `00000000 00000000 00000000 0000` |
| `bluetoothExternalDongleFailed` | Data | `00` |

**Screenshot**:

![](https://www.insanelymac.com/uploads/monthly_2024_12/Bildschirmfoto2024-12-15um14_18_08.png.d2057e458a2be3cefff9afe2ae40688f.png)

If Broadcom Bluetooth still fails on macOS 14 or newer, add `-btlfxboardid` to `boot-args`.

### Intel Bluetooth

If your Intel Wi-FI/BT card is [supported](https://openintelwireless.github.io/itlwm/Compat.html) by the OpenIntelWireless project, the following kexts are required for enabling Bluetooth in macOS 14+ (&rarr; [plist](/plist/Sonoma_WIFI_Intel.plist)):

Kext | Comment | `MinKernel` | `MaxKernel`
-----|---------|:-----------:|:-----------:
[**`BlueToolFixup.kext`**](https://github.com/acidanthera/BrcmPatchRAM) | BT enabler for macOS 12 and newer | 21.0.0 | 
[**`IntelBluetoothFirmware.kext`**](https://github.com/OpenIntelWireless/IntelBluetoothFirmware) | Intel BT FIrmware |  | 

**Kext Order and Min/Max Kernel Settings**:

![intelbt](https://github.com/user-attachments/assets/96062780-6a52-430c-a1ac-4b45515c81a6)

> [!NOTE]
> 
> - Make sure to perform an NVRAM reset after applying these settings and after switching macOS versions (if you have different versions of macOS installed)
> - I have noticed that it can take like 10 seconds or so until BT is available after applying these changes.
> - For using Intel Bluetooth in macOS Monterey and newer, [**read this**](https://openintelwireless.github.io/IntelBluetoothFirmware/FAQ.html#what-additional-steps-should-i-do-to-make-bluetooth-work-on-macos-monterey-and-newer).

### Troubleshooting Bluetooth

If Bluetooth does not work, inspect NVRAM in Terminal:

```bash
nvram -p | grep bluetoothExternalDongleFailed
```

If the value is `01`, try resetting it to `00` and restarting the Bluetooth daemon:

```bash
sudo nvram bluetoothExternalDongleFailed=%00
sudo pkill bluetoothd
```

> [!TIP]
>
> In cases where `bluetoothExternalDongleFailed` changes from `00` to `01` after rebooting, you can use this [plist](https://gist.github.com/LBates2000/32be3c79dd987e72b98bb669c43fccbf) that acts as a LaunchDaemon to reset the value back to `00`. 

---

## References

- [OpenCore Legacy Patcher](https://github.com/dortania/OpenCore-Legacy-Patcher)
- [OpenIntelWireless documentation](https://openintelwireless.github.io/General/Installation.html)
- [laobamac/itlwm](https://github.com/laobamac/itlwm)
- [BrcmPatchRAM](https://github.com/acidanthera/BrcmPatchRAM)
- [Wi-Fi Patcher Pro](https://github.com/Mirone/WiFiPatcherPro) — alternative automated setup for supported configurations.
