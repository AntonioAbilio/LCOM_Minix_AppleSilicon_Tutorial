# Notes

**All software provided here is offered 'AS IS'. I will not be responsible for any damage that happens while following this tutorial.**

If you do have any problems in this tutorial I will try my best to help you resolve them. Please post an issue.

Also I want to make a small appeal. There exists a beta version of VirtualBox that will run on Apple Silicon however, SSH and networking functionalities are currently broken. If you find a fix please inform me so I can update this tutorial.

**After finishing the tutorial please do not forget to read the section "IMPORTANT".**

# Pre-requisites
- We will use [UTM](https://mac.getutm.app) to emulate x86.

- A qcow2 image of the hard-disk found in MINIX-LCOM (.vdi image).
    - Note: If you would like to use your own image you will need to convert it first.
    - Otherwise use the one provided in the repository (Disk size is bigger than original - this will be discussed later).

- Time.

# Instructions

1. Open UTM and select "Create a New Virtual Machine".
2. Now select "Emulate" followed by "Other".
3. Check the box "Skip ISO boot". **(continue)**
    - We do not want to install an operating system using an ISO image because the OS is already provided.
4. In Hardware do the following:
    - Set Architecture to "i386 (x86)".
    - System to "Standard PC (i440FX + PIIX, 1996) (pc-i440fx-7.1)".
    - Memory to "1536 MB". (This is what is used in VirtualBox.)
    - CPU Cores to "1". (This is what is used in VirtualBox.)
    - **(continue)**
5. You can specify anything under Storage as we will not use this disk. **(continue)**
6. Shared Directory should also be ignored for now we will deal with this later. **(continue)**
7. In Summary do the following:
    - Name: Can be anything.
    - Check the box "Open VM Settings"
    - Press "**(Save)**"
8. You should now see a new Menu.
##### The next modifications need to be done before we can turn on our Virtual Machine otherwise minix will not work properly (Kernel Panic).

9. Under "**QEMU**" uncheck "UEFI Boot" and "RNG Device" and check "Force PS/2 controller".
10. Under "**Input**" set "USB Support "to "Disabled" (USB 2.0 is unsupported by minix).
11. Disable "**Sharing**".
12. Under "**Display**" set "**Emulated Display Card**" to "virtio-vga".
13. Under "**Network**" do the following:
    - Select "Emulated VLAN" in "**Network Mode**".
    - Select "virtio-net-pci" in "**Emulated Network Card**".
    - And set the "**MAC Address**" to "08:00:27:99:34:53". (Same as VirtualBox.)
    - Under "**Port Forwarding**" select "**New**" and do:
        - **TCP as Protocol**
        - **Leave black for first input.**
        - **Set 22 for second input**
        - **Leave black for third input.**
        - **Set 2222 for forth input**
14. Under Sound select "**Intel 82801AA AC97 Audio (AC97)**" for "**Emulated Audio Card**".
15. Now finally under Drives delete the only **IDE Drive** that is present and click **New** followed by **Import...** and finally select the qcow2 disk.

# First boot

Upon booting for the first time you'll find out that currently there is no Network.

Let's start by fixing this.

## Network

1. Power ON the VM and login.
2. Switch to root user by doing: ```minix$ su```
3. Now let's do a series of steps:
    - ```minix$ netconf```
    - ```Select option 1 which should be vio0``` 
    - ```Select option 1```
    - ```Select option 2```
    - ```minix$ service network restart```
    - ```minix$ reboot```
4. Great! Network should now be working.

### SSH

SSH should be configured by now, let's try to connect to our guest OS.

1. Open the terminal app on macOS and execute the following command: ```$ ssh lcom@127.0.0.1 -p 2222```
    - If it asks to accept a fingerprint say yes.
    - You should also be asked to login. Do login :)
2. If everything is working properlly then you should see ```minix$ ```
3. SSH should also work properlly now.


# Sharing files between host and guest

This step is a work in progress.

