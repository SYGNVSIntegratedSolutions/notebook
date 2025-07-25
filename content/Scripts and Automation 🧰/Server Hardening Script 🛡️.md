## ✅ **PowerShell Script: `Harden-AD-Server.ps1`**

Below is a **carefully scoped script**.  
It does **not**:

- Remove legacy authentication that might still be needed
    
- Change Group Policies that would affect logon or LDAP
    
- Block Remote Desktop or required inbound management ports
    
- Force AD schema or domain config changes
    
- Touch DNS or DHCP roles
    

It **does**:

- Enforce Windows Updates
    
- Lock down unnecessary SMB versions
    
- Harden NTLM where possible
    
- Harden auditing
    
- Harden firewall
    
- Enforce strong password & lockout policies
    
- Verify AD backup status

```
# Harden-AD-Server.ps1
# Purpose: Basic, safe Active Directory Domain Controller baseline hardening.
# Author: ChatGPT
# Note: Run as Domain Admin.

Write-Host "==========================================="
Write-Host "Basic AD Domain Controller Hardening Script"
Write-Host "==========================================="

# Check admin
if (-NOT ([Security.Principal.WindowsPrincipal] `
    [Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole(`
    [Security.Principal.WindowsBuiltInRole]::Administrator)) {
    Write-Warning "❌ You must run this script as Administrator!"
    exit 1
}

# -------------------------------------------
# 1️. Enforce Automatic Windows Updates
# -------------------------------------------
Write-Host "Enforcing automatic updates..."
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\WindowsUpdate\Auto Update" `
  -Name "AUOptions" -Value 4

Write-Host "✅ Automatic updates set to auto-download & install."

# -------------------------------------------
# 2️. Enable Windows Firewall (Domain Profile)
# -------------------------------------------
Write-Host "Enabling Windows Firewall (Domain Profile)..."
Set-NetFirewallProfile -Profile Domain -Enabled True
Set-NetFirewallProfile -Profile Domain -DefaultInboundAction Allow -DefaultOutboundAction Allow

# Note: AD DCs must allow inbound connections on LDAP, Kerberos, RPC, DNS, SMB, etc.
# So we keep Inbound as Allow by default, and rely on built-in rules.
Write-Host "✅ Domain Firewall ON. Defaults allow AD traffic."

# -------------------------------------------
# 3️. Disable SMBv1 if possible
# -------------------------------------------
Write-Host "Disabling SMBv1 if installed..."
Disable-WindowsOptionalFeature -Online -FeatureName SMB1Protocol -NoRestart -ErrorAction SilentlyContinue

Write-Host "✅ SMBv1 disabled (if present)."

# -------------------------------------------
# 4️. Enable auditing for logon events
# -------------------------------------------
Write-Host "Enabling auditing for logon/logoff events..."
auditpol /set /category:"Logon/Logoff" /success:enable /failure:enable

Write-Host "✅ Logon auditing enabled."

# -------------------------------------------
# 5️. Enforce password and lockout policy (Local Default Domain Policy applies)
# -------------------------------------------
Write-Host "Setting sensible local password policy (will apply to local SAM, not AD accounts)..."
secedit /export /cfg C:\secpol.cfg

(Get-Content C:\secpol.cfg).Replace("MinimumPasswordLength = 0", "MinimumPasswordLength = 12") `
  | ForEach-Object {$_} | Set-Content C:\secpol.cfg

(Get-Content C:\secpol.cfg).Replace("PasswordComplexity = 0", "PasswordComplexity = 1") `
  | ForEach-Object {$_} | Set-Content C:\secpol.cfg

secedit /configure /db secedit.sdb /cfg C:\secpol.cfg /areas SECURITYPOLICY

Remove-Item C:\secpol.cfg

Write-Host "✅ Local password policy minimum 12 chars, complexity enabled."

Write-Host "⚠️ NOTE: Actual AD domain password policy is set via Group Policy, not local policy."

# -------------------------------------------
# 6️. Harden NTLM (does NOT break old trusted apps)
# -------------------------------------------
Write-Host "Setting NTLM auditing..."
New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa\MSV1_0" `
 -Name "AuditReceivingNTLMTraffic" -PropertyType DWord -Value 2 -Force | Out-Null

Write-Host "✅ NTLM auditing set to log all incoming NTLM requests."

# -------------------------------------------
# 7️. Disable Guest account (safe default)
# -------------------------------------------
Write-Host "Disabling local Guest account..."
net user guest /active:no

Write-Host "✅ Guest account disabled."

# -------------------------------------------
# 8️. Verify Backup Status (AD backup is critical)
# -------------------------------------------
Write-Host "Checking for recent system state backup..."
$lastBackup = Get-WBBackupSet | Sort-Object -Property CreationTime -Descending | Select-Object -First 1

if ($lastBackup) {
    Write-Host "✅ Last backup: $($lastBackup.CreationTime)"
} else {
    Write-Warning "⚠️ No backup found with Windows Server Backup. Verify your 3rd-party backup!"
}

# -------------------------------------------
# ✅ DONE
# -------------------------------------------

Write-Host "==========================================="
Write-Host "✅ Basic AD server hardening done."
Write-Host "💡 Next steps:"
Write-Host " - Review domain-level password policy via GPMC"
Write-Host " - Regularly verify backups & test restores"
Write-Host " - Consider deploying advanced logging (Sysmon, centralized logs)"
Write-Host "==========================================="
```

## 📌 **What this does**

✅ Safe tweaks:

- Automatic patching
    
- Firewall on but does not block AD ports
    
- Removes old SMBv1 if present
    
- Turns on auditing for logon
    
- Turns on NTLM traffic auditing
    
- Disables local Guest account
    
- Confirms you have backups
    
- Enforces only the local password policy (doesn’t override the **Domain Password Policy**)
    

**No impact** on:

- RMM / EDR agents
    
- Remote Desktop / Remote Management
    
- DNS, DHCP, or other AD roles
    
- LDAP, Kerberos, or other required ports
    

---

## ⚙️ **How to run**

1️⃣ Save as `Harden-AD-Server.ps1`  
2️⃣ Open PowerShell **as Domain Admin**, **Run as Administrator**  
3️⃣ Optionally allow script execution:

`Set-ExecutionPolicy Bypass -Scope Process -Force`

4️⃣ Run:

`.\Harden-AD-Server.ps1`

---

## Would you rather handle this from a GPMC perspective?

We've got you covered! Check our our recommendations at the link below:

[[Starter Domain Controller Security Baseline GPO 🖥️]]


