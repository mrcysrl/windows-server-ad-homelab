# Windows Server & Active Directory Home Lab

## Overview

This project simulates a small company's IT infrastructure using Windows Server 2022 and Windows 10 Pro, built entirely in VirtualBox. The goal was to gain hands-on experience with the core responsibilities of a Windows systems administrator: deploying a domain controller, managing users and groups, configuring DHCP and DNS, enforcing security policy through Group Policy, and setting up secure file sharing.

**Environment:**
- Oracle VirtualBox
- Windows Server 2022 (Desktop Experience)
- Windows 10 Pro Client VM
- Isolated internal (Host-only) virtual network

---

## 1. Initial Server Setup

![Server Manager Initial](screenshots/00-server-manager-initial.png)

Before turning this server into a domain controller, I first configured the basics: named the machine "DC01" and set a static IP address so it would always be reachable on the network. At this point, the server was just a standalone machine (Workgroup), not yet part of any company domain.

---

## 2. Virtual Network Configuration

![VirtualBox Host-only Network](screenshots/15-virtualbox-hostonly-network.png)
![VirtualBox Host-only Network](screenshots/16-virtualbox-hostonly-network.png)

To simulate an isolated company network, I used VirtualBox's "Host-only" networking mode instead of connecting the lab directly to the internet. This creates a private virtual network (192.168.56.0/24) that only the virtual machines can see — similar to how a real company's internal network is separated from the public internet.

I disabled the built-in DHCP server for this network because the domain controller (DC01) would handle IP address assignment later, just like in a real enterprise environment.

---

## 3. Server Network Configuration

![DC01 Network Settings](screenshots/17-dc01-network-settings.png)

DC01's virtual network adapter was attached to the Host-only network created above, placing it on the private 192.168.56.0/24 network.

I then assigned the server a static IP address (192.168.56.10) instead of using DHCP. Servers are always given static, unchanging IP addresses in real environments — this ensures other computers on the network can reliably find the domain controller at all times. I also set the DNS server to point to itself (127.0.0.1), since this server would also act as the network's DNS server.

---

## 4. Domain Controller Successfully Created

![AD DS Promotion Complete](screenshots/01-adds-promotion-complete.png)

After installing the Active Directory service and promoting the server, DC01 became the domain controller for a new company domain: **corp.local**. I logged in as the domain administrator (CORP\Administrator), confirming the domain was created successfully and ready to manage users, computers, and security policies.

---

## 5. Creating Organizational Units

![OU Creation](screenshots/02-ou-creation.png)
![OU Creation](screenshots/03-ou-creation.png)
![OU Creation](screenshots/04-ou-creation.png)

To organize the domain the way a real company would, I created four Organizational Units (OUs) — folders used to separate and manage different types of accounts and resources:

- **Employees** – user accounts for staff members
- **Workstations** – employee computers joined to the domain
- **Servers** – additional servers in the environment
- **Groups** – security groups used to manage permissions

Note: AD automatically creates a default "Computers" container during setup, but it's a container (not an OU) and can't have Group Policies applied to it directly. I created a separate "Workstations" OU instead, so computer accounts can be managed with proper policies — a standard enterprise practice.

For each OU, I kept the "Protect container from accidental deletion" option checked — a safeguard that prevents an OU (and everything inside it) from being accidentally deleted.

---

## 6. Creating User Accounts

![Users Created](screenshots/05-users-created.png)
![Users Created](screenshots/06-users-created.png)
![Users Created](screenshots/07-users-created.png)
![Users Created](screenshots/08-users-created.png)

I created sample employee accounts inside the "Employees" OU to simulate a small company's staff. Each account was set up with the "User must change password at next logon" option enabled — a common security practice that forces new employees to set their own private password on first login, rather than continuing to use a temporary one set by IT.

---

## 7. Creating Security Groups

![Groups Created](screenshots/09-groups-created.png)
![Groups Created](screenshots/10-groups-created.png)
![Groups Created](screenshots/11-groups-created.png)

I created four security groups inside the "Groups" OU to represent different departments within the simulated company: Sales, IT, HR, and Management.

Security groups are used to manage permissions efficiently — instead of granting access to files, folders, or resources one employee at a time, IT administrators assign permissions to a group, and then simply add or remove employees from that group as needed.

---

## 8. Assigning Employees to Security Groups

![Group Membership](screenshots/12-group-membership.png)
![Group Membership](screenshots/13-group-membership.png)
![Group Membership](screenshots/14-group-membership.png)

Each employee account was added as a member of their corresponding department group (Sales, IT, HR, or Management) using the "Member Of" tab in their user account properties.

This demonstrates how access control works in a real company: permissions and policies are typically applied to a group rather than to individual users, so adding someone to the right group automatically gives them the appropriate access for their role.

---

## 9. Installing and Configuring DHCP Server

