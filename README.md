<Div Align="Center">
<img width="1024" height="434" alt="image" src="https://github.com/user-attachments/assets/f5051e6f-d66d-42b9-9fca-606baaaeb3d2" />
</Div>
<br>
Created and maintained by Brian Draper
<hr>

Several of the tools listed below reference using relative paths for the command syntax. To make the reference syntax function at a command prompt the Windows path statement needs to be modified to include these additional directory search locations. As a shortcut the PATH statement can be modified at the command prompt as shown below.<br>

<b>NOTE: </b>To add a new application directory to the Windows path statement use: path set=%path%;structureoffolderstoapplication (or you can modify the environment variables within Windows; this change will require a reboot the adding paths at CMD doesn’t require a reboot but also doesn’t persist after a reboot).

<b>**An Example of a path statement: **</b><br>
path set=%path%;c:\forens~1\yara;c:\python32;c:\forens~1\tzworks;c:\forens~1\volat~1; C:\Users\Admin\AppData\Local\Programs\Python\Python~1\Lib\venv\scripts\nt;

Or path set=%path%;c:\yara – The percent marks tells the command prompt to append the new path after the ; to the existing path statement. This can also be updated using the system environment variable within Windows.
<br><br>
Living off the land binaries and Libraries

    LOLBAS Project - Site dedicated to showing tools and tactics commonly used by attackers.

    CrowdStrike Living off the Land Attacks

Windows Logon Types and reasons

    Reference: Microsoft Docs - Logon Types

    Additional Reference: FR Secure - RDP Connection Event Logs

Column Definitions

    Logon - type: Identifies the logon type initiated by the connection.   

    Reusable - credentials on destination: Indicates that the following credential types will be stored in LSASS process memory on the destination computer where the specified account is logged on locally:   

        LM and NT hashes   

        Kerberos TGTs   

        Plaintext password (if applicable)

Symbol Definitions:

    (-) denotes when credentials are not exposed.   

    (v) denotes when credentials are exposed.   

For management applications that are not in this table, you can determine the logon type from the logon type field in the audit logon events. For more information, see Audit logon events.   

In Windows-based computers, all authentications are processed as one of several logon types, regardless of which authentication protocol or authenticator is used. This table includes most common logon types and their attributes relative to credential theft:   
Logon type	#	Authenticators accepted	Reusable credentials in LSA session	Examples
Interactive (also known as, Logon locally)	2	Password, Smartcard, other	Yes	Console logon; RUNAS; Hardware remote control solutions (such as Network KVM or Remote Access / Lights-Out Card in server); IIS Basic Auth (before IIS 6.0)
Network	3	Password, NT Hash, Kerberos ticket	No (except if delegation is enabled, then Kerberos tickets present)	NET USE; RPC calls; Remote registry; IIS integrated Windows auth; SQL Windows auth; PSExec, RDP (if NLA (Network Level Authentication) is enabled)
Batch	4	Password (stored as LSA secret)	Yes	Scheduled tasks
Service	5	Password (stored as LSA secret)	Yes	Windows services
Screen Unlock	7	Password, Smartcard, etc		Reconnecting after screenlock has been activated
NetworkCleartext	8	Password	Yes	IIS Basic Auth (IIS 6.0 and newer); Windows PowerShell with CredSSP
NewCredentials	9	Password	Yes	RUNAS /NETWORK
RemoteInteractive	10	Password, Smartcard, other	Yes	Remote Desktop (formerly known as "Terminal Services")
Event Log Definitions:

    Logon - type: The type of logon requested.

    #: The numeric identifier for the logon type that is reported in audit events in the Security event log.

    Authenticators - accepted: Indicates which types of authenticators are able to initiate a logon of this type.

    Reusable - credentials in LSA session: Indicates whether the logon type results in the LSA session holding credentials, such as plaintext passwords, NT hashes, or Kerberos tickets that could be used to authenticate to other network resources.   

    Examples: List of common scenarios in which the logon type is used.

Using Sudo with options

Below is a listing of options that can be used with Sudo.
Mounting shares in Linux

Reference: How-To Geek - Mounting Removable Drives and Network Locations in WSL

Let’s say you have an external drive G: that represents either a USB drive or an optical disc drive. To mount it, you’d run the following commands:
Bash

sudo mkdir /mnt/g
sudo mount -t drvfs G: /mnt/g

You don’t actually have to mount the drive at /mnt/g, of course. You can mount it wherever you want. Just replace both instances of /mnt/g in the commands with your desired path.

To unmount the drive later so you can safely remove it, run the standard umount command:
Bash

sudo umount /mnt/g/

When working with an external device formatted with a FAT file system or any network file system, there are some limitations. The file system will not be case sensitive and you can’t create symbolic links or hard links on it.   
Checking free space on a mapped drive using Linux
Bash

