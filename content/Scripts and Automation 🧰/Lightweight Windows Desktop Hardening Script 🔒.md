If you are using a machine that is particularly high-risk (management, accounting/billing, etc), you might be understandably concerned about an attack on your system that could expose valuable data.

In the security field, it is crucial that we simply understand cyberattacks as an inevitability. Security professionals must assume that all machines will eventually be breached in order to take practical actions to mitigate the potential for damage after a machine or server has been compromised. 

However, the goal of this script is NOT to make our machines so hard to navigate that our end users are unable to efficiently work on their machines.

With that being said, here are some features for the script that is contained by this section:

✅ **Safe for everyday users**  
✅ Doesn’t break networking, remote management, or backup agents  
✅ Uses only built-in Windows features — no 3rd party install needed  
✅ Applies common CIS / NIST style baseline tweaks — without going “military lockdown”

### ✅ **PowerShell Script: `Basic-Windows-Hardening.ps1`**

```
# Basic-Windows-Hardening.ps1
# Sensible hardening for a Windows desktop/laptop
# Safe for EDR/RMM/Backup agents (like N-Able, Datto, Barracuda)

Write-Host "==========================================="
Write-Host "Running Basic Windows Desktop Hardening..."
Write-Host "==========================================="

# Ensure admin
if (-NOT ([Security.Principal.WindowsPrincipal] `
    [Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole(`
    [Security.Principal.WindowsBuiltInRole]::Administrator)) {
    Write-Warning "❌ You must run this script as Administrator!"
    exit 1
}

# -------------------------------------------------------------
# 1️. Windows Updates: Force to auto-install critical updates
# -------------------------------------------------------------

Write-Host "Enabling automatic updates..."

Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\WindowsUpdate\Auto Update" `
  -Name "AUOptions" -Value 4

Write-Host "✅ Automatic updates set to download & install automatically."

# -------------------------------------------------------------
# 2️. Enable Windows Firewall for all profiles
# -------------------------------------------------------------

Write-Host "Enabling Windows Defender Firewall for all profiles..."

Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled True

Write-Host "✅ Firewall is enabled."

# -------------------------------------------------------------
# 3️. Block all inbound connections except allowed
# -------------------------------------------------------------

Write-Host "Blocking all inbound connections except allowed rules..."

Set-NetFirewallProfile -Profile Domain,Public,Private -DefaultInboundAction Block -DefaultOutboundAction Allow

Write-Host "✅ Inbound connections are restricted to allowed rules only."

# -------------------------------------------------------------
# 4️. Enable SmartScreen for Edge and Windows
# -------------------------------------------------------------

Write-Host "Enabling SmartScreen..."

Set-MpPreference -PUAProtection Enabled

New-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer" -Name "SmartScreenEnabled" -PropertyType String -Value "RequireAdmin" -Force | Out-Null

Write-Host "✅ SmartScreen is on."

# -------------------------------------------------------------
# 5️. Enable BitLocker (prompt only)
# -------------------------------------------------------------

Write-Host "Checking BitLocker status..."

$drives = Get-BitLockerVolume | Where-Object {$_.VolumeType -eq 'OperatingSystem'}
foreach ($drive in $drives) {
    if ($drive.ProtectionStatus -eq 'Off') {
        Write-Host "⚠️ BitLocker is OFF on drive $($drive.MountPoint). Please enable it manually if supported."
    } else {
        Write-Host "✅ BitLocker is already ON for $($drive.MountPoint)."
    }
}

# -------------------------------------------------------------
# 6️. Enable Windows Defender Real-Time Protection
# -------------------------------------------------------------

Write-Host "Enabling Windows Defender Real-Time Protection..."

Set-MpPreference -DisableRealtimeMonitoring $false

Write-Host "✅ Real-Time Protection is enabled."

# -------------------------------------------------------------
# 7️. Disable AutoRun for all drives
# -------------------------------------------------------------

Write-Host "Disabling AutoRun for all drives..."

New-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer" `
  -Name "NoDriveTypeAutoRun" -PropertyType DWord -Value 255 -Force | Out-Null

Write-Host "✅ AutoRun disabled."

# -------------------------------------------------------------
# 8️. Disable Remote Assistance (but NOT Remote Desktop)
# -------------------------------------------------------------

Write-Host "Disabling Remote Assistance..."

Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Remote Assistance" `
  -Name "fAllowToGetHelp" -Value 0

Write-Host "✅ Remote Assistance disabled. Remote Desktop untouched."

# -------------------------------------------------------------
# 9️. Enable User Account Control (UAC) at highest level
# -------------------------------------------------------------

Write-Host "Enforcing strong User Account Control (UAC)..."

Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" `
  -Name "EnableLUA" -Value 1

Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" `
  -Name "ConsentPromptBehaviorAdmin" -Value 2  # Always notify

Write-Host "✅ UAC set to highest level."

# -------------------------------------------------------------
# 10. Disable SMBv1 (safe for modern Windows)
# -------------------------------------------------------------

Write-Host "Disabling SMBv1 protocol..."

Disable-WindowsOptionalFeature -Online -FeatureName "SMB1Protocol" -NoRestart -ErrorAction SilentlyContinue

Write-Host "✅ SMBv1 disabled if it was installed."

# -------------------------------------------------------------
# ✅ DONE
# -------------------------------------------------------------

Write-Host "==========================================="
Write-Host "✅ Basic Windows hardening applied."
Write-Host "💡 You may need to restart for some changes."
Write-Host "==========================================="
```

## ✅ **What this script does**

**Good for most orgs:**

- Automatic critical updates
    
- Windows Defender Firewall ON for all profiles
    
- Block inbound except allowed
    
- SmartScreen ON (browser + OS level)
    
- AutoRun disabled (common USB threat vector)
    
- Remote Assistance OFF (prevents social engineering help scams)
    
- UAC at highest level (always prompt for elevation)
    
- SMBv1 disabled (old attack vector)
    
- Leaves **Remote Desktop** untouched (safe for RMM/EDR)
    
- Checks BitLocker status (doesn’t force it in case hardware doesn’t support it)

## 📌 **Safe for EDR / RMM / Backup**

✅ Won’t break:

- N-Able EDR agent
    
- N-Able RMM agent
    
- Datto or Barracuda backup agent
    
- Remote control or scripting tools
    

✅ Does **not** block outbound connections  
✅ Does **not** restrict common remote ports or management protocols  
✅ Keeps Remote Desktop untouched (only disables **Remote Assistance**)

## ⚡️ **How to run**

1️⃣ Save as `Basic-Windows-Hardening.ps1`  
2️⃣ Open PowerShell **as Administrator**  
3️⃣ Temporarily allow script execution:

`Set-ExecutionPolicy Bypass -Scope Process -Force`

4️⃣ Run:

`.\Basic-Windows-Hardening.ps1`

### **⚠️ IMPORTANT NOTE! ⚠️**

This script assumes that the machine being worked on is NOT joined to a domain. If that is the case, we can actually take similar actions simply by creating a safe, practical GPO baseline. 

To access that baseline, check out the GPO version of this task that allows us to set the same rules without configuring individual machines below:

[[GPO Windows Desktop Hardening 💻]]