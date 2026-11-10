# Recovering from Failed Root Patching Attempts

## About

Whenever a new macOS update is released, there's a chance that Apple has removed or changed system components that root patches depend on. As a result, root patching might fail or cause issues that render macOS unusable.

This happened with the release of macOS 14.2 Beta 3, when users could no longer log in to macOS on Ivy Bridge systems after applying root patches with OCLP 1.2.1 to re-enable Intel HD 4000 graphics. The patches caused `WindowServer` to crash. Fortunately, the issue was resolved in OCLP 1.3.0. However, similar problems can occur again whenever Apple releases a new macOS update.

Below are several options for recovering from failed root patching attempts. They are sorted from easiest (Scenario 1) to most difficult (Scenario 3).

## Preparations

Before reverting root patches, take the following precautions if your system requires root patches to access the internet:

- Check whether a newer version of [OCLP](https://github.com/dortania/OpenCore-Legacy-Patcher/releases) is available and download it.
- If the macOS version you want to install is newer than the latest official OCLP release, check whether newer commits are available in the source code and [download the latest nightly build](https://github.com/dortania/OpenCore-Legacy-Patcher/blob/main/SOURCE.md) instead. Make sure to use a build that includes support for your macOS version.
- If your system requires root patches to enable network adapters, download the latest [Kernel Debug Kit](https://github.com/dortania/KdkSupportPkg/releases) (KDK) before reverting root patches. Expand the "Assets" section and download the `.dmg` file.

> [!NOTE]
>
> If OCLP is already installed and your system has internet access, OCLP automatically detects when a macOS update is being downloaded. It then notifies you that the latest KDK needs to be installed to successfully perform the update:
>
> ![kdkdl](https://github.com/5T33Z0/OC-Little-Translated/assets/76865553/1bd1f9f0-9ea9-4ad4-b112-93bad9261b56)
>
> Simply click "OK" and OCLP will take care of the rest.

## Scenario 1: Reverting Root Patches When macOS Is Still Usable

If you can still boot into macOS and use it, follow these steps to revert the root patches:

1. Launch OpenCore Legacy Patcher.
2. Click on "Post-Install Root Patch".
3. Click on "Revert Root Patches".
4. This will restore the previous unpatched system snapshot.
5. Reboot macOS without root patches.
6. Click "Cancel" when OCLP prompts you to reapply root patches.
7. Instead, check whether a newer version of OCLP is available, download it, and apply the root patches again.
8. Reboot.

Everything should be back to normal.

## Scenario 2: Reverting Root Patches in Safe Mode

If you've applied root patches that leave macOS in an unusable state, for example, after installing iGPU/GPU drivers, you might still be able to revert them by booting macOS into Safe Mode.

Safe Mode prevents certain graphics drivers and other system extensions from loading. This might allow you to log in to macOS and revert the root patches.

1. In OpenCore's boot menu, select macOS.
2. Hold <kbd>Shift</kbd> and press <kbd>Enter</kbd> to boot into Safe Mode. This takes significantly longer than a normal boot.
3. Click on "Post-Install Root Patch".
4. Click on "Revert Root Patches".
5. This will restore the previous unpatched system snapshot.
6. Reboot macOS without root patches.
7. Click "Cancel" when OCLP prompts you to reapply root patches.
8. Instead, check whether a newer version of OCLP is available, download it, and apply the root patches again.
9. Reboot.

Everything should be back to normal.

## Scenario 3: Reverting Root Patches via macOS Recovery and Terminal

This is the last resort if macOS won't boot normally or in Safe Mode. The idea is to boot into Recovery and use Terminal to restore the last working, unpatched macOS snapshot.

1. In OpenCore's boot menu, press <kbd>Spacebar</kbd> to show hidden entries.
2. Select the Recovery partition associated with the affected macOS installation.
3. Press <kbd>Enter</kbd> to boot into Recovery.
4. Once the Recovery menu appears, select **Disk Utility**.
5. Change the "View" setting to "Show All Volumes" (or the equivalent option, depending on your language settings).
6. In the sidebar, locate the "Volumes of XYZ" entry (XYZ = the name of the volume containing the broken macOS installation). For example, if your volume is named "Sonoma", look for "Volumes of Sonoma".
7. Select the volume and click "Info" (or press <kbd>Cmd</kbd> + <kbd>I</kbd>) to open its information dialog.
8. Look for the "Mount Point" entry. It should display a path similar to `/Volumes/XYZ`.
9. Take note of this path and close Disk Utility.
10. Back in the Recovery menu, select "Utilities > Terminal" from the menu bar.
11. Enter the following commands one by one (replace `XYZ` with the name of your volume):

    ```bash
    mount -uw "/Volumes/XYZ"
    bless --mount "/Volumes/XYZ" -bootefi --last-sealed-snapshot
    ```

12. Close Terminal.
13. Restart the system using the Apple menu.
14. Boot into macOS without root patches.
15. Click "Cancel" when OCLP prompts you to reapply root patches.
16. Instead, check whether a newer version of OCLP is available, download it, and apply the root patches again.
17. Reboot.

Everything should be back to normal.

## Scenario 4: Root Patches Cannot Be Applied Because a Newer macOS Version Is Being Downloaded in the Background

**Error message:** `Please ensure no updates are pending`

See Dortania's troubleshooting guide:

[**System version mismatch error when root patching**](https://github.com/dortania/OpenCore-Legacy-Patcher/blob/main/docs/TROUBLESHOOT-APP.md#system-version-mismatch-error-when-root-patching)

---

> [!IMPORTANT]
>
> If you don't have internet access, root patching might fail because the required Kernel Debug Kit (KDK) is missing. In that case, install the appropriate KDK first and then run OCLP again.

> [!TIP]
>
> - If your system can access the internet via Ethernet without root patches, you can also use Safari in Recovery to follow these instructions.
> - There's also a very helpful [YouTube video](https://youtu.be/mNcjmvzS0Vo?si=OtNeB4r1q3s3sW9T) that walks you through the entire process.

## Links

- [Bless, and What Makes a Volume Bootable](https://eclecticlight.co/2018/08/30/booting-the-mac-bless-and-what-makes-a-volume-bootable/)