du -d 1 -h /mnt/sandbox/_Engagements | sort -hr

This will show the disk usage and sort it by largest to smallest.
Mounting E01 files using Linux/WSL (EWFMount)

    cd /mnt

    sudo mkdir ewf_mount

    Change the permissions so that we can mount to it without needing to do a sudo. This is needed because /mnt is a restricted area: sudo chmod 777 /mnt/ewf_mount

    Next, is ewfmount imagename.E01 /mnt/ewf_mount

    If the mount was successful you’ll see a message similar to “ewfmount 20140807”.

    This will mount the image to a raw image mount similar to the one below. The “ewf1” is the raw mount of the image itself.

    The next part is to make a place to mount the logical images: sudo mkdir /mnt/logical_mount

    Set the permissions correct: sudo chmod 777 /mnt/logical_mount

    Now, we need to view the partition table to see which partition we’re going to be mounting. To do this we’re going to use mmls. So the command is mmls /mnt/ewf_mount/ewf1 - That should give us a result like what’s below.

    Optionally, we could run fdisk instead of mmls this will give us a little different format but still much of the same info. However, now we also know the disklabel and disk identifier. The partition sizes are also listed in a friendlier format.

Image Conversion using QEMU-IMG

    Website: www.qemu.com

    QEMU: A generic and open source machine emulator and virtualizer.

Simple conversion from QEMU qcow2 (qc2) file format to RAW:
Bash

qemu-img convert ./Diskimagefile.qc2 ./Diskimagefile.raw

Open-VM-Tools

Reference: Kali Linux Documentation - Install VMware Guest Tools

Should you decide to create your own VMware installation of Kali Linux (rather than using our pre-made VMware images), and you want to force a manual reinstall of open-vm-tools (as something has gone wrong), first make sure you are fully updated, then enter the following:   
Bash

sudo apt update
sudo apt install -y --reinstall open-vm-tools-desktop fuse
sudo reboot -f

Adding Support for Shared Folders When Using OVT

Unfortunately, shared folders will not work out of the box, some additional scripts are needed. Those can be installed easily with kali-tweaks.
Bash

kali-tweaks

In the Kali Tweaks menu, select Virtualization, then Install additional packages and scripts for VMware. Congratulations, you now have two additional tools in your toolbox!   

The first one is a little script to mount the VMware Shared Folders. Invoke it with:   
Bash

sudo mount-shared-folders

And with a bit of luck, checking /mnt/hgfs/ you should see your shared folders.   

The second script is a helper to restart the VM tools. Indeed, it’s not uncommon for OVT to stops functioning correctly (e.g. such as copy/paste between the host OS and guest VM stops working). In this case, running this script can help to fix the issues:   
Bash

sudo restart-vm-tools

How to Mount Network Locations

You can also mount network locations. Any network location you can reach from within Windows, you can mount from the Linux shell.   

Network locations can be mounted in one of two ways. If you map a network drive to a drive letter, you can mount it using the same options as above. This would give you an opportunity to easily sign into the network share and enter your credentials in File Explorer. For example, if your mapped network drive is F:, you could run the following commands to mount it:   
Bash

sudo mkdir /mnt/f
sudo mount -t drvfs F: /mnt/f
sudo mount -t cifs //192.168.1.66/NTDS /mnt/c -o username=dma-user

You can also specify a drive using its UNC (Universal Naming Convention) path. For example, if the path to the network share is \\server\folder, you’d run the following command. Again, use whatever mount point you like in place of /mnt/folder.
Bash

sudo mkdir /mnt/folder
sudo mount -t '\\server\folder' /mnt/folder

Setting Time In Linux

Setting Linux time:
Bash

sudo date -s "19 APR 2012 11:14:00"

Using ClamAV for Linux to scan folder
Bash

sudo freshclam              # Updates the AV Signatures
clamscan -r /mnt/i/[root]/  # Recursively scans a folder path

Changing / Setting the Default User for WSL Ubuntu

The “ubuntu” part of the command references the Linux distro that’s installed, depending on the version that’s installed this could be ubuntu2004.exe or even ubuntu2204.exe. You can identify the version by typing wsl --list.

    NOTE: This is useful when you don’t know the su password. You can change to the root user and then change the su password and then change back.

Bash

ubuntu config --default-user <username>

Creating a Triage Image Using KAPE

    Reference: SANS 508 Workbook Exercise 3.2-7 or 508.3 Page 22

Targeted Collection:
DOS

kape --tsource F --target RegistryHives --vhdx localhost --tdest G:\cdrive

Full Triage Collection:
DOS

kape --tsource I --target !All --vhdx Server2013_TriageImage --tdest e:\Chem27\Server2012

Collection Examples:
DOS

