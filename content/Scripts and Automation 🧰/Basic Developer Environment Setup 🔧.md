This is an advanced, powerful script for setting up Developer Environments on Windows. 

In practice, this is a fairly uncomplicated script that will install and enable WSL (Windows Subsystems for Linux), create and activate an Ubuntu environment within WSL, download and install Git for Windows (Git Bash), download and install Visual Studio code, download and install Docker for Windows, and download and install Python and JS Node.

This is the perfect script for bootstrapping a full Windows dev machine for web, Python, containers, and WSL development. Simply save as "Setup-FullDevEnv.ps1," run as administrator, and we are off to the races!
## ⚙️ **What this script does**

✅ **WSL & Ubuntu:** Enables WSL + Virtual Machine Platform → Installs WSL2 → Installs Ubuntu → Downloads the WSL update MSI (optional but recommended for full kernel support).  
✅ **Git Bash:** Installs the latest Git for Windows silently.  
✅ **VS Code:** Installs latest stable silently, won’t auto-launch after install.  
✅ **Docker Desktop:** Installs Docker Desktop with `--quiet`.  
✅ **Python:** Installs latest stable Python (adjust version if needed), adds to PATH.  
✅ **Node.js:** Installs latest **LTS** Node version (adjust version if needed).

Here's the code:
```
# Setup-FullDevEnv.ps1

Write-Host "===================================================="
Write-Host "Setting up FULL Developer Environment on Windows:"
Write-Host "WSL (Ubuntu), Git Bash, VS Code, Docker, Python, Node.js"
Write-Host "===================================================="

# Ensure running as Administrator
if (-NOT ([Security.Principal.WindowsPrincipal] `
    [Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole(`
    [Security.Principal.WindowsBuiltInRole]::Administrator)) {
    Write-Warning "❌ You must run this script as Administrator!"
    exit 1
}

# Working directory
$WorkDir = "$env:TEMP\DevEnvSetup"
New-Item -ItemType Directory -Path $WorkDir -Force | Out-Null
Set-Location $WorkDir

# -----------------------------------------------
# 1️. Enable WSL & Install Ubuntu
# -----------------------------------------------

Write-Host "-----------------------------------------------"
Write-Host "Enabling WSL and installing Ubuntu..."
Write-Host "-----------------------------------------------"

dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart

# Download and install WSL update if needed
Invoke-WebRequest -Uri "https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi" -OutFile "wsl_update_x64.msi"
Start-Process msiexec.exe -ArgumentList "/i wsl_update_x64.msi /quiet /norestart" -Wait

wsl --set-default-version 2

# Install Ubuntu if not already installed
wsl --install -d Ubuntu

Write-Host "✅ WSL and Ubuntu setup done (first-time setup will open terminal to set username/password)."

# -----------------------------------------------
# 2️. Install Git Bash
# -----------------------------------------------

Write-Host "-----------------------------------------------"
Write-Host "Downloading and installing Git Bash..."
Write-Host "-----------------------------------------------"

$gitUrl = "https://github.com/git-for-windows/git/releases/latest/download/Git-2.45.1-64-bit.exe"
Invoke-WebRequest -Uri $gitUrl -OutFile "GitInstaller.exe"
Start-Process -FilePath "GitInstaller.exe" -ArgumentList "/VERYSILENT" -Wait

Write-Host "✅ Git Bash installed."

# -----------------------------------------------
# 3️. Install Visual Studio Code
# -----------------------------------------------

Write-Host "-----------------------------------------------"
Write-Host "Downloading and installing Visual Studio Code..."
Write-Host "-----------------------------------------------"

$vsCodeUrl = "https://update.code.visualstudio.com/latest/win32-x64-user/stable"
Invoke-WebRequest -Uri $vsCodeUrl -OutFile "VSCodeInstaller.exe"
Start-Process -FilePath "VSCodeInstaller.exe" -ArgumentList "/VERYSILENT /MERGETASKS=!runcode" -Wait

Write-Host "✅ Visual Studio Code installed."

# -----------------------------------------------
# 4️. Install Docker Desktop
# -----------------------------------------------

Write-Host "-----------------------------------------------"
Write-Host "Downloading and installing Docker Desktop..."
Write-Host "-----------------------------------------------"

$dockerUrl = "https://desktop.docker.com/win/main/amd64/Docker%20Desktop%20Installer.exe"
Invoke-WebRequest -Uri $dockerUrl -OutFile "DockerInstaller.exe"

# Silent install Docker
Start-Process -FilePath "DockerInstaller.exe" -ArgumentList "install --quiet" -Wait

Write-Host "✅ Docker Desktop installed (you may need to log out/log back in after installation)."

# -----------------------------------------------
# 5️. Install Python
# -----------------------------------------------

Write-Host "-----------------------------------------------"
Write-Host "Downloading and installing Python..."
Write-Host "-----------------------------------------------"

$pythonUrl = "https://www.python.org/ftp/python/3.12.2/python-3.12.2-amd64.exe"
Invoke-WebRequest -Uri $pythonUrl -OutFile "PythonInstaller.exe"

# Install Python for all users, add to PATH
Start-Process -FilePath "PythonInstaller.exe" -ArgumentList "/quiet InstallAllUsers=1 PrependPath=1" -Wait

Write-Host "✅ Python installed."

# -----------------------------------------------
# 6️. Install Node.js (LTS)
# -----------------------------------------------

Write-Host "-----------------------------------------------"
Write-Host "Downloading and installing Node.js (LTS)..."
Write-Host "-----------------------------------------------"

$nodeUrl = "https://nodejs.org/dist/v20.12.2/node-v20.12.2-x64.msi"
Invoke-WebRequest -Uri $nodeUrl -OutFile "NodeInstaller.msi"

Start-Process msiexec.exe -ArgumentList "/i NodeInstaller.msi /quiet /norestart" -Wait

Write-Host "✅ Node.js installed."

# -----------------------------------------------
# ✅ DONE
# -----------------------------------------------

Write-Host "-----------------------------------------------"
Write-Host "✅ Full Developer Environment Setup Complete!"
Write-Host "-----------------------------------------------"
Write-Host "👉 Open Ubuntu, VS Code, Docker Desktop, Git Bash and start coding!"
```

## ✅ **Next Steps**

After the script runs:

- **WSL (Ubuntu)** will launch on first run — you’ll set up a username/password.
    
- **Docker Desktop** may need you to **log out & back in** or reboot once so it can add your user to the `docker-users` group.
    
- In VS Code, install **Remote - WSL** for seamless dev inside Linux.
    
- Run `python --version` and `node --version` to confirm installs.

Need more tools to play around with? We also have a more advanced version of this same script, which can be found below!

[[Advanced Dev Environment Setup 🚀]]