At the moment the only way I found to "share files" between the two OSes is by using sftp... I searched for way too long and found nothing that could help me mount a shared folder while using qemu (UTM's basis for emulating other architectures on ARM.). 

This step is going to require loading a kext (Driver) as we are going to install macFUSE and SSHFS. This requieres changing settings in macOS Recovery. If you're not confortable doing this you can always just use regular old sftp but do remember that every time you want to share a folder / file you're either going to need to do it using the terminal or an app.

Also this is different than an actual shared folder ... What you're actually doing is mounting minix's drive... This is also why I said in the begining that the disk size is larger.

With this out of the way let's begin.

## Pre-requisites

Like I said before we are going to use macFUSE and SSHFS. Both can be downloaded from this website:

[OSXFUSE](https://osxfuse.github.io)

## Instructions

1. First make sure you save all of your work as we are going to have to restart the OS.
2. Let's begin by Installing macFUSE.
    - Open the installation package and follow all steps.
3. After installing you should see some pop-ups appearing. (This is expected)
4. Open settings using the pop-up.
5. Now follow this [guide](https://support.apple.com//guide/mac-help/mchl768f7291/mac). (Note: While in Startup Security Utility you'll only need to choose Reduced Security and the first checkbox.)

6. Back in macOS go to Settings -> Privacy and Security
    - Allow the extention to load.
7. Now install the SSHFS package.
8. After this you should be ready.
## Mounting

1. First create a folder. This is going to be used to mount the minix drive.
2. Now turn on the virtual machine and wait for the login prompt.
3. Now open the macOS terminal and run the following command to mount the disk:
    - ```sshfs -o allow_other,default_permissions,port=2222 lcom@127.0.0.1:/ [Drag and drop the newly created folder]```

##  Unmounting

1. Open macOS terminal and run ```unmount [Previously used folder] ``` **or** Drag the volume to Trash.

## Optional

1. Create a script to start up the virtual machine and mount / unmount the drive automatically.
    - ```open "utm://start?name=Virtual_Machine_Name"``` This allows you to start up the virtual machine from terminal.

# Serial Ports

This does not need to be configured at the moment however, it's here for future proofing sake.

## Serial Port between two Virtual Machines
Let's begin:

1.  First open [UTM](UTM://).
2. Select your first Virtual Machine and left-click it.
3. Now select "**clone**".

	<div style="display: flex; justify-content: center;">
		  <img src="https://github.com/AntonioAbilio/LCOM_Minix_AppleSilicon_Tutorial/blob/2f4c536766cbddbbcbc23c8fc4953cb0b819a8f2/Resources/Serial/Serial_1.png?raw=true") alt="Clone_Machine">
	</div>
4. Open the "**settings**" of the Virtual Machine.

5. Under "**devices**" click the "**+**" icon and select "**Serial**".

	<div style="display: flex; justify-content: center;">
		  <img src="https://github.com/AntonioAbilio/LCOM_Minix_AppleSilicon_Tutorial/blob/b52bd947c5c015807efa664e4c6d3d2f299aa038/Resources/Serial/Serial_2.png?raw=true") alt="Serial_Port_Add">
	</div>

6. Now change your values so the configuration looks like this ... and save it:

	<div style="display: flex; justify-content: center;">
		  <img src="https://github.com/AntonioAbilio/LCOM_Minix_AppleSilicon_Tutorial/blob/b52bd947c5c015807efa664e4c6d3d2f299aa038/Resources/Serial/Serial_Port_Conf_1.png?raw=true") alt="Serial_Port_Configuration_FirstVM">
	</div>

7. Now select the cloned Virtual Machine and repeat the steps 4 and 5.
8. Now change your values so the configuration looks like this:

	<div style="display: flex; justify-content: center;">
		  <img src="https://github.com/AntonioAbilio/LCOM_Minix_AppleSilicon_Tutorial/blob/b52bd947c5c015807efa664e4c6d3d2f299aa038/Resources/Serial/Serial_Port_Conf_2.png?raw=true") alt="Serial_Port_Configuration_SecondVM">
	</div>

9. Under "**Network**" click the random for Mac Address.

	<div style="display: flex; justify-content: center;">
		  <img src="https://github.com/AntonioAbilio/LCOM_Minix_AppleSilicon_Tutorial/blob/29c418e65bb22e4445e56e575d17d534b825d379/Resources/Serial/Serial_Network_1.png?raw=true") alt="Serial_Port_Configuration_SecondVM">
	</div>

10. Under "**Port Forwarding**" click on the first option and next click on the "**Edit**" button. 

	<div style="display: flex; justify-content: center;">
		  <img src="https://github.com/AntonioAbilio/LCOM_Minix_AppleSilicon_Tutorial/blob/29c418e65bb22e4445e56e575d17d534b825d379/Resources/Serial/Serial_Network_2.png?raw=true") alt="Serial_Port_Configuration_SecondVM">
	</div>
	1. Make sure the values are as follows ... click "**Save**":
		<div style="display: flex; justify-content: center;">
			  <img src="https://github.com/AntonioAbilio/LCOM_Minix_AppleSilicon_Tutorial/blob/29c418e65bb22e4445e56e575d17d534b825d379/Resources/Serial/Serial_Network_3.png?raw=true") alt="Serial_Port_Configuration_SecondVM">
		</div>

11. Finally save all changes by clicking "**Save**".

# IMPORTANT

You must be careful while doing some things as they might crash (or worst brick) your computer. These include:

- Do not shutdown the virtual machine without unmounting the minix disk from the host OS.
    - If you happen to do this you'll find out that Finder will become unresponsive and if this does not crash your computer you'll certently not have a good time unmounting the disks or gracefully shutting down your machine. If you accidently do this you can try the following:
        - Somehow opening a terminal window and performing the unmount command (note that while in this state the tab autocomplete may render the current shell unusable.)
        - Forcefully shutting down your machine (May corrupt files / render your machine unbootable.)



