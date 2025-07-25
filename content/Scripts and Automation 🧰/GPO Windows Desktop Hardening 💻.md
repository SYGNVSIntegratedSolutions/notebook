
If you're here, that means that you want to harden some desktops and laptops by using Group Policy Objects instead of scripting on each workstation. 

There's a lot to go over here, so we will keep the intro nice and short and let you get right into it. If you would rather one of our technicians handle this for you, then we're always happy to help! Simply email helpdesk@sygnvs.com to create a ticket, and one of our amazing technicians will jump on it right away!

## ✅ **1️. Recommended GPO Settings**

Here’s the **same security baseline**, but mapped to real Group Policy settings you can configure in the **Group Policy Management Console (GPMC)**.

|**Goal**|**GPO Setting**|**Where to find it**|
|---|---|---|
|**Enable Windows Updates**|`Configure Automatic Updates` → Enabled → Auto download and schedule install|Computer Config → Policies → Admin Templates → Windows Components → Windows Update|
|**Enable Windows Firewall**|`Windows Defender Firewall: Protect all network connections` → Enabled|Computer Config → Policies → Windows Settings → Security Settings → Windows Defender Firewall|
|**Block all inbound except allowed**|`Windows Defender Firewall: Inbound connections` → Block (Default)|Same as above|
|**Enable SmartScreen**|`Configure Windows Defender SmartScreen` → Enabled → Require approval from admin|Computer Config → Policies → Admin Templates → Windows Components → File Explorer|
|**BitLocker**|`Require additional authentication at startup` → Enabled (TPM + PIN optional)|Computer Config → Policies → Admin Templates → Windows Components → BitLocker Drive Encryption → Operating System Drives|
|**Defender Real-Time Protection**|`Turn off real-time protection` → Disabled|Computer Config → Policies → Admin Templates → Windows Components → Microsoft Defender Antivirus → Real-time Protection|
|**Disable AutoRun**|`Turn off AutoPlay` → Enabled for all drives|Computer Config → Policies → Admin Templates → Windows Components → AutoPlay Policies|
|**Remote Assistance**|`Configure Offer Remote Assistance` → Disabled|Computer Config → Policies → Admin Templates → System → Remote Assistance|
|**UAC to max**|`User Account Control: Behavior of the elevation prompt for administrators` → `Prompt for consent for non-Windows binaries`|Computer Config → Policies → Windows Settings → Security Settings → Local Policies → Security Options|
|**SMBv1**|`Enable insecure guest logons` → Disabled (and SMBv1 feature removed manually or scripted)|Computer Config → Policies → Admin Templates → Network → Lanman Workstation|
## ✅ **2️. Example GPO Walkthrough**

**In GPMC:**

1️. **Create a new GPO**  
→ Right-click your domain or OU → `Create a GPO in this domain` → Name it **“Desktop Security Baseline”**

2️. **Edit the GPO**  
→ Right-click → `Edit` → Open `Computer Configuration` → set each policy as above.

3️. **Deploy BitLocker** carefully!  
→ If you don’t want to break older hardware, set **BitLocker policies** to **allow TPM-only or TPM+PIN** and deploy in phases.  
→ Or **just enforce the setting but don’t auto-encrypt**, and prompt IT to enable manually.

4️. **Link the GPO**  
→ Link it to your **Laptops / Workstations OU** — **not** Domain Controllers or Servers!

5️. **Test** on a **pilot group**  
→ Apply to a test OU, run `gpupdate /force`, check settings with `rsop.msc`.

---

## ✅ **3️. Optional: Additional best practice tweaks**

If you want to go slightly further without user friction:

- **Disable Windows Store (optional)**: Helps avoid unvetted app installs.
    
    - `Turn off the Store application` → Enabled
        
- **Disable Cortana (optional)**:
    
    - `Allow Cortana` → Disabled
        
- **Enforce password complexity & length**:
    
    - Local Policies → Security Options → Password Policy
        

---

## ✅ **4️. SMBv1 removal**

**Note:** SMBv1 is best removed by feature uninstall, not GPO alone.  
You can use a **startup script** in the same GPO:

**Script:**

powershell

CopyEdit

`Disable-WindowsOptionalFeature -Online -FeatureName SMB1Protocol -NoRestart`

Put it under:  
`Computer Config → Policies → Windows Settings → Scripts (Startup/Shutdown) → Startup`

---

## ✅ **5️. Exclusions for RMM/Backup**

No settings above block your:

- N-Able RMM/EDR agent
    
- Datto or Barracuda backup agents  
    **Because**:
    
- We’re not disabling Remote Desktop (RDP)
    
- We’re not restricting outbound connections
    
- We’re not disabling standard admin shares
    
- Windows Firewall default rules allow management agents to function
    

---

## ✅ **6️. Documentation tip**

**Document this baseline** in your MSP’s client security policies:

- Name: `Standard Workstation Baseline`
    
- Scope: `All domain-joined laptops/desktops`
    
- Exceptions: `Test lab machines, specialized hardware, servers`
    