[![OpenCore Version](https://img.shields.io/badge/OpenCore_Version:-0.9.4+-success.svg)](https://github.com/acidanthera/OpenCorePkg) ![macOS](https://img.shields.io/badge/Supported_macOS:-≤15.x-white.svg)

# Installing macOS Ventura and newer on Ivy Bridge systems

## macOS installation
With all the prep work out of the way, you can now upgrade to macOS Ventura. Depending on the version of macOS you are coming from, the installation process differs.

### Getting macOS
- Download the latest release of [OpenCore Patcher GUI App](https://github.com/dortania/OpenCore-Legacy-Patcher/releases) and run it
- Click on "Create macOS Installer"
- Click on "Download macOS Installer"
- Select the macOS version you want to install
- Once the download is completed the "Install macOS…" app will be located in your "Programs" folder

> [!NOTE]
> 
> OCLP can also create a USB installer if you want to perform a clean install (highly recommended). Creating a USB installer is a necessity if you want to install an older OS since macOS does not allow downgrading.

### Option 1: Upgrading from macOS 11.3 or newer 
Only applicable when upgrading from macOS 11.3+. If you are on macOS Catalina or older, use Option 2 instead.

- Run the "Install macOS…" App
- There will be a few reboots
- Boot from the new macOS Partition until it's no longer present in the Boot Picker

Once the installation has finished and the system boots it will run without graphics acceleration if you only have an iGPU or if you GPU is not supported by macOS. We will address this in Post-Install.

> [!TIP]
> 
> Instead of upgrading your runnning macOS installation, create a new APFS volume and install macOS on there. This way you can always revert back to your previous macOS installation if you are facing issues with the new macOS version.

### Option 2: Upgrading from macOS Catalina or older
When upgrading from macOS Catalina or older a clean install from USB flash drive is recommended. To create a USB Installer, you can use OpenCore Legacy Patcher:

- Run Disk utility
- Create a new APFS Volume on your internal SSD or use a separate internal disk (at least 60 GB in size) for installing macOS 13. DON'T install it on an external drive – it won't boot!
- Attach an empty USB flash drive for creating the installer (16 GB+)
- Run OCLP and follow the [**instructions**](https://dortania.github.io/OpenCore-Legacy-Patcher/INSTALLER.html#creating-the-installer)
- Once the USB Installer has been created, do the following:
	- Copy the OpenCore-Patcher App to the USB Installer
	- Add additional tools (optional, in case internet is not working): 
		- Add Python Installer
		- Add MountEFI
		- Add ProperTree
- Reboot 
- Select "Install macOS Ventura" from the BootPicker
- Install macOS Ventura on the volume you prepared earlier 
- There will be a few reboots during installation. Boot from the new "Install macOS" Partition until it's no longer present in the Boot Picker
- Once the macOS Ventura installation is finished, switch back to an SMBIOS best suited for your Ivy Bridge CPU mentioned earlier.

After the installation is completed and the system boots it will run without hardware graphics acceleration if you only have an iGPU or if you GPU is no longer supported by macOS. We will address this in Post-Install. 

---

[← **Previous: Config Edits**](Ivy_config.md) | [**Next: Post-Installation →**](Ivy_Post-Install.md)