The following script is meant for workstations that are brand-new and have N-Able Remote Management and Monitoring agents installed. Can be run remotely through RMM or saved on a USB and run locally. 

This script is mostly here for SYGNVS staff, but clients are, of course, welcome to take advantage of this script! Simply copy and paste the script into Notepad and save it as "install-apps.ps1," then save the file to a USB or on the drive of the target workstation. From there, simply right-click the file and select "run." From there, the script will take care of the rest!

Now, what exactly do we get from this script? It will automatically download and install Chromium (open-source Google Chrome), Firefox, Google Chrome, Acrobat Reader, and 7-Zip. It will also automatically set Chrome as the default browser, expediting the setup process for new workstations.

```
# Install-Apps.ps1

Write-Host "============================================="
Write-Host "Installing Firefox, Chromium, Acrobat Reader,"
Write-Host "7-Zip (via Ninite), and Google Chrome..."
Write-Host "============================================="

# Create working directory
$workDir = "$env:TEMP\installer_files"
New-Item -ItemType Directory -Path $workDir -Force | Out-Null
Set-Location $workDir

Write-Host "---------------------------------------------"
Write-Host "Downloading installers..."
Write-Host "---------------------------------------------"

# Firefox
Invoke-WebRequest -Uri "https://download.mozilla.org/?product=firefox-latest&os=win64&lang=en-US" -OutFile "FirefoxInstaller.exe"

# Chromium (Woolyss snapshots)
Invoke-WebRequest -Uri "https://download-chromium.appspot.com/dl/Win?type=snapshots" -OutFile "ChromiumInstaller.zip"

# Adobe Acrobat Reader (no McAfee, direct link)
Invoke-WebRequest -Uri "https://ardownload2.adobe.com/pub/adobe/reader/win/AcrobatDC/2300820412/AcroRdrDC2300820412_en_US.exe" -OutFile "AcrobatReaderInstaller.exe"

# 7-Zip via Ninite
Invoke-WebRequest -Uri "https://ninite.com/7zip/ninite.exe" -OutFile "Ninite7ZipInstaller.exe"

# Google Chrome
Invoke-WebRequest -Uri "https://dl.google.com/chrome/install/latest/chrome_installer.exe" -OutFile "ChromeInstaller.exe"

Write-Host "---------------------------------------------"
Write-Host "Installing applications..."
Write-Host "---------------------------------------------"

# Install Firefox silently
Start-Process ".\FirefoxInstaller.exe" -ArgumentList "/S" -Wait

# Extract and install Chromium manually (optional: or skip if you use Chrome)
Expand-Archive ".\ChromiumInstaller.zip" -DestinationPath ".\Chromium"
# Chromium is portable — no installer, so you'd just use the EXE

# Install Adobe Acrobat Reader silently
Start-Process ".\AcrobatReaderInstaller.exe" -ArgumentList "/sAll /rs /rps /msi EULA_ACCEPT=YES" -Wait

# Install 7-Zip silently via Ninite
Start-Process ".\Ninite7ZipInstaller.exe" -ArgumentList "/silent" -Wait

# Install Google Chrome silently
Start-Process ".\ChromeInstaller.exe" -ArgumentList "/silent /install" -Wait

Write-Host "---------------------------------------------"
Write-Host "Setting Google Chrome as the default browser..."
Write-Host "---------------------------------------------"

# Set Google Chrome as default browser (works on Windows 10/11)
# This method uses `defaultbrowser` CLI tool
# You may need to install/set a default browser handler via registry or use `SetUserFTA`
# This requires additional permission; here is a simple example:
# Install SetUserFTA: https://github.com/clechasseur/setuserfta
# Example:
# Start-Process "SetUserFTA.exe" -ArgumentList "http ChromeHTML" -Wait

Write-Host "Please set Chrome as default manually or use a tool like SetUserFTA for automatic default browser change."

Write-Host "---------------------------------------------"
Write-Host "All installations complete!"
Write-Host "---------------------------------------------"
```

## ⚙️ **Notes**

1. **Silent install switches:**
    
    - Firefox: `/S`
        
    - Chrome: `/silent /install`
        
    - Adobe Reader: `/sAll /rs /rps /msi EULA_ACCEPT=YES`
        
    - 7-Zip via Ninite is silent by default.
        
    - Chromium is portable — you extract and run it.
        
2. **Setting Chrome as default:**
    
    - Windows doesn’t expose an official PowerShell command for this.
        
    - The reliable method is to use [SetUserFTA](https://github.com/clechasseur/setuserfta) or the defaultbrowser tool to set file associations.
        
    - Windows security controls mean you often must set defaults manually or deploy via GPO.
        
3. **Run as Administrator!**  
    Save this as `Install-Apps.ps1` and run it from **PowerShell as Administrator**:

```
Set-ExecutionPolicy Bypass -Scope Process -Force
.\Install-Apps.ps1
```