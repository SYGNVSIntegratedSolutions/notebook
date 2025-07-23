![[istockphoto-1390481905-612x612.jpg]]
## How to Enable MFA for Microsoft 365 Accounts 💻

**A Practical Admin Checklist**

---

### 1. Sign in with a Global Admin Account

Go to [https://portal.office.com](https://portal.office.com) and log in with a **Global Administrator** or **Privileged Role Administrator** account.

---

### 2. Open the Microsoft 365 Admin Center

From the Office home page, select **Admin** to launch the **Microsoft 365 Admin Center**.

---

### 3. Navigate to Active Users

In the left sidebar, go to **Users > Active Users**. This displays a list of all accounts in your tenant.

---

### 4. Open the Legacy MFA Portal

At the top of the Active Users page, click **Multi-factor authentication** (sometimes under **More actions**).  
This opens the classic per-user MFA management page.

---

### 5. Enable MFA for Specific Users

In the list, check the box next to each user you want to enable MFA for.  
On the right, click **Enable** then confirm when prompted.  
This will require the user to register a second factor the next time they sign in.

---

### 6. Notify Users and Provide Setup Instructions

After enabling, users must sign in and follow the prompts to register their second factor.  
Recommend using the **Microsoft Authenticator app** for the best experience.  
SMS or phone call can be used as backup methods.

---

### 7. Verify User Registration Status

Go back to the MFA portal to check who has completed registration.  
Status will show as **Enabled** or **Enforced** once registration is complete.

---

### 8. Use Conditional Access for Stronger Control (Recommended)

Per-user MFA is basic. For more flexible policies, use **Conditional Access** in Entra ID:

1. Go to [https://entra.microsoft.com](https://entra.microsoft.com) (Entra Admin Center).
    
2. In the left pane, select **Protection > Conditional Access**.
    
3. Click **+ New policy**.
    
4. Target users or groups.
    
5. Set conditions (e.g., all cloud apps).
    
6. Under **Access controls**, choose **Grant access > Require multi-factor authentication**.
    
7. Enable the policy and click **Create**.
    

---

### 9. Consider Enabling Security Defaults (Good for Small Organizations)

If you don’t want to build Conditional Access rules:

- In Entra ID, go to **Properties**.
    
- Click **Manage Security Defaults** at the bottom.
    
- Set to **Enabled**.  
    Security Defaults enforce MFA for all admins and encourage MFA for all users.
    

---

### 10. Monitor and Review Sign-ins

Regularly review sign-in logs:

- Go to **Entra ID > Monitoring > Sign-ins**.
    
- Look for sign-ins that bypass MFA or come from unfamiliar locations.
    
- Adjust policies if needed.
    

---

## Bonus Tips

- **Use Authenticator App:** Recommend the Microsoft Authenticator app for push notifications. It’s more secure than SMS.
    
- **Test With a Pilot Group First:** Before enforcing org-wide, test with a small group to check user experience.
    
- **Backup Methods:** Encourage users to add multiple MFA methods (like a phone number and an authenticator) in case they lose a device.
    
- **Train Employees:** Send out clear instructions on how to set up and use MFA.
    
- **Document Emergency Access:** Make sure you have a break-glass account with strong protections and backup methods for emergencies.