[![OpenCore Version](https://img.shields.io/badge/OpenCore_Version:-0.9.4+-success.svg)](https://github.com/acidanthera/OpenCorePkg) ![macOS](https://img.shields.io/badge/Supported_macOS:-≤15.x-white.svg)

# Installing macOS 13+ on Haswell/Broadwell systems

:construction: **OVERHAUL in PROGRESS**

## EFI and Config Adjustments

You can use his [config.plist](/Haswell-Broadwell_OCLP_Wintel_Patches.plist) as reference. It contains all the settings below. You can cross-reference or copy entries directly from it.

<details> <summary><b>Booter Section (click to expand)</b></summary><br>
	
**Purpose**: Skips macOS board-id verification, allowing you to:

- Boot macOS with the native SMBIOS best suited for your CPU (instead of spoofed models)
- Receive system updates on unsupported hardware (when combined with RestrictEvents kext)

**Instructions**:

1. Open your config.plist in ProperTree
2. Copy the following code (click the copy button):
	```xml
	<dict>
		<key>Arch</key>
		<string>x86_64</string>
		<key>Comment</key>
		<string>Skip Board ID check</string>
		<key>Count</key>
		<integer>0</integer>
		<key>Enabled</key>
		<true/>
		<key>Find</key>
		<data>AFAAbABhAHQAZgBvAHIAbQBTAHUAcABwAG8AcgB0AC4AcABsAGkAcwB0</data>
		<key>Identifier</key>
		<string>Apple</string>
		<key>Limit</key>
		<integer>0</integer>
		<key>Mask</key>
		<data></data>
		<key>Replace</key>
		<data>AC4ALgAuAC4ALgAuAC4ALgAuAC4ALgAuAC4ALgAuAC4ALgAuAC4ALgAu</data>
		<key>ReplaceMask</key>
		<data></data>
		<key>Skip</key>
		<integer>0</integer>
	</dict>
	```
3. Paste it in the `Booter/Patch` Section (CMD+V):<br> <img width="1146" height="508" alt="Board-ID-Skip" src="https://github.com/user-attachments/assets/751068e7-0d67-4ff3-a6ce-46a2a596a734" />

Gif-Animation:<br> ![Booter_patch](https://github.com/user-attachments/assets/e00333d1-cc31-4f50-ada2-4d48c73d0eeb)

</details>

<details> <summary><b>DeviceProperties (click to expand)</b></summary><br>

**Location**: `DeviceProperties/Add/PciRoot(0x0)/Pci(0x2,0x0)`

**Supported iGPUs**: Intel HD 4200/4400/4600, HD 5000/5100/5200/5600, Iris Pro 6200

ℹ️ This section ensures you're using the correct `AAPL,ig-platform-id`. If using iGPU for display output, you may need additional framebuffer properties - consult the OpenCore Install Guide for complete patches.

<details>
<summary><b>Desktop - Haswell Systems (click to expand)</b></summary>

#### Haswell with dedicated GPU (Headless configuration)
**When to use**: iMac SMBIOS + Haswell CPU + iGPU enabled + dedicated GPU handles display

```
AAPL,ig-platform-id: 04001204
device-id: 12040000
```
**Note**: `device-id` only required for HD 4400

#### Haswell iGPU for display
**When to use**: Desktop PC where the iGPU directly drives your monitor

```
AAPL,ig-platform-id: 0300220D
device-id: 12040000
```
**Note**: `device-id` only required for HD 4400

</details>

<details>
<summary><b>Desktop - Broadwell Systems (click to expand)</b></summary>

#### Broadwell iGPU for display
**When to use**: Desktop PC with Broadwell CPU where iGPU drives your monitor

```
AAPL,ig-platform-id: 07002216
device-id: 12040000
```
**Note**: `device-id` only required for HD 4400

</details>

<details>
<summary><b>Laptop/NUC Systems (click to expand)</b></summary>

Laptop and NUC configurations require specific framebuffer patches depending on your exact iGPU model. Refer to the appropriate guide:

- **Haswell Laptops/NUCs**: [OpenCore Install Guide - Haswell](https://dortania.github.io/OpenCore-Install-Guide/config-laptop.plist/haswell.html#add-2)
- **Broadwell Laptops/NUCs**: [OpenCore Install Guide - Broadwell](https://dortania.github.io/OpenCore-Install-Guide/config-laptop.plist/broadwell.html#add-2)

Different combinations of `AAPL,ig-platform-id` and `device-id` may be required based on your specific hardware.

</details>
</details>

<details>
<summary><b>Kernel/Add Section (click to expand)</b></summary>

**Location**: `Kernel/Add` in config.plist and `EFI/OC/Kexts` folder

#### Required Kexts (All Users)

Add these kexts to both your config and EFI folder:

| Kext | MinKernel | Purpose |
|------|-----------|---------|
| [**AutoPkgInstaller**](https://github.com/dortania/OpenCore-Legacy-Patcher/tree/main/payloads/Kexts/Acidanthera) | - | Applies root patches automatically during macOS installation (requires [prepared installer](/Guides/Auto-Patching.md)) |
| [**AMFIPass**](https://github.com/dortania/OpenCore-Legacy-Patcher/tree/main/payloads/Kexts/Acidanthera) | `21.0.0` | Allows booting macOS 12+ without fully disabling AMFI |
| [**RestrictEvents**](https://github.com/acidanthera/RestrictEvents) | `20.4.0` | Enables OTA updates with VMM spoofing |

#### Optional Kexts

<details>
<summary><b>FeatureUnlock - Unlock macOS features (click to expand)</b></summary>

[**FeatureUnlock**](https://github.com/acidanthera/FeatureUnlock) unlocks macOS features normally restricted to newer Mac models:
- Sidecar
- AirPlay to Mac
- Universal Control

**Note**: Available features depend on your chosen SMBIOS. Check the [FeatureUnlock documentation](https://github.com/acidanthera/FeatureUnlock) for compatibility. Use the `-disable_sidecar_mac` boot-arg to disable these patches if needed.

</details>

<details>
<summary><b>WiFi Support for macOS Sonoma (click to expand)</b></summary>

**Only needed if you have a Broadcom or Atheros WiFi card** and are running macOS Sonoma.

### Supported WiFi Cards

**Modern Broadcom**: BCM94350, BCM94360, BCM43602, BCM94331, BCM943224  
**Legacy**: Atheros chipsets, Broadcom BCM94322, BCM94328

#### Required Kexts

Add both kexts with `MinKernel: 23.0.0`:

1. [**IOSkywalk.kext**](https://github.com/dortania/OpenCore-Legacy-Patcher/blob/e21efa975c0cf228cb36e81a974bc6b4c27c7807/payloads/Kexts/Wifi/IOSkywalkFamily-v1.0.0.zip)
2. [**IO80211FamilyLegacy.kext**](https://github.com/dortania/OpenCore-Legacy-Patcher/blob/e21efa975c0cf228cb36e81a974bc6b4c27c7807/payloads/Kexts/Wifi/IO80211FamilyLegacy-v1.0.0.zip)
   - **Important**: This contains `AirPortBrcmNIC.kext` as a plugin - ensure it's also enabled in your config

#### Additional Required Step

You must also block macOS's native IOSkywalk kext. Add to `Kernel/Block`:

```
Identifier: com.apple.iokit.IOSkywalkFamily
Strategy: Exclude
MinKernel: 23.0.0
Enabled: true
```

**Reference**: [Complete WiFi Patching Guide](/Enable_Features/WiFi_Sonoma.md)

</details>

#### Kexts to Disable

If present in your config, **disable** these kexts:
- **CPUFriend**
- **CPUFriendDataProvider**

ℹ️ When changing SMBIOS, it's recommended to regenerate a new CPUFriendDataProvider.kext for optimal power management.

---

### 4. Kernel - CPU Emulation

**⚠️ HEDT WORKSTATIONS ONLY** 

**Skip this entire section** unless you have a High-End Desktop Workstation with Haswell-E or Broadwell-E CPU. Do NOT add these settings to regular Desktop, Laptop, or NUC configs.

<details>
<summary><b>Haswell-E / Broadwell-E Users (click to expand)</b></summary>

**Location**: `Kernel/Emulate`

#### Haswell-E CPUs
```
Cpuid1Data: C3060300 00000000 00000000 00000000
Cpuid1Mask: FFFFFFFF 00000000 00000000 00000000
```

#### Broadwell-E CPUs
```
Cpuid1Data: D4060300 00000000 00000000 00000000
Cpuid1Mask: FFFFFFFF 00000000 00000000 00000000
```

</details>

---

### 5. Kernel - Patches

**Location**: `Kernel/Patch`

**How to add**: Copy complete patch entries from [OCLP's config.plist](https://github.com/dortania/OpenCore-Legacy-Patcher/blob/main/payloads/Config/config.plist) including all fields (Comment, Enabled, MinKernel, MaxKernel, Find, Replace, etc.)

#### Required Patches (All Users)

**Add and enable**:
- "Disable Library Validation Enforcement"
- "Disable _csr_check() in _vnode_check_signature"

ℹ️ The second patch may not be needed on all systems. Try without it first - if you encounter issues, add it back.

ℹ️ VMM board-id kernel patches are no longer required - RestrictEvents handles this via NVRAM.

<details>
<summary><b>Conditional Patches (click to expand)</b></summary>

#### Force FileVault on Broken Seal
**Only needed if**: You're using FileVault encryption

### Fix PCI bus enumeration (Ventura)
### Fix PCI bus enumeration (Sonoma)

**Only needed if**: Internal PCIe devices (like WiFi/Bluetooth cards) show up as removable express cards in the menu bar:

![PCIe express card issue](https://github.com/user-attachments/assets/d362d81c-01f7-491e-98c9-cd9372f30eb1)

Add the patch that matches your macOS version.

</details>
</details>

### 6. Misc - Security Settings

**Location**: `Misc/Security`

```
SecureBootModel: Disabled
Vault: Optional
```

**Why**: Required when patching in graphics drivers for AMD and NVIDIA GPUs. 

ℹ️ Intel iGPU-only users may be able to use `SecureBootModel: Default` - try for yourself if you want to maintain some security features.

---

<details>
<summary><b>NVRAM Settings</b></summary>

**Location**: `NVRAM/Add/7C436110-AB2A-4BBB-A880-FE41995C9F82`

---

**System Integrity Protection**

Modify existing `csr-active-config` key:

- **Standard setup**: `03080000`  
- **If using NVIDIA GPU**: `030A0000`

---

**Boot Arguments – Required for All Users**

Add the following boot-args:

```
-amfipassbeta
```
```
ipc_control_port_options=0

```

**Notes**

- Required for applying root patches with OCLP.  
  Thanks to `AMFIPass.kext`, this is no longer required for booting – only for patching.  
  Disabling AMFI may cause [camera/microphone permission issues in third-party apps](https://github.com/5T33Z0/OC-Little-Translated/blob/main/13_Peripherals/Fixing_Webcams.md).

- `ipc_control_port_options=0`  
  Required for Intel HD Graphics. Fixes Firefox and Electron-based app issues (e.g. Discord).

---

<details>
<summary><b>AMD GPU Users – Additional boot-args</b></summary>

**Temporary boot-arg (installation only)**

```

-radvesa

```

Purpose: Disables hardware acceleration and forces VESA mode.  
Use only if the screen turns black after installing macOS 12+.

⚠️ After installing GPU drivers with OCLP, this boot-arg **must be removed** or graphics acceleration will not work.

---

**AMD GPU with headless SMBIOS**

Use only if:

- SMBIOS: `iMacPro1,1` or `MacPro7,1`  
- GPU: Polaris or Vega (Navi is not supported by OCLP)

```

-wegnoigpu

```

Purpose: Disables iGPU in macOS and assigns background rendering to the AMD GPU.

Optional: Add `unfairgva=` bitmask (1–7) to fix DRM issues:  
https://github.com/5T33Z0/OC-Little-Translated/tree/main/H_Boot-args#unfairgva-overrides

</details>

---

<details>
<summary><b>NVIDIA GPU Users – Additional boot-args</b></summary>

**Kepler GPUs (GTX 600 / 700)**

Kepler GPUs work out of the box on Monterey through Ventura.  
No additional boot-args required.

---

**Non-Kepler NVIDIA GPUs**

Temporary boot-arg (installation only):

```

nv_disable=1

```

Purpose: Disables acceleration during installation.  
Remove after installing NVIDIA drivers with OCLP.

---

Required boot-args:

```

ngfxcompat=1
ngfxgl=1
nvda_drv_vrl=1

```

- `ngfxcompat=1` → Bypass NVDAStartupWeb compatibility check  
- `ngfxgl=1` → Disable Metal, use OpenGL  
- `nvda_drv_vrl=1` → Enable NVIDIA Web Drivers

---

If black screen occurs after driver installation:

```

agdpmod=vit9696

```

Purpose: Disables board-ID check.

</details>

---

<details>
<summary><b>FeatureUnlock Users</b></summary>

```

-disable_sidecar_mac

```

Purpose: Disables Sidecar / AirPlay / Universal Control patches.

</details>

---

### OCLP and RestrictEvents NVRAM Keys

**Location**  
`NVRAM/Add/4D1FDA02-38C7-4BCCA8B30102`

Add the following keys:

| Key | Type | Value |
|----|----|----|
| OCLP-Settings | String | -allow_amfi |
| revblock | String | media |
| revpatch | String | sbvmm,asset |

Reference: https://github.com/acidanthera/RestrictEvents

---

### NVRAM Delete Entries

**Location**  
`NVRAM/Delete/4D1FDA02-38C7-4BCCA8B30102`

Add the following entries:

| Key |
|-----|
| OCLP-Settings |
| revblock |
| revpatch |

**Purpose**: Clears old values before new ones are written, avoiding manual NVRAM resets.
</details>

### 8. UEFI - Drivers

**Location**: `UEFI/Drivers` in config.plist and `EFI/OC/Drivers` folder

**Add**:
- `ResetNvramEntry.efi`

**How to add in config**:
```
Path: ResetNvramEntry.efi
Enabled: true
```

**Purpose**: Adds a boot menu entry to perform NVRAM reset without changing the boot drive order.

**Requirement**: Your system must have a UEFI BIOS (not legacy BIOS).

---

## ✓ Verification Checklist

After making all changes:

- [ ] **Validate config**: Run `ocvalidate` or use OCAuxiliaryTools/ProperTree to check for errors
- [ ] **Verify MinKernel settings**: Ensure kexts like AMFIPass and RestrictEvents have correct `MinKernel` values
- [ ] **Check kext plugins**: If using IO80211FamilyLegacy, ensure `AirPortBrcmNIC.kext` is enabled
- [ ] **Backup working EFI**: Make a copy of your current working EFI before rebooting with changes
- [ ] **Review GPU boot-args**: Make sure you only added boot-args relevant to YOUR GPU (not all of them)
- [ ] **HEDT check**: If you're NOT using Haswell-E/Broadwell-E, ensure `Kernel/Emulate` is empty

**If components don't work after patching**: Consult [OCLP's patch documentation](https://github.com/dortania/OpenCore-Legacy-Patcher/blob/main/docs/PATCHEXPLAIN.md) for additional settings or troubleshooting steps.
