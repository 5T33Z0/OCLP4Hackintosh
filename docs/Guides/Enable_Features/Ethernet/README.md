# Re-Enable Legacy Broadcom Ethernet Controllers in macOS Big Sur to Tahoe

## Overview

This guide explains how to restore support for legacy Broadcom Ethernet controllers in macOS Big Sur (11) through macOS Tahoe (26) using `CatalinaBCM5701Ethernet.kext` and an OpenCore kernel patch.

The guide covers two categories of devices:

1. **Natively supported chipsets:** Work without binary kernel patches or `device-id` spoofing.
2. **Patched and spoofed chipsets:** Require both `device-id` spoofing and a binary kernel patch.

**Original guide:** [samuelnotfound – Broadcom Ethernet: Big Sur and Newer](https://github.com/samuelnotfound/miscellaneous-hackintosh-guides/blob/main/Broadcom_Ethernet_BigSur_and_newer/README.md)

*Adapted, extended, and updated with additional information.*

---

## Chipset Compatibility

### Natively Supported Devices

The following Broadcom chipsets are supported by the driver without additional configuration:

| Model | PCI ID |
| :--- | :--- |
| **BCM5764M** | `14e4:1684` |
| **BCM57761** | `14e4:16b0` |
| **BCM57762** | `14e4:1682` |
| **BCM57765** | `14e4:16b4` |
| **BCM57766** | `14e4:1686` |

These devices do not require `device-id` spoofing or the binary kernel patch described below.

### Devices Requiring Patching and Spoofing

The following chipsets require both:

- Spoofing their `device-id` to `14e4:16b4` (BCM57765).
- Applying the binary kernel patch described in this guide.

| Model | Original PCI ID |
| :--- | :--- |
| **BCM5700** | `14e4:1644` |
| **BCM5701** | `14e4:1645` |
| **BCM5702** | `14e4:1646` |
| **BCM5703** | `14e4:1647` |
| **BCM5717** | `14e4:1655` / `14e4:1665` |
| **BCM5718** | `14e4:1656` |
| **BCM5719** | `14e4:1657` |
| **BCM5725** | `14e4:1643` |
| **BCM5727** | `14e4:16f3` |
| **BCM5761** | `14e4:1688` |
| **BCM5762** | `14e4:1687` |
| **BCM57760** | `14e4:1690` |
| **BCM57764** | `14e4:1642` |
| **BCM57767** | `14e4:1683` |
| **BCM57781** | `14e4:16b1` |
| **BCM57782** | `14e4:16b7` |
| **BCM57785** | `14e4:16b5` |
| **BCM57786** | `14e4:16b3` |
| **BCM57787** | `14e4:1641` |
| **BCM57788** | `14e4:1691` |
| **BCM57790** | `14e4:1694` |
| **BCM57791** | `14e4:16b2` |
| **BCM57795** | `14e4:16b6` |
| **BCM5785** | `14e4:1699` / `14e4:16a0` |
| **BCM5787M** | `14e4:1693` |
| **Generic Adapter** | `14e4:1689` |

---

## Installation and Configuration

### Prerequisites

Before proceeding, make sure you have the following:

1. **CatalinaBCM5701Ethernet.kext** – Download the [pre-built kext (v1.0.2)](https://github.com/dortania/OpenCore-Legacy-Patcher/raw/refs/heads/main/payloads/Kexts/Ethernet/CatalinaBCM5701Ethernet-v1.0.2.zip) from the [OCLP repo](https://github.com/dortania/OpenCore-Legacy-Patcher/tree/main/payloads/Kexts/Ethernet).
2. **PCI Device Path** – Identify the PCI path of your Broadcom Ethernet controller. You can use [Hackintool](https://github.com/benbaker76/Hackintool) to locate it.

### 1. Add Device Properties

- Mount your System's ESP
- Open config.plist 
- Nvigate to: `DeviceProperties -> Add`
- Locate the PCI path of your Broadcom Ethernet controller and add the following properties:
	
	| Key | Type | Value |
	| :--- | :--- | :--- |
	| `device-id` | Data | `B4160000` |
	| `compatible` | String | `pci14e4,16b4` |

This spoofs the controller's device ID to **BCM57765** (`14e4:16b4`).

### 2. Add the Kext

1. Place `CatalinaBCM5701Ethernet.kext` in your `OC/Kexts/` directory.
2. Add the kext to `Kernel -> Add` in `config.plist`. Alternatively, use **ProperTree** to create an OC Snapshot (`Ctrl/Cmd + R`) and automatically populate the entry.
3. Set `MinKernel` to `20.0.0` for this kext entry.

### 3. Binary Kernel Patch

Add the following entry under `Kernel -> Patch`:

| Key | Value |
| :--- | :--- |
| **Identifier** | `com.apple.iokit.CatalinaBCM5701Ethernet` |
| **Comment** | `Broadcom BCM577XX Patch` |
| **Find** | `E8CA9EFF FF668983 00050000` |
| **Replace** | `B8B41600 00668983 00050000` |
| **MinKernel** | `20.0.0` |
| **Count** | `1` |
| **Enabled** | `true` |

This patch modifies the relevant driver code to accommodate the spoofed device-ID.

---

## Optional: Correcting the Reported Model Name

The driver may report the wrong Ethernet controller model in System Information because multiple chipsets use the same spoofed device ID.

An additional cosmetic kernel patch can correct the displayed model name.

### Example: BCM57765 → BCM57785

Add the following entry under `Kernel -> Patch`:

| Key | Value |
| :--- | :--- |
| **Identifier** | `com.apple.iokit.CatalinaBCM5701Ethernet` |
| **Comment** | `SysReport 57765 -> 57785 (Cosmetic)` |
| **Find** | `3537373635` |
| **Replace** | `3537373835` |
| **MinKernel** | `20.0.0` |
| **Count** | `0` |
| **Enabled** | `true` |

The hexadecimal sequences represent ASCII strings:

- `3537373635` → `57765`
- `3537373835` → `57785`

This patch only changes the model name *reported* by the driver. It does not affect device *functionality* or hardware identification.

---

## Credits

- **[Sunki](https://www.applelife.ru/threads/patching-applebcm5701ethernet-kext.27866/page-8#post-930901)** and **[Acidanthera](https://github.com/acidanthera/OpenCorePkg/blob/cb591b7671215b31dc4a2bc5b1e9da9c92eaebf4/Docs/Sample.plist#L837)** – Original driver patch sources.
- **[Andrey1970AppleLife](https://www.applelife.ru/threads/patching-applebcm5701ethernet-kext.27866/page-9#post-1031837)** – Patch guide reference and cosmetic patches.
- **Dortania / Khronokernel** – Pre-patched `CatalinaBCM5701Ethernet.kext` distributed with OpenCore Legacy Patcher.