.\kape.exe --tsource E: --tdest F:\KapeTest-USG87 --tflush --target MiniTimelineCollection --vss --zip
kape.exe --tsource J: --tdest "F:\USG204\Disk Images\CITIAMFS3_files\kape" --target !BasicCollection --vss --zip

Kape87Test –gui (MFT, Registry and EventLogs) - QuickTimeline

Processing:
DOS

.\kape.exe --msource F:\KapeTest-USG87\2020-06-10T202820_Kape87Test3 --mdest F:\KapeTest-USG87\MiniTimeline --mflush --module Mini_Timeline,Mini_Timeline_Slice_by_Daterange --mvars computerName:ARSHIHIL4059095^dateRange:01/01/2020-03/03/2020 --gui

Listing Volume Shadow Copies

Good for listing the dates and times of copies to determine which VSS could hold information about an incident.
DOS

vssadmin list shadows /for=D: > VSSAdminList_win7-c-drive.txt

Looking for WMI Object Persistence

    Reference: GitHub - davidpany/WMI_Forensics

This repository contains scripts used to find evidence of persistence in WMI repositories, specifically OBJECTS.DATA files located at:   

    C:\WINDOWS\system32\wbem\Repository\OBJECTS.DATA   

    C:\WINDOWS\system32\wbem\Repository\FS\OBJECTS.DATA   

CCM_RUA_Finder.py   

CCM_RUA_finder.py extracts SCCM software metering RecentlyUsedApplication logs from OBJECTS.DATA files. Then it outputs related rules.   

Usage:
Bash

CCM_RUA_Finder.py -i path\to\OBJECTS.DATA -o path\to\output.xls

The output file will be TSV formatted. Excel will automatically parse TSV files with .xls extensions.
PyWMIPersistenceFinder.py

PyWMIPersistenceFinder.py is designed to find WMI persistence via FilterToConsumerBindings solely by keyword searching the OBJECTS.DATA file without parsing the full WMI repository.   

In testing, this script has found the exact same data as python-cim's show_FilterToConsumerBindings.py without requiring the setup. Only further testing will indicate if this script misses any data that python-cim can find.   

In theory, this script will detect FilterToConsumerBindings that are deleted and remain in unallocated WMI space, but I haven't had a chance to test yet.   

Usage:
Bash

PyWMIPersistenceFinder.py <OBJECTS.DATA file>

The output is text based in the following format for each binding:
Plaintext

<consumer name>-<filter name>
             <optional notes>
        Consumer: <consumer name><consumer execution details>
        Filter: <filter name><filter listener details>

Password Cracking using John the Ripper

The password cracking below is from an exercise that I did for the Network+ class, but I thought I’d include the steps as a good overview of the process. I’m pretty sure that the dictionary used had the passwords “added” to it to make sure that the exercise was a success.
1. Crack the root password on Support

    From the Favorites bar, select Terminal

    At the prompt, type cd /usr/share/john and press Enter to change directories to the folder containing the John the Ripper password file.

    Type ls and press Enter to list the files in the directory.

    Type cat password.lst and press Enter to view the password list. This is an abbreviated list.

    Type cd and press Enter to go back to root.

    Type john /etc/shadow and press Enter to crack the Linux passwords. Notice that the root password of 1worm4b8 was cracked.

    Type john /etc/shadow and press Enter to attempt to crack the Linux passwords again. Notice that it does not attempt to crack the password again. The cracked password is already stored in the john.pot file.   

    Type cat ./.john/john.pot and press Enter to view the contents of the john.pot file.

    Type john /etc/shadow --show and press Enter as an alternate method of viewing the previously cracked password.

2. Crack the password of the protected.zip file

    From the Terminal prompt, type ls and press Enter to view the contents of the home directory. Notice the protected.zip file you wish to crack.

    Type zip2john protected.zip > ziphash.txt and press Enter to copy the hashes to a text file.

    Type cat ziphash.txt and press Enter to confirm that the hashes have been copied.

    Type john --format=pkzip ziphash.txt and press Enter to crack the password.

    Type john ziphash.txt --show and press Enter to show the password.

Repairing Zip Files and Checking Integrity

    Integrity check on zip files with:
    Bash

    zip -T filename.zip

    Repairing a Zip file works great for Plaso files, not so good for Iris files:
    Bash

    zip -FF broken.plaso --out fixed.plaso

Password brute forcing with Hydra

Here you will leverage the hydra to try and break into the webserver using the taugratin account. You will also feed the tool a password list and tell it to target the webserver for SSH access. See how the password list is very “tator”-oriented.

Execute the following commands:

    Make sure the directory is on Desktop:
    Bash

    cd /home/kali/Desktop

    Now find the password list:
    Bash

    hydra -l taugratin -P pwlist.txt 172.16.193.171 ssh

Linux Changing File Permissions using Chmod

(Notes section placeholder in original document)
Linux Changing File Ownership using Chown

(Notes section placeholder in original document)
