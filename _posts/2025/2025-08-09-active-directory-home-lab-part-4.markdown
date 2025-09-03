---
title: Active Directory Home Lab Part 4
date: 2025-08-09 12:00:00 + 0100
tags: [Home Lab, Active Directory, Windows, Splunk, Sysmon]
categories: [Projects]
---

For the final part of this series, we'll finally get to simulate some attacks on our environment. To do this we'll use 

![Active Directory Home Lab Diagram](/img/Home_Lab_Background.png)

## Finalizing Kali Linux Setup

First we need to set the correct network settings for Kali.

Open up the Kali virtual machine and log in using the credentials `kali` / `kali`.

Then right click the network icon in the top right and click **Edit Connections...**.

![Kali Networking](/img/KaliNetworking.png)

Select **Wired connection 1** and click the settings icon. Select **IPv4 Settings** and enter the settings from the diagram and save:

![Kali IPv4 Settings](/img/KaliIPv4Settings.png)

To make sure the settings updated, we can right click the network icon in the top right. Then uncheck **Enable Networking** and recheck it.

To verify it worked, we can open up a terminal window and use the command `ip a`:

![Kali ip a](/img/KaliIPA.png)

To make sure everything works we can try pinging `google.com` and our splunk server:

![Kali Connectivity](/img/KaliConnectivity.png)

Next, we should update the kali machines repositories. In the terminal window use this command:

```
sudo apt-get update && sudo apt-get upgrade -y
```

Type in the password and once it's finished downloading we can move on.

## Crowbar Brute Force RDP Attack

**Crowbar** is a brute forcing tool that can be used during penetration tests against different protocols, but we'll be targeting RDP.

To install it, use this command:

```
sudo apt-get install -y crowbar
```

To use **Crowbar**, we need a password list. In a real world scenario an advesary would either buy or create their own wordlist. But since we are simulating the attack, we can just use `rockyou`. `Rockyou` is list of passwords from a data breach in 2009, where an attacker breached the company RockYou and leaked 32 million user accounts. `Rockyou` has been included in Kali Linux since 2013.

The `rockyou` file is located in `/usr/share/wordlists`. Unzip the file using `gunzip`:

```
sudo gunzip /usr/share/wordlists/rockyou.txt.gz
```

Then create a directory for the project on the desktop:

```
cd ~/Desktop
mkdir ActiveDirectory
```

Then we can copy the `rockyou` file to the new directory:

```
cp /usr/share/wordlists/rockyou.txt ~/Desktop/ActiveDirectory
```

Then change into `ActiveDirectory` with `cd`.

Normally a brute force attack takes a while, since you need to run through a big list of passwords, so let's make the `rockyou` file a bit smaller using the `head` command:

```
head -n 20 rockyou.txt > passwords.txt
```

The `head -n 20` command takes the first 20 lines of the file and then outputs it to `passwords.txt`.

To ensure a sucessful attack, we should add one of the passwords of the 2 users we created in our Active Directory environment. Add it using the command `nano passwords.txt`.

> **Note**: Remember to use the password you created for the account.
{: .prompt-warning }

![Kali Nano Passwords.txt](/img/KaliNanoPasswords.png)

### Enabling RDP On Windows

Now we need to actually enable RDP on the windows machine. 

