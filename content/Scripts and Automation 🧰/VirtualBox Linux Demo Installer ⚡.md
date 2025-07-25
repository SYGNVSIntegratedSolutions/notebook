(From Paul) This is probably my favorite script in this suite of automation tools. With this script, we can:
✅ Download & install **VirtualBox** (latest Windows version)  
✅ Download the **latest Ubuntu Desktop ISO**  
✅ Download the **latest Arch Linux ISO**  
✅ _Optionally_ set up **VirtualBox VMs** for each, with some basic settings (RAM, CPUs, disk).

⚡️ **Important note:**  
PowerShell can fully script the **download and VirtualBox install**, but _VirtualBox VM setup_ needs the **VirtualBox command-line tool** (`VBoxManage`). So this script will **automate**:

- Install VirtualBox
    
- Download ISOs
    
- Create VM folders
    
- Use `VBoxManage` to:
    
    - Create VMs
        
    - Attach ISOs
        
    - Configure RAM/CPUs/Storage
        

You’ll just need to start the VM and complete the OS install manually in the VM console. The machines themselves will be automatically set up with some default configurations - you're welcome to tweak those settings as needed for your workflow or virtualization needs!

### ✅ **PowerShell Script: `Setup-VirtualBox-VMs.ps1`**

```
# Setup-VirtualBox-VMs.ps1

Write-Host "==========================================="
Write-Host "VirtualBox + Ubuntu & Arch VMs Setup Script"
Write-Host "==========================================="

# Ensure admin
if (-NOT ([Security.Principal.WindowsPrincipal] `
    [Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole(`
    [Security.Principal.WindowsBuiltInRole]::Administrator)) {
    Write-Warning "❌ You must run this script as Administrator!"
    exit 1
}

$WorkDir = "$env:TEMP\VirtualBoxSetup"
New-Item -ItemType Directory -Path $WorkDir -Force | Out-Null
Set-Location $WorkDir

# ---------------------------------------
# 1️. Download & install VirtualBox
# ---------------------------------------

Write-Host "-------------------------------------------"
Write-Host "Downloading and installing VirtualBox..."
Write-Host "-------------------------------------------"

# Latest VirtualBox 7.x for Windows hosts
$vbUrl = "https://download.virtualbox.org/virtualbox/7.0.18/VirtualBox-7.0.18-162988-Win.exe"
$vbInstaller = "VirtualBox.exe"

Invoke-WebRequest -Uri $vbUrl -OutFile $vbInstaller

Start-Process -FilePath $vbInstaller -ArgumentList "--silent" -Wait

Write-Host "✅ VirtualBox installed."

# ---------------------------------------
# 2️. Download Ubuntu Desktop ISO
# ---------------------------------------

Write-Host "-------------------------------------------"
Write-Host "Downloading Ubuntu Desktop ISO..."
Write-Host "-------------------------------------------"

$ubuntuUrl = "https://releases.ubuntu.com/24.04/ubuntu-24.04-desktop-amd64.iso"
$ubuntuIso = "$WorkDir\ubuntu-24.04-desktop-amd64.iso"

Invoke-WebRequest -Uri $ubuntuUrl -OutFile $ubuntuIso

Write-Host "✅ Ubuntu ISO downloaded: $ubuntuIso"

# ---------------------------------------
# 3️. Download Arch Linux ISO
# ---------------------------------------

Write-Host "-------------------------------------------"
Write-Host "Downloading Arch Linux ISO..."
Write-Host "-------------------------------------------"

$archUrl = "https://mirror.rackspace.com/archlinux/iso/latest/archlinux-x86_64.iso"
$archIso = "$WorkDir\archlinux-x86_64.iso"

Invoke-WebRequest -Uri $archUrl -OutFile $archIso

Write-Host "✅ Arch Linux ISO downloaded: $archIso"

# ---------------------------------------
# 4️. Create VirtualBox VMs using VBoxManage
# ---------------------------------------

Write-Host "-------------------------------------------"
Write-Host "Creating VirtualBox VMs..."
Write-Host "-------------------------------------------"

# VBoxManage path (should be in PATH, adjust if needed)
$VBoxManage = "${env:ProgramFiles}\Oracle\VirtualBox\VBoxManage.exe"

# Ubuntu VM
& $VBoxManage createvm --name "UbuntuVM" --ostype "Ubuntu_64" --register
& $VBoxManage modifyvm "UbuntuVM" --memory 4096 --cpus 2 --vram 128 --nic1 nat
& $VBoxManage createhd --filename "$env:USERPROFILE\VirtualBox VMs\UbuntuVM\UbuntuVM.vdi" --size 30000
& $VBoxManage storagectl "UbuntuVM" --name "SATA Controller" --add sata --controller IntelAhci
& $VBoxManage storageattach "UbuntuVM" --storagectl "SATA Controller" --port 0 --device 0 --type hdd --medium "$env:USERPROFILE\VirtualBox VMs\UbuntuVM\UbuntuVM.vdi"
& $VBoxManage storagectl "UbuntuVM" --name "IDE Controller" --add ide
& $VBoxManage storageattach "UbuntuVM" --storagectl "IDE Controller" --port 0 --device 0 --type dvddrive --medium $ubuntuIso

# Arch VM
& $VBoxManage createvm --name "ArchVM" --ostype "ArchLinux_64" --register
& $VBoxManage modifyvm "ArchVM" --memory 2048 --cpus 2 --vram 64 --nic1 nat
& $VBoxManage createhd --filename "$env:USERPROFILE\VirtualBox VMs\ArchVM\ArchVM.vdi" --size 20000
& $VBoxManage storagectl "ArchVM" --name "SATA Controller" --add sata --controller IntelAhci
& $VBoxManage storageattach "ArchVM" --storagectl "SATA Controller" --port 0 --device 0 --type hdd --medium "$env:USERPROFILE\VirtualBox VMs\ArchVM\ArchVM.vdi"
& $VBoxManage storagectl "ArchVM" --name "IDE Controller" --add ide
& $VBoxManage storageattach "ArchVM" --storagectl "IDE Controller" --port 0 --device 0 --type dvddrive --medium $archIso

Write-Host "✅ VMs created! UbuntuVM and ArchVM are ready."

Write-Host "-------------------------------------------"
Write-Host "To start installing:"
Write-Host "👉 Open VirtualBox, select the VM, click Start."
Write-Host "👉 Install Ubuntu or Arch as normal in the VM."
Write-Host "-------------------------------------------"
```

## 📌 **How to run**

1️. Save it as `Setup-VirtualBox-VMs.ps1`  
2️. Run **PowerShell as Administrator**  
3️. Temporarily allow scripts:

`Set-ExecutionPolicy Bypass -Scope Process -Force`

4️. Run:

`.\Setup-VirtualBox-VMs.ps1`

---

## ✅ **What this does**

✅ Installs **VirtualBox 7.0.x**  
✅ Downloads **Ubuntu 24.04 Desktop ISO**  
✅ Downloads latest **Arch Linux ISO**  
✅ Uses `VBoxManage` to:

- Create `UbuntuVM` & `ArchVM`
    
- Attach ISO as CD drive
    
- Allocate RAM/CPUs/Storage  
    ✅ Leaves the VM ready to boot → you do the OS install via the VirtualBox console.
    

---

## ⚠️ **Notes**

- `VBoxManage` **must be in PATH** — the standard installer does this automatically.
    
- Adjust RAM/CPUs/Disk sizes to suit your hardware.
    
- You can tweak `ostype` for more tuning (see `VBoxManage list ostypes`).
    
- I would strongly advise users to use the "Insert" option at the top to insert the Vbox Guest Additions removable media image in order to enhance functionality
