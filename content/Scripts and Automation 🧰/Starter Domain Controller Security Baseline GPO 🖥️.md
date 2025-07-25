
Like the other GPO-based recommendations, we have this document available so our clients are able to access it. However, because this gets into a pretty technical area, we are happy to help implement any of the changes that are outlined here! Simply email helpdesk@sygnvs.com and a technician will get started on making sure your server images are locked tight!
### 📌 **1️. Account Policies**

These are **domain-level**, so configure them in the **Default Domain Policy** or a dedicated GPO linked at the domain root:

|Setting|Recommended|GPMC Path|
|---|---|---|
|Enforce password history|24 passwords remembered|Computer Config → Policies → Windows Settings → Security Settings → Account Policies → Password Policy|
|Maximum password age|60 days|Same|
|Minimum password age|1 day|Same|
|Minimum password length|12 characters|Same|
|Password must meet complexity requirements|Enabled|Same|
|Account lockout threshold|5 invalid logon attempts|Computer Config → Policies → Windows Settings → Security Settings → Account Policies → Account Lockout Policy|
|Account lockout duration|15 minutes|Same|
|Reset account lockout counter|15 minutes|Same|

---

### 📌 **2️. Local Policies: Audit Policy**

Set strong auditing:

|Setting|Recommended|GPMC Path|
|---|---|---|
|Audit account logon events|Success, Failure|Computer Config → Policies → Windows Settings → Security Settings → Local Policies → Audit Policy|
|Audit logon events|Success, Failure|Same|
|Audit account management|Success, Failure|Same|
|Audit directory service access|Success, Failure|Same|
|Audit policy change|Success, Failure|Same|
|Audit privilege use|Failure|Same|
|Audit object access|Success, Failure|Same|
|Audit process tracking|Success|Same|
|Audit system events|Success, Failure|Same|

---

### 📌 **3️. User Rights Assignment**

Key rights to restrict:

|Setting|Recommended|GPMC Path|
|---|---|---|
|Deny access to this computer from the network|Guests|Computer Config → Policies → Windows Settings → Security Settings → Local Policies → User Rights Assignment|
|Deny log on locally|Guests|Same|
|Deny log on through Remote Desktop Services|Guests|Same|
|Access this computer from the network|Administrators, Authenticated Users|Same|
|Allow log on locally|Administrators|Same|
|Allow log on through Remote Desktop Services|Administrators|Same|

---

### 📌 **4️. Security Options**

Practical defaults for DCs:

|Setting|Recommended|GPMC Path|
|---|---|---|
|Accounts: Administrator account status|Enabled|Computer Config → Policies → Windows Settings → Security Settings → Local Policies → Security Options|
|Accounts: Guest account status|Disabled|Same|
|Accounts: Rename administrator account|e.g., change `Administrator` to something unique|Same|
|Network security: LAN Manager authentication level|Send NTLMv2 response only. Refuse LM & NTLM|Same|
|Network security: Do not store LAN Manager hash value|Enabled|Same|
|User Account Control: Admin Approval Mode for the built-in Administrator account|Enabled|Same|
|User Account Control: Run all admins in Admin Approval Mode|Enabled|Same|
|User Account Control: Behavior of elevation prompt for admins|Prompt for consent on secure desktop|Same|

---

### 📌 **5️. Windows Defender & Firewall**

Most orgs run EDR. If you **do** rely on Defender AV:

|Setting|Recommended|GPMC Path|
|---|---|---|
|Turn on real-time protection|Enabled|Computer Config → Policies → Admin Templates → Windows Components → Microsoft Defender Antivirus → Real-time Protection|
|Configure Windows Defender Firewall: Domain Profile|Enabled|Computer Config → Policies → Windows Settings → Security Settings → Windows Defender Firewall|

---

### 📌 **6️. SMB & Legacy Protocols**

|Setting|Recommended|GPMC Path|
|---|---|---|
|Microsoft network client: Digitally sign communications (always)|Enabled|Computer Config → Policies → Windows Settings → Security Settings → Local Policies → Security Options|
|Microsoft network server: Digitally sign communications (always)|Enabled|Same|
|Microsoft network server: Disconnect clients when logon hours expire|Enabled|Same|

---

### 📌 **7️. Additional: Event Log**

|Setting|Recommended|GPMC Path|
|---|---|---|
|Maximum application log size|32,768 KB|Computer Config → Policies → Windows Settings → Security Settings → Event Log|
|Maximum security log size|196,608 KB (or more)|Same|
|Maximum system log size|32,768 KB|Same|
|Retain security log|Enabled|Same|
|Shut down system immediately if unable to log security audits|Enabled|Same|

---

## ✅ **Linking and Best Practices**

✔️ Create a **new GPO** called `Domain Controller Security Baseline`  
✔️ Link it to the **Domain Controllers OU**  
✔️ Never link DC-specific GPOs to the entire domain  
✔️ Keep **Account Policies** in the **Default Domain Policy** or a single dedicated GPO **linked at the domain root**  
✔️ Apply **advanced auditing** with caution — test logging size & storage

---

## ⚡ **Optional CIS extras**

For **Level 1 +**, add:

- NTLM auditing to log deprecated auth
    
- Restrict anonymous access
    
- Enforce Sysmon or Forwarded Event Log server