---
title: Active Directory Home Lab Part 2
date: 2025-08-06 12:00:00 + 0100
tags: [homelab, active directory, windows, splunk, sysmon]
categories: [projects]
image:
  path: /img/Home_Lab.png
  alt: Active Directory Home Lab Diagram
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

On your host machine, go to [splunk.com](https://splunk.com), create an account, and click **Free Trials and Downloads**. Scroll down to **Splunk Enterprise**, click **Get My Free Trial**, select **Linux** as the operating system, and download the **.deb** file.

![Splunk Download](/img/SplunkDownload.png)

In VirtualBox, with your Ubuntu Server selected, go to **Devices** -> **Shared Folders** -> **Shared Folder Settings**.

![Shared Folder](/img/SharedFolder.png)

Add a new folder:
- **Folder Path**: The directory where you saved the Splunk installer
- **Folder Name**: You can leave this as default
- Check **Auto-mount** and **Make Permanent**

Click **OK**

![Shared Folder Settings](/img/SharedFolderSettings.png)

### **Mounting the Shared Folder**

Back in your Ubuntu Server, add your user to the `vbox` shared folder group:

```bash
sudo adduser johntheuser vboxsf
```

![Add User To Group](/img/AddUser.png)

> **Note**: Replace `johntheuser` with your actual username.
{: .prompt-warning }

Reboot the virtual machine:

```bash
sudo reboot
```

After rebooting, mount the shared folder:

```bash
sudo mount -t vboxsf -o uid=1000,gid=1000 shared_ubuntu share
```
> **Note**: Replace `shared_ubuntu` with your directory name, where Splunk is stored on the host PC.
{: .prompt-warning }

Navigate to the shared folder and install Splunk:

```bash
cd share
ls -la
sudo dpkg -i splunk-10.*.deb
```

![dpkg -i splunk](/img/dpkg-splunk.png)

Once you see **Complete** we can move on.

### **Configuring Splunk**

If you `cd` to `/opt/splunk` and run `ls -la`, you will see that all the files belong to the user `Splunk`. This limits the permissions to that user.

![Splunk User](/img/SplunkUser.png)

Change to the `/opt/splunk` directory and switch to the `splunk` user:

```bash
cd /opt/splunk
sudo -u splunk bash
```

Start Splunk for the first time:

```bash
cd bin
./splunk start
```

![Start Splunk For The First Time](/img/StartSplunkFirstTime.png)

Accept the license agreement by pressing **q** and then **y**.

Create an administrator account:
- **Username**: `ultrajohn` (or whatever you prefer)
- **Password**: Choose a password `UltraPassword49`

Now `exit` back to the regular user and enable **Splunk** to start on boot with the `splunk` user:

```bash
exit
sudo ./splunk enable boot-start -user splunk
```
> **Note**: Make sure you all still in the directory `/opt/splunk/bin`
{: .prompt-warning }

![Splunk Boot Start](/img/SplunkBoot.png)