![DHCP Role Install](screenshots/18-dhcp-role-install.png)
![DHCP Scope Configuration](screenshots/19-dhcp-scope-configuration.png)
![DHCP Scope Configuration](screenshots/20-dhcp-scope-configuration.png)
![DHCP Scope Configuration](screenshots/21-dhcp-scope-configuration.png)
![DHCP Scope Configuration](screenshots/22-dhcp-scope-configuration.png)
![DHCP Scope Configuration](screenshots/23-dhcp-scope-configuration.png)
![DHCP Scope Configuration](screenshots/24-dhcp-scope-configuration.png)
![DHCP Scope Configuration](screenshots/25-dhcp-scope-configuration.png)
![DHCP Scope Configuration](screenshots/26-dhcp-scope-configuration.png)
![DHCP Scope Configuration](screenshots/27-dhcp-scope-configuration.png)
![DHCP Scope Configuration](screenshots/28-dhcp-scope-configuration.png)

I installed the DHCP Server role on DC01 using Server Manager's "Add Roles and Features" wizard, the same method used earlier to install Active Directory Domain Services.

DHCP (Dynamic Host Configuration Protocol) automatically assigns IP addresses to computers on the network, so I don't have to manually configure a static IP on every device that joins.

After installing the role, I created a DHCP scope — a defined range of IP addresses the server is allowed to hand out:
- **Address range:** 192.168.56.100 – 192.168.56.150
- **Subnet mask:** 255.255.255.0
- **Lease duration:** 8 days (default)
- **DNS server:** 192.168.56.10 (DC01, doubling as the network's DNS server)
- **Domain name:** corp.local

I left the default gateway blank since this is an isolated lab network with no internet routing, and left address exclusions empty since the server's static IP already falls outside the scope range.

---

## 10. Verifying DHCP Configuration

![DHCP Client Verification](screenshots/29-dhcp-client-verification.png)
![DHCP Client Verification](screenshots/30-dhcp-client-verification.png)

To confirm the DHCP server was working correctly, I created a second virtual machine (Client01) and connected it to the same internal network as the domain controller — without manually assigning it an IP address.

Running `ipconfig /all` on the client confirmed it automatically received an IP address from DC01's DHCP scope (192.168.56.101), along with the correct DNS server (192.168.56.10) and domain name (corp.local) — proving the DHCP server was successfully handing out network settings to new devices.

---

## 11. Joining a Client Computer to the Domain

![Domain Join Client01](screenshots/31-domain-join-client01.png)
![Domain Join Client01](screenshots/32-domain-join-client01.png)
![Domain Join Client01](screenshots/33-domain-join-client01.png)

I created a second virtual machine (Client01, running Windows 10 Pro) and connected it to the same internal network as the domain controller. After confirming it automatically received network settings from DHCP, I joined the computer to the company domain (corp.local) using domain administrator credentials.

Joining a computer to a domain allows it to be centrally managed — for example, applying company-wide security policies, restricting settings, or letting employees log in with their company account instead of a separate local account on every machine.

Along the way, I encountered a real-world scenario: the domain administrator account required a password change before the join could complete, due to a "must change password" flag. This is a common security policy in real organizations to prevent default or stale passwords from remaining in use.

After resolving that, entering the domain administrator credentials successfully completed the join — Windows displayed a "Welcome to the corp.local domain" confirmation message, indicating the computer account was successfully created in Active Directory and the machine is now recognized as part of the company domain. A restart was then required to fully apply the domain membership.

---

## 12. Verifying Domain User Login

After joining Client01 to the domain, I tested logging in with one of the employee accounts (CORP\sjohnson) instead of a local Windows account.

Since the account was created with "User must change password at next logon" enabled, I was prompted to set a new password on first login — simulating how a new employee would set up their own private password the first time they use their company laptop.

After setting the new password, the login completed successfully, confirming that the client computer can authenticate against the domain controller and that the employee account created earlier works correctly for real logins.

---

## 13. Applying Group Policy: Disabling Control Panel Access

![Creating GPO](screenshots/34-creating-gpo.png)
![Creating GPO](screenshots/35-creating-gpo.png)
![GPO Disable Control Panel](screenshots/36-gpo-disable-control-panel.png)
![GPO Disable Control Panel](screenshots/37-gpo-disable-control-panel.png)
![GPO Disable Control Panel](screenshots/38-gpo-disable-control-panel.png)
![GPO Disable Control Panel](screenshots/39-gpo-disable-control-panel.png)
![GPO Disable Control Panel](screenshots/40-gpo-disable-control-panel.png)

I created a Group Policy Object (GPO) named "Disable Control Panel" and linked it to the "Employees" OU. Inside the policy, I enabled the setting "Prohibit access to Control Panel and PC settings" under User Configuration.

Group Policy allows IT administrators to enforce consistent security and usability restrictions across many computers and users at once, without manually configuring each device individually. By linking this policy to the Employees OU specifically, the restriction only applies to regular staff accounts — administrators and other OUs are unaffected.

---

## 14. Testing the Group Policy Restriction

![GPO Test Control Panel Blocked](screenshots/41-gpo-test-control-panel-blocked.png)
![GPO Test Control Panel Blocked](screenshots/42-gpo-test-control-panel-blocked.png)
![GPO Test Control Panel Blocked](screenshots/43-gpo-test-control-panel-blocked.png)

To apply the new Control Panel restriction, the client computer needs to refresh its Group Policy settings. This can happen automatically on restart, or immediately by running `gpupdate /force` in an elevated Command Prompt — a command that forces Windows to re-check and apply any policy changes from the domain controller without waiting for the next scheduled refresh.

After updating the policy, I logged in as the employee account (CORP\sjohnson) and attempted to open Control Panel. As expected, Windows blocked access and displayed a restriction message, confirming that the policy was successfully applied from the domain controller down to the employee's account.

---

## 15. Creating a Shared Folder

![File Share Creation](screenshots/44-file-share-creation.png)
![File Share Creation](screenshots/45-file-share-creation.png)
![File Share Creation](screenshots/46-file-share-creation.png)
![File Share Creation](screenshots/47-file-share-creation.png)
![File Share Creation](screenshots/48-file-share-creation.png)

I created a shared folder (C:\Shares\HR) on the domain controller to simulate a department-specific network share, similar to how companies store shared files for teams like HR, Sales, or IT.

At the share level, I removed the default "Everyone" permission — which would allow any domain user to access the folder — and granted access only to the "HR" security group instead. This ensures that only employees belonging to HR can connect to and access this shared folder over the network.

---

## 16. Configuring NTFS Permissions

![NTFS Permissions](screenshots/49-ntfs-permissions.png)
![NTFS Permissions](screenshots/50-ntfs-permissions.png)
![NTFS Permissions](screenshots/51-ntfs-permissions.png)
![NTFS Permissions](screenshots/52-ntfs-permissions.png)

In addition to the share-level permissions, I configured NTFS permissions directly on the folder itself — this is the second, more granular layer of access control in Windows, and best practice is to configure both layers rather than relying on just one.

I removed the default "Users" group (which would have allowed all domain users to access the folder) and granted "Modify" permission specifically to the HR security group. Modify access allows HR staff to read, edit, and delete files within the folder, without giving them administrative rights like changing permissions or taking ownership — following the principle of least privilege.

This two-layer approach (share permissions + NTFS permissions) ensures that only authorized personnel can access sensitive department files, even if someone attempts to bypass one layer of restriction.

---

## 17. Verifying Access Control

![Access Control Verification](screenshots/53-access-control-verification.png)
![Access Control Verification](screenshots/54-access-control-verification.png)
![Access Control Verification](screenshots/55-access-control-verification.png)
![Access Control Verification](screenshots/56-access-control-verification.png)
![Access Control Verification](screenshots/57-access-control-verification.png)
![Access Control Verification](screenshots/58-access-control-verification.png)
![Access Control Verification](screenshots/59-access-control-verification.png)

To confirm the share and NTFS permissions were working as intended, I tested access using two different employee accounts:

- **Emily (HR department)** — successfully accessed the \\DC01\HR share and was able to create and save files, confirming her group membership correctly granted access.
- **Marc (IT department)** — attempted to access the same share and was denied, since he is not a member of the HR security group.

This confirms that department-based access control was functioning correctly, ensuring sensitive files are only accessible to authorized personnel — the same principle used by real companies to protect data like payroll, employee records, or financial information.

---

## Key Troubleshooting Experience

During Domain Controller promotion, the VM repeatedly froze on a black screen after the mandatory post-promotion restart, confirmed via VirtualBox's resource monitor (VMM load at 100%, Guest load at 0% — indicating the guest OS was not executing, not just slow to boot). This was resolved by restoring a pre-installation snapshot, increasing allocated RAM, and avoiding switching window focus (alt-tabbing) during the reboot process. This experience reinforced the importance of snapshotting before major system changes and methodically isolating variables (RAM, host virtualization features, VirtualBox version) when diagnosing VM-level issues.

---

## Skills Demonstrated

Active Directory Domain Services (AD DS) &nbsp;|&nbsp; Group Policy Management (GPO) &nbsp;|&nbsp; Organizational Unit (OU) Structuring
DNS Server Configuration &nbsp;|&nbsp; DHCP Server Configuration & Scope Management &nbsp;|&nbsp; Static IP Addressing
User Account Provisioning &nbsp;|&nbsp; Security Group Management &nbsp;|&nbsp; Domain Join & Client Configuration
NTFS Permissions &nbsp;|&nbsp; File Share Management &nbsp;|&nbsp; Windows Server 2022 Administration
VirtualBox Virtualization &nbsp;|&nbsp; Virtual Network Configuration &nbsp;|&nbsp; Windows 10/11 Client Administration
System Troubleshooting &nbsp;|&nbsp; Password Policy Enforcement &nbsp;|&nbsp; Account Lockout Management
