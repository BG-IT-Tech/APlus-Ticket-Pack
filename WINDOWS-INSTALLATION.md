# Windows Installation Guide

## Minimum System Requirements
- **Processor**: 1 GHz or faster compatible processor
- **RAM**: 2 GB or more
- **Storage**: 20 GB of available disk space or more
- **Graphics**: DirectX 9 or later with WDDM 1.0 driver

## USB/DVD Specifications
- **USB**: 4 GB or larger USB flash drive
- **DVD**: Blank DVD (for burning the installation ISO)

## Detailed Installation Steps
1. **Create Installation Media**: Use the Windows Media Creation Tool to create a bootable USB or DVD.
2. **Boot from Media**: Insert the USB or DVD into your computer and restart. Access the BIOS/UEFI settings and set the boot order to prioritize the USB or DVD.
3. **Install Windows**:
   - Choose your language, time and currency format, and keyboard layout.
   - Click 'Install Now'.
   - Accept the license terms.
   - Select ‘Custom: Install Windows only (advanced)’ for a fresh installation.
   - Select the drive where you want to install Windows and click 'Next'.
4. **Configuration**: Follow the on-screen prompts to configure your settings (e.g., user account, privacy settings).

## Troubleshooting
- **PowerShell Commands**:
  - `Get-ComputerInfo` - To retrieve computer info.
  - `Get-WindowsFeature` - To list installed features.

- **Command Prompt Commands**:
  - `sfc /scannow` - To scan and repair system files.
  - `chkdsk` - To check the disk for errors.

## Post-Installation Steps
1. **Update Windows**: Ensure your system is up to date by checking for updates in Settings.
2. **Install Drivers**: Visit manufacturer’s website to download the latest drivers.
3. **Configure Settings**: Customize your system settings, such as privacy settings and user accounts.

## FAQs
- **How do I recover my Windows if it fails to boot?**
  - You can use the recovery options available on the installation media.

- **What should I do if Windows does not recognize my USB drive?**
  - Try a different USB port or check disk management settings.