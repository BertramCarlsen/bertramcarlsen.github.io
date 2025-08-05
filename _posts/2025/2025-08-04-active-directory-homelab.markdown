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

 - **User Object**: 
    - `Bob`
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
 - 7-Zip

## VirtualBox
To host this home lab, we'll use [VirtualBox](https://www.virtualbox.org/) to simulate a real-ish corporate environment. VirtualBox is a virtualization software that can be downloaded from [here](https://www.virtualbox.org/wiki/Downloads).
VirtualBox is fairly easy to install, but if needed [here](https://www.youtube.com/watch?v=homRENM8KVY) is a youtube tutorial.

## Downloading and Installing Windows 10

For the company machine we're gonna be using Windows 10. Head over to (Microsofts website)[https://www.microsoft.com/en-gb/software-download/windows10] to download the **"Windows 10 Installation Media"**:

![Windows 10 Installation Media](/img/Windows10.png)

Open the downloaded `MediaCreationTool_22H2.exe`. The tool will need administrator privileges.

Accept the License Agreements. Then we'll be presented with this screen:

![Create Installation Media](/img/CreateInstallationMedia.png)

Check **"Create Installation Media"** and hit next.

You can customize the Windows 10 settings if you want, but the recommended options are fine. Hit next.

Now we are presented with the option of which media to use. Check the ISO file option and hit next. You can then save the ISO to whatever place you want on your host pc.

![Choose Which Media To Use](/img/MediaToUse.png)

Once the Installation Media is finished downloading, we can move on to the next step.

### VirtualBox Windows 10 Setup

Open VirtualBox and click **"New"**. 

We are now presented with a screen where we need to choose:
 - A name for our virtual machine
 - The location of where to store the virtual machines data
 - The location of the ISO image 
 - The version of the operating system we are installing

Make sure to check the **"Skip Unattended Install"** box.

![Virtual Machine Name and Operating System Settings](/img/VirtualMachineName.png)

Now go to the "Hardware" tab. We should use **4096 MB** of Base Memory and **1** Processor:

![Memory and Processor Hardware Settings](/img/Hardware.png)

Leave the Hard Disk settings as is, with 50 GB of storage and using **VDI**. Click finish.

![Hard Disk Settings](/img/HardDisk.png)

We can now boot up our brand new Company Windows 10 Machine by clicking **"Start"**.
Choose the language and keyboard settings you want on the machine, and then click "**Next**" and then **"Install Now"**:

![Windows Setup Language Settings](/img/WindowsSetupLanguage.png)

On the **"Activate Windows"** screen, click **"I don't have a product key"**. Then choose **"Windows 10 Pro"**.
Accept the License Agreements and click **"Next"**. Choose the **"Custom: Install Windows Only (Advanced)"** option.

Then click the **"Drive 0 Unallocated Space"** and click **"Next"**. 

![Choose "Drive 0 Unallocated Space"](/img/Drive0UnallocatedSpace.png)

Windows will now be begin to be installed on the virtual machine. You can leave this on in the background and move on to the next step.

## Downloading and Installing Kali Linux

Kali Linux is an open-source Bebian-based Linux distribution made for Penetration Testing, Computer Forensics and Reverse Engineering. It comes with a lot of ready to go applications for attacking the company machine. 

Before installing Kali Linux we need to install [7-Zip](https://www.7-zip.org/), because Kali Linux comes in a `.7z` archive.

To download Kali Linux head over to the [Kali website](https://www.kali.org/get-kali/#kali-platforms) and click **"Virtual Machines"**. Then click the download button on **"VirtualBox"** and save it to your pc:

![Kali Linux VirtualBox Download](/img/KaliVirtualBox.png)

Once the download is finished, unzip the archive with 7-zip by right clicking the archive, and then clicking **"Show more options"**. 
Then hover over 7-Zip and click **"Extract To kali-linux-2025.2-virtualbox-amd64"** (The file name might be different for you).

Once the extraction is complete, just double click the `.vbox` file in the new folder. This will automatically import it into VirtualBox.
The default credentials for Kali Linux is:
 - **Username**:
    - `kali`
 - **Password**:
     - `kali`

## Downloading and Installing the Windows Server

Nagivate over to the [Microsoft website](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2022). Under **"Get started for free"** click **"Download the ISO"** and then enter some information.
Then click the 64-bit english ISO download and save it on your pc:

![Windows Server ISO Download](/img/WindowsServerDownload.png)

> **Note**: This evaluation edition of **Windows Server** expires in 180 days.
{: .prompt-warning }

Once the download is complete, move on to the next step.

### VirtualBox Windows Server Setup

Open up VirtualBox and click **"New"**. 

We are now presented with a screen where we need to choose:
 - A name for our virtual machine
 - The location of where to store the virtual machines data
 - The location of the ISO image 
 - The version of the operating system we are installing

Make sure to check the **"Skip Unattended Install"** box.

![Virtual Machine Name and Operating System Settings](/img/ADDC01VirtualMachineSettings.png)

Now go to the "Hardware" tab. We should use **4096 MB** of Base Memory and **1** Processor:

![Memory and Processor Hardware Settings](/img/Hardware.png)

Leave the Hard Disk settings as is, with 50 GB of storage and using **VDI**. Click finish.

![Hard Disk Settings](/img/HardDisk2.png)

Now we can click **"Start"** on the **ADDC01**.
Choose the language and keyboard settings you want on the machine, and then click "**Next**" and then **"Install Now"**:

![Windows Setup Language Settings](/img/WindowsSetupLanguage2.png)

On the **"Activate Windows"** screen, click **"I don't have a product key"**. Then choose **"Windows Server 2022 Standard Evaluation (Desktop Experience)"**.
Accept the License Agreements and click **"Next"**. Choose the **"Custom: Install Microsoft Server Operating System only (Advanced)"** option.

![Windows Server Version](/img/WindowsServerVersion2.png)

Then click the **"Drive 0 Unallocated Space"** and click **"Next"**. 

![Choose "Drive 0 Unallocated Space"](/img/Drive0UnallocatedSpace2.png)

Windows Server will now be begin to be installed on the virtual machine.

When the installation is done we'll be presented with this screen:

![Choose Password Windows Server 2022](/img/ChoosePasswordWindowsServer.png)


## Downloading and Installing the Ubuntu Server

Nagivate over to the [Ubuntu website](https://ubuntu.com/download/server).

![Ubuntu Website](/img/UbuntuWebsite.png)

### Ubuntu Server VirtualBox Setup

Open up VirtualBox and click **"New"**. 

We are now presented with a screen where we need to choose:
 - A name for our virtual machine
 - The location of where to store the virtual machines data
 - The location of the ISO image 
 - The version of the operating system we are installing

![Virtual Machine Name and Operating System Settings](/img/UbuntuVirtualMachineSettings.png)