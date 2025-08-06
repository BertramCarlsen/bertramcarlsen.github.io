---
title: Active Directory Home Lab Part 2
date: 2025-08-06 12:00:00 + 0100
tags: [homelab, active directory, windows, splunk, sysmon]
categories: [projects]
---

In [part 1]({% post_url 2025-08-04-active-directory-home-lab-part-1 %}) of this project, we set up all of our virtual machines and did some basic configuration. 

In this part we'll be installing and configuring **Splunk** on our Ubuntu server and setting up **Sysmon** and **Splunk Universal Forwarder** on both our Windows 10 machine and Windows Server to start collecting telemetry and sending logs to our Splunk server.


## **Installing Splunk Enterprise**

To install Splunk on our Ubuntu Server. First, we need to set up **VirtualBox Guest Additions** to enable shared folders between our host and virtual machine.

### **Setting up Shared Folders**

On your Ubuntu Server, install the necessary packages:

```bash
sudo apt-get update
sudo apt-get install virtualbox-guest-additions-iso
sudo apt-get install virtualbox-guest-utils
```

Create a directory for our shared folder:

```bash
mkdir share
```

