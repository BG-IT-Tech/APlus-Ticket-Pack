# Comprehensive Windows Installation Guide

## Table of Contents
1. [Pre-Installation Checks](#pre-installation-checks)
2. [Minimum System Requirements](#minimum-system-requirements)
3. [Installation Media Preparation](#installation-media-preparation)
4. [Step-by-Step Installation](#step-by-step-installation)
5. [Post-Installation Configuration](#post-installation-configuration)
6. [Troubleshooting Guide](#troubleshooting-guide)
7. [Helpful Resources](#helpful-resources)

---

## Pre-Installation Checks

### System Compatibility Verification

Before beginning the installation, verify your system meets requirements:

```powershell
# Check processor details
Get-WmiObject -Class Win32_Processor | Select-Object Name, MaxClockSpeed, NumberOfCores

# Check RAM (in GB)
$ram = (Get-WmiObject -Class Win32_ComputerSystem).TotalPhysicalMemory / 1GB
Write-Host "Total RAM: $ram GB"

# Check free disk space (in GB)
$disk = Get-PSDrive C | Select-Object @{Name="FreeSpace(GB)";Expression={$_.Free/1GB}}
$disk

# Check for TPM 2.0 (Windows 11 requirement)
Get-WmiObject -Class Win32_Tpm | Select-Object SpecVersion, Manufacturer

# Check Secure Boot status
Confirm-SecureBootUEFI

# Check DirectX version
$DXVersion = (Get-ItemProperty -Path 'HKLM:\SOFTWARE\Microsoft\DirectX' -Name Version).Version
Write-Host "DirectX Version: $DXVersion"
```

### Disk Health Checks

```powershell
# Check disk health
Get-Volume -DriveLetter C | Get-StorageReliabilityCounter

# Run full CHKDSK scan (requires admin and restart)
chkdsk C: /F /R

# For SSD health monitoring
Get-PhysicalDisk | Select-Object FriendlyName, @{N='HealthStatus';E={$_.HealthStatus}}, @{N='Operational Status';E={$_.OperationalStatus}}

# Check disk space available
$disk = Get-PSDrive C
Write-Host "Free Space: $([Math]::Round($disk.Free/1GB, 2)) GB"
Write-Host "Used Space: $([Math]::Round($disk.Used/1GB, 2)) GB"
Write-Host "Total Size: $([Math]::Round(($disk.Used + $disk.Free)/1GB, 2)) GB"
```

**Troubleshooting Disk Issues:**
- [Microsoft Disk Management Guide](https://support.microsoft.com/en-us/windows/disk-management-in-windows)
- [Check Disk Health](https://support.microsoft.com/en-us/windows/check-your-pc-s-health-in-windows)

### Data Backup

```powershell
# List important directories to backup
$backupDirs = @("$env:USERPROFILE\Documents", "$env:USERPROFILE\Desktop", "$env:USERPROFILE\Pictures")
foreach ($dir in $backupDirs) {
    $size = (Get-ChildItem -Path $dir -Recurse | Measure-Object -Property Length -Sum).Sum / 1GB
    Write-Host "$dir : $size GB"
}

# Use built-in backup tool
# Settings > System > Storage > Backup options
```

---

## Minimum System Requirements

| Component | Windows 10 | Windows 11 |
|-----------|-----------|-----------|
| **Processor** | 1 GHz with SSE2 support | 1 GHz, 2+ cores (8th Gen Intel or Ryzen 2000+) |
| **RAM** | 1 GB (32-bit) / 2 GB (64-bit) | 4 GB minimum |
| **Disk Space** | 16 GB (32-bit) / 20 GB (64-bit) | 64 GB minimum SSD |
| **Graphics** | DirectX 9 compatible | DirectX 12 compatible |
| **Display** | 800 x 600 minimum | 720p (1280 x 720) minimum |
| **Firmware** | BIOS or UEFI | UEFI with Secure Boot |
| **TPM** | Optional | TPM 2.0 required |

### Recommended Specifications
- **Processor**: Multi-core 2.0 GHz or faster
- **RAM**: 8 GB or more
- **Storage**: 128+ GB SSD
- **Internet**: Broadband for updates

---

## Installation Media Preparation

### USB Drive Specifications

- **Capacity**: Minimum 8 GB (16 GB recommended)
- **Speed**: USB 3.0+ recommended
- **Type**: USB flash drive or external drive
- **Format**: Will be formatted to FAT32/NTFS

### USB Drive Health Check

```powershell
# List all connected drives
Get-WmiObject -Class Win32_LogicalDisk | Select-Object DeviceID, Size, FreeSpace

# Get detailed USB drive info
Get-WmiObject -Class Win32_LogicalDisk -Filter "DriveType=2" | Select-Object DeviceID, VolumeName, Size, FreeSpace

# Check for bad sectors (run as Administrator)
Repair-Volume -DriveLetter E -OfflineScanAndFix

# Verify USB drive connectivity
Get-WmiObject -Class Win32_USBControllerDevice
```

**Troubleshooting USB Issues:**
- [USB Drive Not Recognized](https://support.microsoft.com/en-us/windows/external-drive-not-showing-up)
- [Fix USB Write Protection](https://support.microsoft.com/en-us/windows/write-protected-usb-drive-fix)

### Creating Windows Installation Media

**Download Tools:**
- [Windows 10 Media Creation Tool](https://www.microsoft.com/en-us/software-download/windows10)
- [Windows 11 Media Creation Tool](https://www.microsoft.com/en-us/software-download/windows11)

**Steps:**

1. **Insert and Prepare USB Drive**
```powershell
# Format USB drive (WARNING: deletes all data)
$driveLetter = "E"
Format-Volume -DriveLetter $driveLetter -FileSystem NTFS -NewFileSystemLabel "WindowsInstall" -Confirm:$false
```

2. **Run Media Creation Tool**
   - Right-click MediaCreationTool.exe → "Run as administrator"
   - Select "Create installation media for another PC"
   - Choose language and Windows edition (64-bit recommended)

3. **Select USB Flash Drive**
   - Choose your USB drive from the list
   - **Verify the correct drive** to avoid data loss

```powershell
# Monitor creation progress
Get-Process MediaCreationTool

# Free up disk space if needed
$size = (Get-ChildItem -Path "$env:TEMP" -Recurse | Measure-Object -Property Length -Sum).Sum / 1GB
Write-Host "Temp folder size: $size GB"
```

4. **Verify Installation Media**

```powershell
# Check USB contents
Get-ChildItem -Path "E:\*" -Recurse | Measure-Object -Property Length -Sum

# Verify required files
$requiredFiles = @("bootmgr", "boot", "sources", "efi", "windows")
foreach ($file in $requiredFiles) {
    $exists = Test-Path "E:\$file"
    Write-Host "$file : $(if ($exists) { 'Found' } else { 'Missing' })"
}

# Verify ISO hash before burning
$isoHash = Get-FileHash "Path\To\Windows.iso" -Algorithm SHA256
Write-Host "ISO SHA256: $($isoHash.Hash)"
```

**Media Creation Troubleshooting:**
- Ensure sufficient free space (at least 20GB)
- Try different USB port
- Disable antivirus temporarily
- [Media Creation Tool Issues](https://support.microsoft.com/en-us/windows/get-help-with-windows-10-installation)

---

## Step-by-Step Installation

### Step 1: Boot from Installation Media

```powershell
# Common boot menu keys by manufacturer:
# Dell: F12, F2
# HP: F9, F2
# Lenovo: F12, F2
# ASUS: F8, F2
# MSI: F11, Delete
```

**Instructions:**
1. Insert USB drive or DVD
2. Restart computer
3. Press boot menu key during startup
4. Select USB/DVD from boot menu
5. Press Enter

### Step 2: Windows Setup

1. Select language, time format, and keyboard
2. Click "Install Now"
3. Enter Windows product key (or skip for evaluation)
4. Accept license terms

### Step 3: Choose Installation Type

**Custom Installation (Recommended for Clean Install):**
- Select "Custom: Install Windows only (advanced)"
- Choose partition or create new one
- Minimum 50GB recommended (100GB+ for optimal performance)

```powershell
# Pre-installation partition check
Get-Disk | Select-Object Number, @{N='Size(GB)';E={$_.Size/1GB}}
Get-Partition | Select-Object DriveLetter, @{N='Size(GB)';E={$_.Size/1GB}}
```

### Step 4: Complete Installation

- Windows will copy files and restart multiple times
- Computer will restart automatically during process
- Installation typically takes 15-30 minutes

---

## Post-Installation Configuration

### Update Drivers

```powershell
# List all devices with driver info
Get-WmiObject -Class Win32_PnPSignedDriver | Select-Object DeviceName, DriverVersion, Manufacturer | Format-Table -AutoSize

# Check for driver updates
Get-WindowsDriver -Online -All | Where-Object {$_.State -ne 'Staged'}

# Update drivers
Install-WindowsUpdate -AcceptAll -Verbose
```

**Resources:**
- [Update Drivers Guide](https://support.microsoft.com/en-us/windows/update-drivers-in-windows)
- [Device Manager Troubleshooting](https://support.microsoft.com/en-us/windows/update-a-driver-for-hardware-that-isn-t-working-well)

### Install Windows Updates

```powershell
# Check for available updates
Get-WindowsUpdate

# Install all available updates
Install-WindowsUpdate -AcceptAll -Verbose

# View installed updates
Get-HotFix | Select-Object HotFixID, InstalledOn, Description | Sort-Object InstalledOn -Descending
```

### Verify Installation

```powershell
# Check Windows version
Get-ComputerInfo | Select-Object OsName, OsVersion, OsBuildNumber

# Check system uptime
Get-ComputerInfo | Select-Object LastBootUpTime

# Check Windows activation
Get-CimInstance SoftwareLicensingProduct -Filter "Name like 'Windows'" | Select-Object Name, LicenseStatus

# List installed software
Get-WmiObject -Class Win32_Product | Select-Object Name, Version | Sort-Object Name
```

---

## Troubleshooting Guide

### Issue 1: Installation Fails or Hangs

**Symptoms:** Setup stops responding or restarts continuously

**Solutions:**

```powershell
# Check available disk space
Get-WmiObject -Class Win32_LogicalDisk | Select-Object DeviceID, @{N='FreeSpace(GB)';E={$_.FreeSpace/1GB}}

# Disconnect all USB devices except keyboard/mouse
# Disable antivirus temporarily
# Try with different USB port
# Recreate installation media
```

**Resources:**
- [Installation Hangs](https://support.microsoft.com/en-us/windows/windows-installation-fails-or-hangs)

### Issue 2: Blue Screen Errors (BSOD)

**Symptoms:** Computer displays blue screen with stop code

**Solutions:**

```powershell
# Check event logs for errors
Get-WinEvent -LogName System | Where-Object {$_.LevelDisplayName -eq 'Error'} | Select-Object TimeCreated, Message, Id -First 20

# Check for bad sectors
Check-Volume -DriveLetter C -FullScan

# Check BSOD history
Get-WinEvent -LogName System -FilterXPath "*[System[EventID=1001]]" | Select-Object TimeCreated, Message
```

**Troubleshooting Steps:**
- Note the stop code displayed
- Update BIOS/firmware
- Check hardware compatibility
- Disable fast startup: `powercfg /h off`

**Resources:**
- [Blue Screen Errors](https://support.microsoft.com/en-us/windows/blue-screen-error-recovery)
- [Stop Codes Reference](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/bug-check-code-reference)

### Issue 3: Driver Issues

**Symptoms:** Hardware not functioning, missing devices

**Solutions:**

```powershell
# List unknown devices
Get-WmiObject -Class Win32_PnPEntity | Where-Object {$_.Status -like '*Error*'} | Select-Object Name, DeviceID

# Check Device Manager errors
Get-WmiObject -Class Win32_PnPDevice | Where-Object {$_.ConfigManagerErrorCode -ne 0} | Select-Object Name, ConfigManagerErrorCode

# Reinstall device drivers
# Download chipset drivers first from manufacturer
# Install graphics and other drivers in order
```

**Resources:**
- [Driver Update Guide](https://support.microsoft.com/en-us/windows/update-drivers-in-windows)
- [Find Device Drivers](https://support.microsoft.com/en-us/windows/find-and-install-device-drivers)

### Issue 4: Windows Update Failures

**Symptoms:** Updates fail to install or get stuck

**Solutions:**

```powershell
# Reset Windows Update components
Stop-Service -Name wuauserv -Force
Stop-Service -Name cryptSvc -Force

Remove-Item "C:\Windows\SoftwareDistribution\*" -Recurse -Force -ErrorAction SilentlyContinue

Start-Service -Name wuauserv
Start-Service -Name cryptSvc

# Run Windows Update Troubleshooter
Get-WindowsUpdate -Verbose
```

**Resources:**
- [Windows Update Troubleshooter](https://support.microsoft.com/en-us/windows/windows-update-troubleshooter)

### Issue 5: Network Issues After Installation

**Symptoms:** No internet connection or network adapter not found

**Solutions:**

```powershell
# List network adapters
Get-NetAdapter

# Check network status
Get-NetAdapterStatistics

# Reset network settings
Restart-NetAdapter -Name "Ethernet"

# Renew DHCP lease
ipconfig /renew
```

**Resources:**
- [Network Troubleshooting](https://support.microsoft.com/en-us/windows/fix-network-connection-issues)

---

## Helpful Resources

### Official Microsoft Documentation
- [Windows 10 Installation](https://support.microsoft.com/en-us/windows/install-windows-10)
- [Windows 11 Installation](https://support.microsoft.com/en-us/windows/install-windows-11)
- [Windows Hardware Compatibility](https://docs.microsoft.com/en-us/windows-hardware/drivers/dashboard/win-hardware-compatibility-list)

### Support & Community
- [Microsoft Support](https://support.microsoft.com/en-us/windows)
- [Windows Community Forums](https://answers.microsoft.com/en-us/windows/forum)
- [Tech Community](https://techcommunity.microsoft.com/)

### Troubleshooting Tools
- [Windows Update Troubleshooter](https://support.microsoft.com/en-us/windows/windows-update-troubleshooter)
- [Windows Hardware Compatibility Checker](https://www.microsoft.com/en-us/windows/windows-11-specifications)
- [Disk Management](https://support.microsoft.com/en-us/windows/disk-management-in-windows)

### Additional Resources
- [Secure Boot Configuration](https://support.microsoft.com/en-us/windows/what-is-secure-boot)
- [TPM Security Information](https://docs.microsoft.com/en-us/windows/security/information-protection/tpm/trusted-platform-module-top-node)
- [System Recovery Options](https://support.microsoft.com/en-us/windows/recovery-options-in-windows)
- [BIOS/UEFI Updates](https://support.microsoft.com/en-us/windows/how-to-update-your-bios)

---

## Installation Checklist

- [ ] Verify system meets minimum requirements
- [ ] Backup all important data
- [ ] Check disk health and free space
- [ ] Update current drivers and BIOS
- [ ] Create Windows installation media
- [ ] Verify installation media contents
- [ ] Disable antivirus temporarily
- [ ] Boot from installation media
- [ ] Complete Windows installation
- [ ] Install critical updates
- [ ] Install chipset drivers
- [ ] Install GPU and other drivers
- [ ] Verify system stability
- [ ] Re-enable antivirus
- [ ] Activate Windows

---

**Document Information**
- **Created**: 2026-03-29
- **Author**: BG-IT-Tech
- **Version**: 2.0
- **Last Updated**: 2026-03-29

For additional help, visit [Microsoft Support](https://support.microsoft.com/) or contact your IT department.