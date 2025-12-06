---
title: Soulmate Write Up (HackTheBox)
date: 2025-12-05 12:00:00 + 0100
tags: [HackTheBox, Easy]
categories: [Write Ups]
---

This is a write up for the Soulmate Box on HackTheBox (link [here](https://www.hackthebox.com/machines/soulmate)).

## Initial Configuration

Add this line to the bottom of your `/etc/hosts` file:

```
10.10.11.86 soulmate.htb
```

## Reconnaissance

To start, `rustscan` was used to enumerate the ports open on the target machine:

```
sudo rustscan 10.10.11.86
```

![Rust Scan](../../img/Soulmate_Rustscan.png)

We see ports: 

 - `22`: SSH
 - `80`: HTTP Web Server
 - `4369`: EPMD (Erlang Port Mapper Daemon)

`Nmap` scan with the flags `-sV -sC` on the open ports: 

```
sudo nmap -sV -sC 10.10.11.86 -p22,80,4369
```

![Nmap Scan](../../img/Soulmate_NmapScan.png)

`Gobuster` scan to enumerate sub directories:

```
gobuster dir -u http://soulmate.htb -w /usr/share/wordlists/dirbuster/directory-list-1.0.txt
```

![Gobuster Scan](../../img/Soulmate_Gobuster.png)

Gobuster didn't really return anything valuable.

Checking out the website on port `80`, we can see that it's a dating site:

![Soulmate Website](../../img/Soulmate_Website.png)

To enumerate the subdomains of the website `ffuf` can be used:

```
ffuf -w /usr/share/wordlists/dirbuster/directory-list-1.0.txt -u http://soulmate.htb/ -H "Host:FUZZ.soulmate.htb" -c -t 50 -fs 154  
```

![Ffuf Scan](../../img/Soulmate_Ffuf.png)

The subdomain `ftp` was found. This needs to be added to our `/etc/hosts` file, as such:

![Hosts File](../../img/Soulmate_HostsFile.png)

Now we can navigate over to `http://ftp.soulmate.htb`:

![CrushFTP Website](../../img/Soulmate_CrushFTP.png)

In the source code of the website page, we can see the version of CrushFTP:

![CrushFTP Version](../../img/Soulmate_CrushFTPVersion.png)

A google search reveals CVE-2025-31161. Thanks to Immersive Labs Security, a POC can be found on [github](https://github.com/Immersive-Labs-Sec/CVE-2025-31161).

Armed with this exploit, we can create a user on the site:

![CVE-2025-31161 Exploit POC](../../img/Soulmate_FTPExploit.png)

Now we can log in with the credentials we just created:

