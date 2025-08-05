---
title: "Active Directory Home Lab"
date: 2025-08-04 12:00:00 + 0100
tags: [homelab, active directory, windows, splunk]
categories: [projects]
---

# Active Directory with Splunk

I wanted to learn some more about [Active Directory](https://en.wikipedia.org/wiki/Active_Directory), so I figured I would set up a home lab.

This can serve as a tutorial for anyone wanting to build a home lab, where you can simulate attacks and a SIEM solution for logs.

Before jumping into it straight away, I decided to make a diagram with the different systems and solutions I would use:

 - Company PC running [Windows 10](https://www.microsoft.com/en-gb/software-download/windows10)
 - Attacker PC running [Kali Linux](https://www.kali.org/)
 - Active Directory running on [Windows Server 2022](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2022)
 - Splunk SIEM running on [Ubuntu Server](https://ubuntu.com/download/server)

![Home Lab](/img/Home_lab.png){: width="700" height="400" }

## What is Active Directory? (High Level Overview)

Active Directory is a database that contains **objects** such as:
 - Users
 - Groups
 - Computers
 - Security Policies
 - And much more

These **objects** will contain **attributes** that hold information about that **object**.
An example would be:

 - **User Object**: `Bob`
 - **Attributes**: 
  - First Name = `Bob`
  - Last Name = `Smith`
  - Age = `31`


In order to use Active Directory a server must install a service called **Active Directory Domain Service** or **ADDS** for short. Then the server must be promoted to a **Domain Controller** or **DC** for short. This allows the server to perform **authentication** using a protocol called [Kerberos](https://en.wikipedia.org/wiki/Kerberos_(protocol)) and **authorization** for our domain.

## Prerequisites

Some prerequisites for your host system is:
 - Windows OS
 - 250 GB of storage (not all will be used, but it's recommended to have some breathing room)
 - 16 GB of RAM

## VirtualBox
To host this home lab, we'll use [VirtualBox](https://www.virtualbox.org/) to simulate a real-ish corporate enviromnent. VirtualBox is a virtualization software that can be downloaded from [here](https://www.virtualbox.org/wiki/Downloads).
VirtualBox is fairly easy to install, but if needed [here](https://www.youtube.com/watch?v=homRENM8KVY) is a youtube tutorial.

## Installing Windows 10

For the company machine we're gonna be using Windows 10. Head over to (Microsofts website)[https://www.microsoft.com/en-gb/software-download/windows10] to download the "Windows 10 Installation Media":

![Windows 10 Installation Media](/img/Windows10.png)

Open the downloaded `MediaCreationTool_22H2.exe`. The tool will need administrator privileges.

Accept the License Agreements. Then we'll be presented with this screen:

![Create Installation Media](/img/CreateInstallationMedia.png)

Check "Create Installation Media" and hit next.

You can customize the Windows 10 settings if you want, but the recommended options are fine. Hit next.

Now we are presented with the option of which media to use. Check the ISO file option and hit next. You can then save the ISO to whatever place you want on your host pc.

![Choose Which Media To Use](/img/MediaToUse.png)

Once the Installation Media is finished downloading, we can move on to the next step.

### VirtualBox Windows 10 Setup


