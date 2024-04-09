---
layout: post
title : "Installing Ubuntu Server VM on Managed Network Using VMWare Fusion - MacOS"
image : "/assets/images/post/post-1.png"
author : "Marvin"
categories: development
date: 2024-03-22 1:27:58 +0600
tags: [Virtualization, Enterprise, Virtual Machines, Linux, Proxmox]
top: 1
---

If you are interested in installing an Ubuntu server VM to install different software components to enhance the security of your personal network on MacOS, you can do so using VMWare Fusion. You can download VMWare Fusion from [their official download site](https://www.vmware.com/products/fusion/fusion-evaluation.html).

### Download Ubuntu Server ISO
* Go to the Ubuntu server website and download the [Ubuntu Server ISO](https://ubuntu.com/download/server).
* After downloading the iso image, make sure it is in a location that you can refer to while setting up the virtual machine on VMWare Fusion.

### Setup VM on VMWare Fusion
* Open VMWare Fusion and select + to add a new VM in the top left corner.
* Select New
* Click on Install from disc or image
* Select iso image that was downloaded in the previous step.
* Select Firmware Type
* Before finishing up, select customize settings
* Name the Ubuntu Server VM
* In the settings windows - Customize any resource specific settings.
* In order for the VM to show as its own separate device on your managed network, go to Network Adapter Settings.
* Under bridged networking, select WiFi. Do not select shared with my Mac, as it will not get its own IP Address.
* Close out the window and click the play button to start up the Server VM.

### Setup Ubuntu Server
* Press play button or start ubuntu server VM
* Select try or install ubuntu server
* Select language
* Select update to new installer if option is available
* Select layout and variant for keyboard settings
* Select type of install (Ubuntu Server) and any additional options
* VM will automatically be assigned an IP Address - You can set this statically on your managed network service or you do so in the network connections setup of the VM, select done to continue
* Add a proxy address, if none, select done
* Mirror address should automatically be filled in, select done to continue
* In the resources configurations, it is highly encouraged to encrypt the VM. To encrypt the VM select encrypt the LVM with LUKS and set an encryption passphrase. Make sure to remember the passphrase as this will be used to unencrypt the vm disk to access the VM
* In the storage configuration, make sure to change the ubuntu-lv size as this is under the encrypted partition of the VM and will be used as the main storage component of the vm. Change this to the capped partion size. It will tell you the max size it allows. Set it to that
* Are you sure you want to continue, select continue.
* Next is the VM profile setup to login to the vm
* Name: name of the user.
* Server Name: name of the ubuntu server
* Username: username used to login into the server
* Password: password used to login into the server, choose a different password than the encryption password.
* Select done

After reboot, the user will be able to access the Ubuntu Server and can install different security components to monitor their network.

