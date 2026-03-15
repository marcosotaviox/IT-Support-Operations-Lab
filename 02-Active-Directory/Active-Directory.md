# Active Directory

## What is Active Directory?

Active Directory is a Microsoft service that acts as the central control system of a company. It manages users, passwords, permissions, and computers within the corporate network. Every employee has an account in Active Directory, and through it they access their computer, email, shared folders, and internal systems.

---

## Core Concepts

- **Domain** — The "universe" of the company (e.g., thm.local)
- **OU (Organizational Unit)** — Folders that organize users by department (IT, Marketing, Sales, etc.)
- **Users** — Individual employee accounts
- **Groups** — Sets of users with the same permissions
- **Domain Controller** — The server that runs Active Directory and manages all authentication

---

## Lab Environment

- Platform: TryHackMe — Active Directory Basics room
- Tool: Active Directory Users and Computers (ADUC)
- Access: Remote Desktop Protocol (RDP)

---

## Task 1 — Managing Users and OUs

### Active Directory Structure

The lab domain (thm.local) was organized into the following OUs:

![AD Structure](../assets/screenshots/ad-structure.png)

---

### User Management Tasks

#### Reset Password
Right-clicking a user reveals the option to reset their password directly from ADUC.

![Reset Password](../assets/screenshots/ad-reset-password.png)

#### Unlock Account
When a user account is locked due to failed login attempts, the "Unlock account" checkbox appears under Properties > Account tab.

![Unlock Account](../assets/screenshots/ad-unlock-account.png)

#### Create New User
New users are created by right-clicking an OU and selecting New > User.

![Create User](../assets/screenshots/ad-create-user.png)

#### Disable Account
When an employee leaves the company, their account is disabled via right-click > Disable Account.

![Disable Account](../assets/screenshots/ad-disable-account.png)

---

## Task 2 — Managing OUs and Delegation

### Deleting a Protected OU

By default, OUs are protected from accidental deletion. Attempting to delete one returns an error.

![Delete OU Error](../assets/screenshots/ad-delete-ou-error.png)

To remove the protection, enable Advanced Features under the View menu.

![Advanced Features](../assets/screenshots/ad-advanced-features.png)

Then right-click the OU > Properties > Object tab and uncheck "Protect object from accidental deletion."

![Uncheck Protection](../assets/screenshots/ad-uncheck-protection.png)

---

### Delegation of Control

Delegation allows specific users to perform tasks on OUs without needing full Domain Admin privileges. In this lab, Phillip (IT Support) was granted permission to reset passwords for the Sales OU.

#### Step 1 — Right-click the OU and select Delegate Control

![Delegate Control](../assets/screenshots/ad-delegate-control.png)

#### Step 2 — Select Phillip as the delegated user

![Delegate Wizard](../assets/screenshots/ad-delegate-wizard.png)

#### Step 3 — Select "Reset user passwords and force password change at next logon"

![Delegate Tasks](../assets/screenshots/ad-delegate-tasks.png)

---

### Password Reset via PowerShell (as Phillip)

After delegation, Phillip can reset passwords using PowerShell via RDP without needing access to ADUC.

#### RDP Connection as Phillip

![RDP Connection](../assets/screenshots/ad-rdp-connection.png)

#### Reset Sophie's Password
```powershell
Set-ADAccountPassword sophie -Reset -NewPassword (Read-Host -AsSecureString -Prompt 'New Password') -Verbose
```

![PowerShell Reset](../assets/screenshots/ad-powershell-reset.png)

#### Force Password Change at Next Logon
```powershell
Set-ADUser -ChangePasswordAtLogon $true -Identity sophie -Verbose
```

![Force Password Change](../assets/screenshots/ad-powershell-force-password-change.png)

#### Sophie prompted to change password on next login

![Sophie Force Password](../assets/screenshots/ad-sophie-force-password.png)

#### Sophie's desktop confirming successful access

![Sophie Desktop](../assets/screenshots/ad-sophie-desktop.png)

---

## Task 3 — Managing Computers

### Computers Container (Before Organization)

By default, all machines join the "Computers" container — laptops, PCs, and servers all mixed together.

![Computers Container](../assets/screenshots/ad-computers-container.png)

### After Organization — Workstations OU

Personal computers and laptops were moved to the Workstations OU.

![Workstations Content](../assets/screenshots/ad-workstations-content.png)

### After Organization — Servers OU

Servers were moved to the Servers OU.

![Servers Content](../assets/screenshots/ad-servers-content.png)

---

## Task 4 — Group Policy Objects (GPO)

### What is a GPO?

A Group Policy Object is a collection of settings that can be applied to OUs. GPOs can target users or computers and are used to enforce security baselines across the domain.

### Group Policy Management Console

![GPO Management](../assets/screenshots/ad-gpo-management.png)

### Default Domain Policy Settings

![Default Domain Policy](../assets/screenshots/ad-gpo-default-domain-policy.png)

### Updating Minimum Password Length to 10 Characters

![Password Policy](../assets/screenshots/ad-gpo-default-domain-polic...png)

### GPO Distribution — Force Update via PowerShell

GPOs are distributed via a network share called SYSVOL. To force an immediate update:
```powershell
gpupdate /force
```

![GPO Distribution](../assets/screenshots/ad-gpo-distribution.png)

---

### GPO 1 — Restrict Control Panel Access

Created a GPO to block non-IT users from accessing the Control Panel.

#### GPO Created

![GPO Restrict Created](../assets/screenshots/ad-gpo-restrict-created.png)

#### Policy Enabled

![GPO Restrict Enabled](../assets/screenshots/ad-gpo-restrict-enabled.png)

#### GPO Linked to Marketing, Management and Sales OUs

![GPO Restrict Linked](../assets/screenshots/ad-gpo-restrict-linked.png)

---

### GPO 2 — Auto Lock Screen

Created a GPO to automatically lock workstations and servers after 5 minutes of inactivity.

#### Inactivity Limit set to 300 seconds

![Auto Lock Enabled](../assets/screenshots/ad-gpo-auto-lock-enabled.png)

#### GPO Linked to Root Domain (thm.local)

![Auto Lock Linked](../assets/screenshots/ad-gpo-auto-lock-linked.png)

---

### Verification — Control Panel Blocked for Mark

Logged in as Mark (Marketing) via RDP and attempted to open the Control Panel.

![Control Panel Denied](../assets/screenshots/ad-gpo-mark-control-panel-denied.png)

---

## Task 5 — Authentication Methods

### Kerberos
- Default authentication protocol in modern Windows domains
- Uses tickets (TGT and TGS) instead of transmitting passwords
- Process: User requests TGT from KDC → Uses TGT to request TGS → Uses TGS to access service

### NetNTLM
- Legacy protocol kept for compatibility
- Uses a challenge-response mechanism
- Password is never transmitted over the network

---

## Task 6 — Trees, Forests and Trust Relationships

### Trees
A group of Windows domains sharing the same namespace. Example: thm.local split into uk.thm.local and us.thm.local.

### Forests
The union of multiple trees with different namespaces. Example: thm.local + mht.local forming a forest.

### Trust Relationships
Allow users from one domain to access resources in another domain.
- **One-way trust** — Domain AAA trusts BBB: users from BBB can access AAA
- **Two-way trust** — Both domains mutually authorize each other
- Trust does not automatically grant access — permissions still need to be configured
