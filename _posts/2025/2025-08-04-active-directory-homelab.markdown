---
title: "Active Directory Home Lab"
date: 2025-08-04 12:00:00 + 0100
tags: [homelab, active directory, windows, splunk]
categories: [projects]
---

# Active Directory with Splunk

For this project I wanted to learn some more about Active Directory, so I figured I would set up a home lab.

This can serve as a tutorial for anyone wanting to build a home lab, where you can simulate attacks and a SIEM

Before jumping into it straight away, I decided to make a diagram with the different systems and solutions I would use:

 - Company machine running [Windows 10](https://www.microsoft.com/en-gb/software-download/windows10)
 - Attacker machine running [Kali Linux](https://www.kali.org/
 - Active Directory running on [Windows Server 2022](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2022)
 - Splunk SIEM running on [Ubuntu Server]()

![Home Lab](/img/Home_lab.png){: width="700" height="400" }

## What is Active Directory? (High Level Overview)

Active Directory is a database that contains **objects** such as:
 - Users
 - Groups
 - Computers
 - Security Policies
 - And much more



In order to use Active Directory a server must install a service called **Active Directory Domain Service** or **ADDS** for short. Then the server must be promoted to a **Domain Controller** or **DC** for short. This allows the server to perform **authentication** using a protocol called [Kerberos]() and **authorization** for our domain.

## Getting Started

Some prerequisites for your host system is:
 - 250 GB of storage (not all will be used, but it's recommended to have some breathing room)
 - 16 GB of RAM
 - 

[VirtualBox](https://www.virtualbox.org/) is a virtualization software that can be downloaded from [here](https://www.virtualbox.org/wiki/Downloads).