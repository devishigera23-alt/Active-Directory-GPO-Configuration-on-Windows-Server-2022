# Active Directory GPO Configuration on Windows Server 2022

This repository documents a hands-on lab where I set up Active Directory and applied Group Policy Objects (GPOs) to domain users.  
It also includes the issues I faced during configuration and how I troubleshooted them.

---

## Lab Environment

- **Host machine:** macOS (UTM)
- **Server OS:** Windows Server 2022 Standard (Evaluation)
- **Client OS:** Windows 11 Pro (domain joined)
- **Domain name:** lab.local
- **Domain Controller hostname:** BIRKIN-SRV

### Tools Used
- Server Manager  
- Active Directory Users and Computers  
- Group Policy Management Console  
- Command Prompt (`gpupdate`, `gpresult`, `whoami`)  

---

## Active Directory Setup

After installing AD DS, I created a domain and structured it using **custom Organizational Units (OUs)** instead of relying on default containers.

### OUs Created
- `birkin-TEST-USERS`
- `birkin-USERS`
- `birkin-COMPUTERS`
- `birkin-ADMINS`

Test users were created and placed inside **birkin-TEST-USERS** so that policies could be applied and tested without affecting the entire domain.

This helped me understand how **OU-based scoping** works and why applying policies directly to default OUs is not a good practice.

---

## Group Policy Objects Implemented

All GPOs were linked specifically to the **birkin-TEST-USERS** OU.

---

### GPO: Disable Control Panel

**Purpose**  
To prevent standard domain users from accessing system-level configuration options.

**Configuration Path**
User Configuration
→ Administrative Templates
→ Control Panel
→ Prohibit access to Control Panel

**Observed Result**  
When logged in as a domain user, access to Control Panel was blocked and Windows displayed a restriction message.

**Verification**
- `gpupdate /force`
- Verified via `gpresult /r` under *Applied Group Policy Objects*

---

### GPO: Disable Registry Editor

**Purpose**  
To restrict users from modifying the Windows Registry, which could lead to system instability or security issues.

**Configuration Path**
User Configuration
→ Administrative Templates
→ System
→ Prevent access to registry editing tools

**Observed Result**  
Attempting to open `regedit` resulted in a restriction message indicating that the action was blocked by administrator policy.

**Verification**
- `gpupdate /force`
- Confirmed via `gpresult /r`

---

### GPO: Automatic Windows Update

**Purpose**  
To manage Windows Update behavior centrally instead of allowing users to control update settings manually.

**Configuration Path**
Computer Configuration
→ Administrative Templates
→ Windows Components
→ Windows Update

**Observed Result**  
Windows Update settings were managed by the system, and users could not modify update behavior.

**Verification**
- GPO appeared in `gpresult /r`
- Confirmed policy linkage in Group Policy Management Console

---

## Domain & Policy Verification

To ensure policies were applied from the domain and not from local configuration, the following checks were performed:

- Verified logged-in user context using:
- Verified domain logon server using:
echo %LOGONSERVER%


This confirmed that users were authenticated against **\\BIRKIN-SRV** and policies were applied from the domain controller.

---

## Issues Faced & Troubleshooting

While working on this lab, I encountered multiple issues that helped me understand how Group Policy behaves in real environments.

- Some security-related policies (such as password requirements and brute-force protection) did not appear in `gpresult /r` even though they were configured.
- Initially, this caused confusion regarding whether the policies were applied correctly.
- I learned that certain **account and security policies are enforced by the Local Security Authority (LSA)** and may not always appear in `gpresult`, especially on client systems.
- Administrative Template–based policies were consistently visible and easier to verify using `gpresult /r`.

During testing:
- `gpupdate /force` was used frequently
- OU placement was rechecked
- GPO links were verified in Group Policy Management

### DNS-Related Issues

At times, domain users were unable to authenticate properly or reflect policy changes immediately.  
This led me to investigate DNS configuration on the domain controller.

From this, I learned that **Active Directory is tightly dependent on DNS**, and even small misconfigurations can disrupt authentication and policy application.

---

## What I Learned

- Importance of **OU design** before applying Group Policies  
- Difference between **local policies** and **domain-based GPOs**  
- How `gpresult /r` works and what it does *not* show  
- Why testing policies on a separate OU is safer than applying them domain-wide  
- Troubleshooting is a core part of system administration, not an exception  

---

## Screenshots

Screenshots included in this repository demonstrate:
- Active Directory OU structure  
- GPO linkage to OUs  
- Policy configuration inside the GPO editor  
- User-side restriction messages  
- `gpupdate /force` execution  
- `gpresult /r` output showing applied policies  

---

*This lab was intentionally kept focused and controlled to better understand Group Policy behavior rather than applying a large number of policies.*
