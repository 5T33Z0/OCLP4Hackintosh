# Restoring Features in macOS Tahoe Without OCLP

## The Problem

At the time of writing, there is no release date for a version of OpenCore Legacy Patcher (OCLP) that fully supports running macOS Tahoe on legacy hardware.

The primary blocker is Apple's removal of the legacy Intel Metal graphics drivers required by Ivy Bridge and older Intel GPU architectures (HD 4000, HD 3000, and earlier). Unlike previous macOS releases, Tahoe no longer includes the necessary Metal frameworks and GPU drivers that OCLP traditionally restores through root patches.

Because these drivers are missing from the operating system itself, OCLP cannot simply patch or re-enable them. The OCLP developers first need to determine whether the removed frameworks can be successfully backported from earlier macOS versions or if an entirely different solution is required. Until that work is completed, Macs that depend on these legacy GPUs cannot receive the graphics acceleration required for a usable desktop experience.

As a result, any OCLP root patches that depend on legacy graphics support are currently unavailable in Tahoe. This also affects other hardware features that rely on the root patching framework, such as legacy Broadcom Wi-Fi and AppleHDA audio on some systems.

Fortunately, if your system **does not require graphics root patches** and only needs Wi-Fi or audio functionality restored, there are alternative methods that can re-enable those features without relying on OCLP.

## Alternatives to OCLP

### For Wi-Fi

If your system doesn't require additional root patches for running macOS besides enabling Wi-Fi, you can use [**Wi-Fi Patcher Pro**](https://github.com/Mirone/WiFiPatcherPro) instead. It provides a one-click solution that automates the entire process—including kext installation, config changes, and root patches—to restore Broadcom and Intel wireless support.

**For more details check the guide:** [**Fixing WiFi and Bluetooth in macOS Sonoma+**](/Enable_Features/WiFi_Sonoma.md)

### For Audio

**Alternative 1: Using MyKextInstaller**

If your system doesn't require any graphics root patches to run macOS, you can use MyKextInstaller to install an older version of `AppleHDA.kext` and restore native analog audio. Follow the instructions in the [**MyKextInstaller**](https://github.com/Mirone/MyKextInstaller) repository.

**Alternative 2: Using VoodooHDA**

As a fallback, you can use `VoodooHDA.kext`, an alternative audio driver that doesn't rely on `AppleHDA`. Chris1111 provides an updated [**VoodooHDA Installer for macOS Tahoe**](https://github.com/chris1111/VoodooHDA-Tahoe) that simplifies the installation process.

**For more details check the guide:** [**Fixing Audio in macOS Tahoe**](/Enable_Features/Audio_Tahoe.md)
