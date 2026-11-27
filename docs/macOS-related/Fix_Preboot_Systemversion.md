# Fixing incorrect Preboot metadata in APFS containers

## Overview

On multi-boot macOS systems, installing or updating one macOS version can interfere with the Preboot environment of another installation. This can result in incorrect version information being stored in the `SystemVersion.plist` files associated with individual APFS Preboot volumes.

A particularly well-known historical example is **macOS High Sierra (10.13)**. It could modify the `ProductVersion` entry in Preboot metadata, causing compatibility-check failures when attempting to boot newer macOS installations. In some cases, this resulted in the forbidden symbol (🚫) appearing during startup.

Adding `-no_compat_check` to the boot arguments could temporarily work around the compatibility check, but it did not correct the underlying Preboot metadata. The issue was particularly relevant to Hackintosh systems running High Sierra alongside newer macOS releases.

Similar problems can also occur following macOS installations or updates, where Preboot metadata no longer correctly reflects the associated System volume. This can affect both boot compatibility and OpenCore's ability to distinguish between macOS installations.

OpenCore uses version information from `SystemVersion.plist` to identify macOS installations and select the corresponding picker flavour icons. If multiple Preboot directories contain identical or incorrect version information, different macOS installations may consequently display the same icon.

This guide covers how to identify affected APFS volumes, inspect their Preboot metadata and restore a consistent Preboot environment.

## Diagnosis

Before attempting repairs, identify the affected APFS volumes and compare their version information.

### 1. Identify APFS containers and volumes

Run:

```bash
diskutil apfs list
```

This displays all APFS containers, including their associated volumes, roles, UUIDs, mount points and seal status.

Pay particular attention to:

- **System:** The macOS system volume.
- **Data:** The corresponding writable data volume.
- **Preboot:** The volume containing boot support files for the macOS installations.
- **Recovery:** The recovery environment.

Each APFS volume group has an associated Preboot directory identified by its System volume UUID.

### 2. Inspect Preboot SystemVersion.plist files

The Preboot volume is normally mounted at `/System/Volumes/Preboot`.

Use the following command to inspect the version information of all available Preboot installations:

```bash
for d in /System/Volumes/Preboot/*/System/Library/CoreServices; do
    echo "=== $d ==="
    plutil -p "$d/SystemVersion.plist" 2>/dev/null \
    | grep -E 'ProductName|ProductVersion|ProductBuildVersion'
done
```

Example output:

```text
=== /System/Volumes/Preboot/UUID-1/System/Library/CoreServices ===
  "ProductBuildVersion" => "..."
  "ProductName" => "macOS"
  "ProductVersion" => "15.x"

=== /System/Volumes/Preboot/UUID-2/System/Library/CoreServices ===
  "ProductBuildVersion" => "..."
  "ProductName" => "macOS"
  "ProductVersion" => "26.x"
```

Compare the reported versions with the corresponding macOS installations.

### 3. Identify the corresponding System volumes

Use the output from `diskutil apfs list` to identify the System volume UUIDs and their mount points.

The UUID of a System volume corresponds to its directory name on the Preboot volume.

To inspect the actual SystemVersion.plist files, use their respective mounted System volume paths:

```bash
plutil -p "/path/to/macOS/System/Library/CoreServices/SystemVersion.plist"
```

For example:

```bash
plutil -p "/Volumes/Tahoe/System/Library/CoreServices/SystemVersion.plist"
```

The currently running macOS installation may have its System volume mounted at a different location, such as `/System/Volumes/Update/mnt1`. Use `diskutil apfs list` to determine the actual mount point.

### 4. Compare the results

Check whether the following information matches:

- The macOS version reported by the actual System volume.
- The macOS version reported by its corresponding Preboot directory.
- The expected macOS version associated with that APFS volume group.

If multiple installations report identical version information despite running different macOS releases, their Preboot metadata may be incorrect.

**Important:** Do not assume that a broken System volume seal automatically indicates a damaged bootable system. APFS system snapshots and their underlying System volumes have separate seal states.

## Repair

There are several possible approaches, depending on the underlying problem.

### Option 1: Manually repair the Preboot environment

For manual repairs, MacSchrauber provides a collection of tools, including **PrebootFixer and Renamer**.

- Download MacSchrauber's ROM Dump tool from his [**GitHub releases**](https://github.com/Macschrauber/Macschrauber-s-Rom-Dump/releases), specifically `Download.the.Dumper.from.github.zip`.
- Extract the archive.
- Run the `Download the Dumper from github` applet.
- Select `Download`.
- This downloads `Macschrauber's Rom Dump.dmg`. The applet also provides the means to unlock and mount the password-protected disk image.
- Once mounted, navigate to:

  `/Volumes/Macschrauber's CMP Rom Dump/Readme & other tools/Preboot fixer and renamer`

- Locate and run **PrebootFixer and Renamer**.
- Follow the [**Preboot repair instructions**](https://github.com/Macschrauber/Macschrauber-s-Rom-Dump/blob/main/Rename_and_repair_preboot.md).

> [!IMPORTANT]
>
> Do not delete the `Download the Dumper from github` applet after downloading the disk image. It is also required to unlock and mount the password-protected image containing the tools.

Manual modification of `SystemVersion.plist` may be restricted by macOS system protections. Do not assume that changing file permissions alone will make the file writable.

### Option 2: Reinstall the affected macOS version

If manual repair attempts fail, for example because macOS prevents access to protected system files, reinstalling the affected macOS version is an alternative. The installer regenerates the system and associated Preboot support files, potentially restoring the correct version metadata.

This was the solution in my case: after a macOS Tahoe update, the Sequoia Preboot directory contained Tahoe's `SystemVersion.plist` information. Reinstalling macOS Sequoia restored the correct metadata and resolved the OpenCore picker icon issue.

This approach avoids manually modifying protected system files and does not normally require erasing the existing macOS installation.

## Best Practices

- Avoid mixing older macOS releases with newer installations in the same multi-boot environment unless there is a specific reason to do so.
- Be aware that installing or updating one macOS version can affect shared APFS boot infrastructure.
- After major macOS updates, verify that each installation has the correct Preboot metadata if unexpected boot picker behaviour occurs.
- Keep a backup of your EFI and OpenCore configuration before attempting repairs.
- Identify the affected APFS volume group before modifying or regenerating Preboot files.
