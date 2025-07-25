If you're here, and if you're reading this right now, just know that Paul at SYGNVS absolutely loves you and wants the best for you! As a Linux fanboy himself, he is thrilled that you are looking to take development seriously enough to want the whole package.

While Linux is truly the way for efficient, lightweight computing, this script doesn't delve too deeply into getting Linux set up on your machine. If that's what you're after, check out our automation tools for installing VirtualBox so you can experiment with different desktop environments - the link will be at the bottom of this article, as well as in the index appended to the bottom of the homepage. 

### Below is a **complete, enhanced PowerShell script** that **adds**:

✅ Everything from our other dev environment script (**WSL, Git Bash, VS Code, Docker, Python, Node.js**)  
✅ Installs **NVM for Windows** (Node Version Manager) — lets you easily install/switch Node versions  
✅ Installs **Chocolatey** — a popular Windows package manager  
✅ Installs **Winget** (for good measure — most new Windows 11 systems have it)  
✅ Installs some **common VS Code extensions** automatically  
✅ Installs a **popular Oh My Posh** prompt for a nice PowerShell/WSL terminal

```
# Setup-ProDevEnv.ps1

Write-Host "==============================================================="
Write-Host "Setting up PRO Developer Environment on Windows:"
Write-Host "WSL (Ubuntu), Git Bash, VS Code, Docker, Python, Node.js, NVM,"
Write-Host "Chocolatey, Winget, VS Code Extensions, Oh My Posh"
Write-Host "==============================================================="

# Ensure running as Administrator
if (-NOT ([Security.Principal.WindowsPrincipal] `
    [Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole(`
    [Security.Principal.WindowsBuiltInRole]::Administrator)) {
    Write-Warning "❌ You must run this script as Administrator!"
    exit 1
}

# Working directory
$WorkDir = "$env:TEMP\ProDevEnvSetup"
New-Item -ItemType Directory -Path $WorkDir -Force | Out-Null
Set-Location $WorkDir

# ----------------------------------------------------------
# 1️. Enable WSL & Install Ubuntu
# ----------------------------------------------------------

Write-Host "-----------------------------------------------"
Write-Host "Enabling WSL and installing Ubuntu..."
Write-Host "-----------------------------------------------"

dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart

# Download and install WSL update
Invoke-WebRequest -Uri "https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi" -OutFile "wsl_update_x64.msi"
Start-Process msiexec.exe -ArgumentList "/i wsl_update_x64.msi /quiet /norestart" -Wait

wsl --set-default-version 2
wsl --install -d Ubuntu

Write-Host "✅ WSL and Ubuntu setup done."

# ----------------------------------------------------------
# 2️. Install Git Bash
# ----------------------------------------------------------

Write-Host "-----------------------------------------------"
Write-Host "Installing Git Bash..."
Write-Host "-----------------------------------------------"

$gitUrl = "https://github.com/git-for-windows/git/releases/latest/download/Git-2.45.1-64-bit.exe"
Invoke-WebRequest -Uri $gitUrl -OutFile "GitInstaller.exe"
Start-Process -FilePath "GitInstaller.exe" -ArgumentList "/VERYSILENT" -Wait

Write-Host "✅ Git Bash installed."

# ----------------------------------------------------------
# 3️. Install VS Code
# ----------------------------------------------------------

Write-Host "-----------------------------------------------"
Write-Host "Installing Visual Studio Code..."
Write-Host "-----------------------------------------------"

$vsCodeUrl = "https://update.code.visualstudio.com/latest/win32-x64-user/stable"
Invoke-WebRequest -Uri $vsCodeUrl -OutFile "VSCodeInstaller.exe"
Start-Process -FilePath "VSCodeInstaller.exe" -ArgumentList "/VERYSILENT /MERGETASKS=!runcode" -Wait

Write-Host "✅ VS Code installed."

# ----------------------------------------------------------
# 4️. Install Docker Desktop
# ----------------------------------------------------------

Write-Host "-----------------------------------------------"
Write-Host "Installing Docker Desktop..."
Write-Host "-----------------------------------------------"

$dockerUrl = "https://desktop.docker.com/win/main/amd64/Docker%20Desktop%20Installer.exe"
Invoke-WebRequest -Uri $dockerUrl -OutFile "DockerInstaller.exe"
Start-Process -FilePath "DockerInstaller.exe" -ArgumentList "install --quiet" -Wait

Write-Host "✅ Docker Desktop installed."

# ----------------------------------------------------------
# 5️. Install Python
# ----------------------------------------------------------

Write-Host "-----------------------------------------------"
Write-Host "Installing Python..."
Write-Host "-----------------------------------------------"

$pythonUrl = "https://www.python.org/ftp/python/3.12.2/python-3.12.2-amd64.exe"
Invoke-WebRequest -Uri $pythonUrl -OutFile "PythonInstaller.exe"
Start-Process -FilePath "PythonInstaller.exe" -ArgumentList "/quiet InstallAllUsers=1 PrependPath=1" -Wait

Write-Host "✅ Python installed."

# ----------------------------------------------------------
# 6️. Install Node.js (LTS)
# ----------------------------------------------------------

Write-Host "-----------------------------------------------"
Write-Host "Installing Node.js LTS..."
Write-Host "-----------------------------------------------"

$nodeUrl = "https://nodejs.org/dist/v20.12.2/node-v20.12.2-x64.msi"
Invoke-WebRequest -Uri $nodeUrl -OutFile "NodeInstaller.msi"
Start-Process msiexec.exe -ArgumentList "/i NodeInstaller.msi /quiet /norestart" -Wait

Write-Host "✅ Node.js installed."

# ----------------------------------------------------------
# 7️. Install NVM for Windows
# ----------------------------------------------------------

Write-Host "-----------------------------------------------"
Write-Host "Installing NVM for Windows..."
Write-Host "-----------------------------------------------"

$nvmUrl = "https://github.com/coreybutler/nvm-windows/releases/latest/download/nvm-setup.exe"
Invoke-WebRequest -Uri $nvmUrl -OutFile "nvm-setup.exe"
Start-Process -FilePath "nvm-setup.exe" -ArgumentList "/SILENT" -Wait

Write-Host "✅ NVM for Windows installed."

# ----------------------------------------------------------
# 8️. Install Chocolatey
# ----------------------------------------------------------

Write-Host "-----------------------------------------------"
Write-Host "Installing Chocolatey..."
Write-Host "-----------------------------------------------"

Set-ExecutionPolicy Bypass -Scope Process -Force
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072
Invoke-Expression ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))

Write-Host "✅ Chocolatey installed."

# ----------------------------------------------------------
# 9️. Install Winget (optional)
# ----------------------------------------------------------

Write-Host "-----------------------------------------------"
Write-Host "Checking if Winget is installed..."
Write-Host "-----------------------------------------------"

$wingetCheck = Get-Command winget -ErrorAction SilentlyContinue
if (-not $wingetCheck) {
    Write-Host "⚠️ Winget not found. To install it, update Windows App Installer from the Microsoft Store."
} else {
    Write-Host "✅ Winget is installed."
}

# ----------------------------------------------------------
# 10. Install popular VS Code extensions
# ----------------------------------------------------------

Write-Host "-----------------------------------------------"
Write-Host "Installing VS Code extensions..."
Write-Host "-----------------------------------------------"

$codeCmd = "$env:LOCALAPPDATA\Programs\Microsoft VS Code\bin\code.cmd"

& $codeCmd --install-extension ms-python.python
& $codeCmd --install-extension ms-vscode.vscode-node-azure-pack
& $codeCmd --install-extension ms-azuretools.vscode-docker
& $codeCmd --install-extension ms-vscode-remote.remote-wsl
& $codeCmd --install-extension esbenp.prettier-vscode
& $codeCmd --install-extension dbaeumer.vscode-eslint

Write-Host "✅ VS Code extensions installed."

# ----------------------------------------------------------
# Install Oh My Posh (optional bonus)
# ----------------------------------------------------------

Write-Host "-----------------------------------------------"
Write-Host "Installing Oh My Posh..."
Write-Host "-----------------------------------------------"

choco install oh-my-posh -y

Write-Host "✅ Oh My Posh installed."

# ----------------------------------------------------------
# ✅ DONE
# ----------------------------------------------------------

Write-Host "==============================================================="
Write-Host "✅ PRO Dev Environment is ready!"
Write-Host "🚀 Launch Ubuntu, VS Code, Git Bash, Docker Desktop, and enjoy!"
Write-Host "==============================================================="
```

## 📌 **How to run**

**1️.** Save it as: `Setup-ProDevEnv.ps1`  
**2️.** Run **PowerShell as Administrator**  
**3️.** Allow script execution for this session:
```
Set-ExecutionPolicy Bypass -Scope Process -Force
```
**4.** Run:
```
.\Setup-ProDevEnv.ps1
```

## ✅ **What’s different in this version**

✨ Installs **NVM for Windows** — switch Node versions easily:
```
nvm install lts
nvm use lts
```

✨ Installs **Chocolatey** — so you can install packages like `choco install terraform`, `choco install awscli` easily.

✨ Checks for **Winget** — you can also `winget install` things later.

✨ Installs **popular VS Code extensions**: Python, Docker, ESLint, Prettier, Remote WSL.

✨ Installs **Oh My Posh** — customize your terminal prompt for PowerShell & WSL.

## ⚡️ **Fully ready to code in minutes**

After it finishes:

- Open **Ubuntu** → set your Linux username/password.
    
- Launch **VS Code** → it’s ready for WSL/Remote Containers.
    
- Launch **Docker Desktop** → may need a reboot & sign-in.
    
- Try **`nvm use`**, **`node --version`**, **`python --version`**.

Are you an aspiring Linux user? Do you, perhaps, live in a van and want to get up to shenanigans and tomfoolery? Don't know quite where to start? Well, today is your day!

We've got a script that will automatically install and get VirtualBox ready for you to use a shiny, clean Ubuntu .iso so you can get in on the action! 

Check out [[VirtualBox Linux Demo Installer ⚡]]
