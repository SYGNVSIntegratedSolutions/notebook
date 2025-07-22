![[Gemini_Generated_Image_1a12s91a12s91a12.png]]

### **Current Day

Throughout Q2 2025, SCATTERED SPIDER's activities have significantly broadened in scope to include US-based airlines with the goal being the extraction of customer data containing information on millions of American and international passengers. While this represents a significant departure from the areas that the digital crime syndicate is known for, it also represents an industry-wide problem that airlines face concerning the retention of customer data. Because much of this data is not subject to the same regulations which govern the collection and storage of data by retailers and insurance corporations, it represents a critical point of compromise. 

The purpose of this article is to explore what exactly SS is, outline their essential methodology when conducting attacks, what their goals are, and what we can do to proactively prevent attacks which follow their TTPs.


![[Gemini_Generated_Image_t4n3jst4n3jst4n3.png]]

### **What is Scattered Spider?**

Also known within the cybersecurity industry as UNC3944, Scattered Spider is a hacking group composed mostly of teenagers and young adults believed to live in the United States and United Kingdom. The group gained substantial infamy in the early 2020's for massive breaches which led to the extortion of Caesars Entertainment and MGM Resorts International, which are two of the largest casino and general gambling corporations in the US (*Wiki*). 

Their primary attacks seem to consistently target the retail, financial, and insurance sectors of the American economy, with the goal being to ransom data back to the companies which are attacked. While they have expanded to include airlines as of this year, Scattered Spider has also targeted Visa, [Marks & Spencer](https://en.wikipedia.org/wiki/Marks_%26_Spencer "Marks & Spencer"), PNC Financial Services Group Inc., Transamerica, New York Life Insurance Co., [Synchrony Financial](https://en.wikipedia.org/wiki/Synchrony_Financial "Synchrony Financial"), [Truist Bank](https://en.wikipedia.org/wiki/Truist_Bank "Truist Bank"), and Twilio (*Wiki*). 

The group seems to have been born out of COVID and is suspected to have been formed in retaliation to the mass layoffs in the tech sector that occurred at around the same time. While the exact origins of the group remain a mystery, its activities since their formation have been exceptionally public for a group of its size. 

![[Gemini_Generated_Image_8upibf8upibf8upi.png]]

### **The Methodology of UNC3944's Attacks**

The following documentation has been sourced from [CrowdStrike Blog](https://www.crowdstrike.com/en-us/blog/crowdstrike-services-observes-scattered-spider-escalate-attacks/) contributors - check out their site, it is a fantastic resource for conducting open-source intelligence on active threats.

The adversary used help desk voice-based phishing in almost all observed 2025 incidents to compromise Microsoft Entra ID, single sign-on (SSO), and virtual desktop infrastructure (VDI) accounts. SCATTERED SPIDER operators routinely accurately respond to help desk verification questions when impersonating legitimate employees in calls made to request password and/or multifactor authentication (MFA) resets.

[SCATTERED SPIDER](https://www.crowdstrike.com/adversaries/scattered-spider/) typically pivots from compromised Entra ID, SSO, and VDI accounts to integrated software-as-a-service (SaaS) applications. They use access to these platforms to search for data that may enable lateral movement (such as network architecture diagrams, VPN instructions, or text files containing credentials), extortion, or other monetization activity.

Below are additional TTPs observed in recent SCATTERED SPIDER activity:

- Conducted Active Directory (AD) reconnaissance on on-premises systems using ADExplorer, ADRecon.ps1, and the Get-ADUser PowerShell (PS) cmdlet
- Used VMware vCenter access to create unmanaged virtual machines (VMs); the adversary often attaches domain controller virtual machine disks to their unmanaged VMs, then dumps the AD database ntds.dit on these systems
- Installed legitimate protocol-tunneling and proxy tools on VMware vCenter and adversary-controlled VMs, including Chisel (configured to communicate with trycloudflare[.]com subdomains), MobaXterm, ngrok, Pinggy, Rsocx, and Teleport
- Manually deleted (i.e., HardDelete, SoftDelete, and MoveToDeletedItems operations) and created transport rules (Set-TransportRule) to delete or redirect emails notifying users of suspicious account activity — in one case, the adversary created a mail transport rule to redirect emails intended for a compromised user to a likely adversary-controlled email address with the googlemail[.]com domain
- Used S3 Browser to enumerate victims’ Amazon Web Services (AWS) S3 buckets (AWS CloudTrail events: ListBuckets and ListObjects) and exfiltrate data to remote adversary-controlled S3 buckets

![[Gemini_Generated_Image_e0fj6pe0fj6pe0fj(1).png]]

Common attack methods for this adversary include: 

- **Social Engineering**: Targeting IT help desk and privileged users through sophisticated phone-based attacks and impersonation
- **SIM Swapping and Phone-Based Credential Theft**: Compromising mobile phone accounts to bypass SMS-based MFA
- **Abuse of Legitimate Remote Access Tools**: Leveraging TeamViewer, AnyDesk, and similar tools for persistent access
- **VMware Infrastructure Compromise**: Targeting vCenter and ESXi environments for ransomware deployment
- **Cloud Environment Lateral Movement**: Exploiting cloud identity providers and moving laterally through cloud resources
- **Data Exfiltration**: Stealing sensitive data before deploying ransomware for double extortion

Common targets include: 

- VMware vCenter and ESXi virtualization environments
- Cloud identity providers (Azure AD/Entra ID, AWS IAM, Google Cloud Identity, Okta)
- Privileged access management systems and administrative accounts
- VPN and remote access solutions
- Backup and recovery systems
- Help desk and IT support personnel

### **Recommendations from CrowdStrike to Counter Scattered Spider's Primary Attack Vectors**

![[Gemini_Generated_Image_uofp6juofp6juofp.png]]

While SS's activities in the US have allowed for some of the most substantial data breaches and successful extortions in history, their primary methods of attack are well documented. It is for this reason that cybersecurity professionals always recommend the enforcement of robust documentation policies being implemented by clients in their purview. 

With that being said, what follows is the highest-priority steps we can all take to prevent ourselves from becoming UNC3944's next victim:

### Infrastructure Monitoring (Highest Priority)

- **VMware vCenter and ESXi**: Essential for detecting virtual infrastructure manipulation and unauthorized access
- **Firewall Logs**: Critical for identifying network-based attack patterns and lateral movement
- **DNS Logs**: Vital for detecting command-and-control communications and data exfiltration attempts
- **Web Proxy Logs**: Monitor for suspicious web traffic and potential data exfiltration

### Identity and Authentication Systems

- **SSO Platforms**: Track authentication anomalies and suspicious login patterns
- **Entra ID Sign-on and Audit Logs**: Monitor for identity-based attacks and privilege escalation attempts
- **PAM Applications**: Detect unauthorized privileged access and credential misuse

### Cloud and SaaS Applications

- **AWS CloudTrail, Google Cloud, Azure Activity Logs**: Monitor cloud resource manipulation and configuration changes
- **Critical SaaS Applications**: Monitor SaaS applications for application-level threat detection

### Deploy Critical Correlation Rule Templates 

Correlation rule templates (CRTs) are critical to increase monitoring and detection posture. Once logs have been ingested into Falcon Next-Gen SIEM, the following CRTs will help detect anomalous behavior. 

### VMware Infrastructure Protection

Essential rules for virtual environment security:

1. VMware - vCenter - Virtual Machine Created with Recently Uploaded ISO
2. VMware - vCenter - Sensitive Resource Search
3. VMware - ESXi - Successful Login to the ESXi Host Client Web Administration Interface
4. VMware - ESXi - New IP for SSH Login Detected
5. VMware - ESXi - SFTP Server Enabled

### Entra ID Identity Protection

Critical Identity Security Rules:

1. Microsoft - Entra ID - Risky Sign-in
2. Microsoft - Entra ID - Admin Deleted MFA Authentication Method
3. Microsoft - Entra ID - Bulk Download User List
4. Microsoft - Entra ID - Temporary Access Pass Added to User Account
5. Microsoft - Entra ID - Global Administrator Role Assigned
### Core SaaS Applications

- **Microsoft 365 Suite**: Exchange, SharePoint, OneDrive, Teams for comprehensive cloud application monitoring
- **Microsoft Defender**: Enhanced integration for security event correlation
- **Google Workspace**: Complete visibility into Google Cloud activities
### Security Platform Integration

- **Enhanced Falcon Integration**: Maximize native CrowdStrike detection capabilities
- **Zscaler Cloud Security**: Monitor secure web gateway and cloud access security broker activities
- **CyberArk PAM**: Comprehensive privileged access monitoring and threat detection
### Business-Critical Applications

- **Snowflake Data Platform**: Monitor for unauthorized data access and exfiltration attempts
- **Workday HR Systems**: Detect suspicious employee data access and modifications
- **GitHub Repositories**: Monitor code repository access and potential intellectual property theft
- **Confluence**: Monitor for suspicious query and searching activity
- **Salesforce**: Track suspicious activities in collaboration and CRM platforms
### Cloud Tenant Registration

- Register all AWS, Azure, and Google Cloud tenants for real-time cloud management activity monitoring
- Enable automated alerting for suspicious cloud resource creation and configuration changes
- Implement continuous compliance monitoring across all cloud environments
### VMware Asset Inventory Collector Deployment

- Deploy collectors to all vCenter devices for complete virtual infrastructure visibility
- Monitor for unmanaged and rogue virtual machine creation
- Track virtual infrastructure changes and detect unauthorized modifications
- Implement automated asset discovery and classification for comprehensive inventory management
## Proactive Hardening and Monitoring Improvements

These are some of the proactive monitoring and employee best practices enterprises must enable to watch for attacks such as SCATTERED SPIDER.

**Identity Protection**

- Deploy phishing-resistant MFA (no SMS) and isolate privileged accounts
- Strengthen password reset processes and limit help desk MFA enrollment

**Detection and Monitoring**

- Track authentication anomalies, administrative actions, and network traffic to critical systems
- Enable comprehensive logging and behavioral analytics
- Monitor for anomalous application usage, suspicious search terms, and unusual data access patterns

**Infrastructure Security**

- Secure VMware environments, segment networks, and block unauthorized tools
- Apply least privilege in cloud environments and disable legacy authentication

**Incident Readiness**

- Maintain isolated backups, develop response playbooks, and conduct regular assessments
- Train IT/help desk staff on social engineering threats

![[Gemini_Generated_Image_5vxzg05vxzg05vxz.png]]

### **Conclusion**

While relatively young compared to syndicates such as Anonymous, Scattered Spider has quickly grown into a significant threat which we should all be on the lookout for.

Of course, this article borrows information a significant amount of information from CrowdStrike's platform. Although their blog has already been linked, their site in general is definitely worth looking into for their informative and essential documentation of threats within the cybersecurity landscape. Check it out here!

### **Sources:**

_“CrowdStrike Services Observes Scattered Spider Escalate Attacks.”_ _CrowdStrike_, 19 Sept. 2023, [https://www.crowdstrike.com/en-us/blog/crowdstrike-services-observes-scattered-spider-escalate-attacks/](https://www.crowdstrike.com/en-us/blog/crowdstrike-services-observes-scattered-spider-escalate-attacks/). Accessed 22 July 2025.

_“Scattered Spider.”_ _Wikipedia: The Free Encyclopedia_, Wikimedia Foundation, last modified 20 July 2025, [https://en.wikipedia.org/wiki/Scattered_Spider](https://en.wikipedia.org/wiki/Scattered_Spider). Accessed 22 July 2025.