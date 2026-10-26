<Div Align="Center">
<img width="1024" height="434" alt="image" src="https://github.com/user-attachments/assets/f5051e6f-d66d-42b9-9fca-606baaaeb3d2" />
</Div>
<br>
Created and maintained by Brian Draper
<hr>

This document is a compilation of my notes and research into the use of different tools. The document started off as my index for the GCFE exam. These tools are used to manually analyze artifacts. 

The best way to use the document is to “search” for key terms relating to what you’re looking for. If there’s something that you’re looking for that isn’t in here please let me know and I’ll work on getting it added. This document is ALWAYS being updated as things change and tools are updated. 

---

## Table of Contents

| Section A | Section B |
| :--- | :--- |
| 1. [Living off the land binaries and Libraries](#living-off-the-land-binaries-and-libraries) | 11. [Using ClamAV for Linux to scan folder](#using-clamav-for-linux-to-scan-folder) |
| 2. [Windows Logon Types and reasons](#windows-logon-types-and-reasons) | 12. [Changing / Setting the Default User for WSL Ubuntu](#changing--setting-the-default-user-for-wsl-ubuntu) |
| 3. [Using Sudo with options](#using-sudo-with-options) | 13. [Creating a Triage Image Using KAPE](#creating-a-triage-image-using-kape) |
| 4. [Mounting shares in Linux](#mounting-shares-in-linux) | 14. [Listing Volume Shadow Copies](#listing-volume-shadow-copies) |
| 5. [Checking free space on a mapped drive using Linux](#checking-free-space-on-a-mapped-drive-using-linux) | 15. [Looking for WMI Object Persistence](#looking-for-wmi-object-persistence) |
| 6. [Mounting E01 files using Linux/WSL (EWFMount)](#mounting-e01-files-using-linuxwsl-ewfmount) | 16. [Password Cracking using John the Ripper](#password-cracking-using-john-the-ripper) |
| 7. [Image Conversion using QEMU-IMG](#image-conversion-using-qemu-img) | 17. [Repairing Zip Files and Checking Integrity](#repairing-zip-files-and-checking-integrity) |
| 8. [Open-VM-Tools](#open-vm-tools) | 18. [Password brute forcing with Hydra](#password-brute-forcing-with-hydra) |
| 9. [How to Mount Network Locations](#how-to-mount-network-locations) | 19. [Linux Changing File Permissions using Chmod](#linux-changing-file-permissions-using-chmod) |
| 10. [Setting Time In Linux](#setting-time-in-linux) | 20. [Linux Changing File Ownership using Chown](#linux-changing-file-ownership-using-chown) |

---

Several of the tools listed below reference using relative paths for the command syntax. To make the reference syntax function at a command prompt the Windows path statement needs to be modified to include these additional directory search locations. As a shortcut the PATH statement can be modified at the command prompt as shown below.

**NOTE:** To add a new application directory to the Windows path statement use: `path set=%path%;structureoffolderstoapplication` (or you can modify the environment variables within Windows; this change will require a reboot the adding paths at CMD doesn’t require a reboot but also doesn’t persist after a reboot).

An Example of a path statement: 
```cmd
path set=%path%;c:\forens~1\yara;c:\python32;c:\forens~1\tzworks;c:\forens~1\volat~1;C:\Users\Admin\AppData\Local\Programs\Python\Python~1\Lib\venv\scripts\nt;
```

Or `path set=%path%;c:\yara` – The percent marks tells the command prompt to append the new path after the `;` to the existing path statement. This can also be updated using the system environment variable within Windows.

The activities of a user can be profiled by gathering together the artifacts that are left behind from application or file execution. I’ve tried to group these different types of artifacts together, explaining what they are and how to work with them.

---

## Living off the land binaries and Libraries
* [LOLBAS Project](https://lolbas-project.github.io/) - Site dedicated to showing tools and tactics commonly used by attackers.
* [CrowdStrike Living off the Land Attacks](https://www.crowdstrike.com/cybersecurity-101/living-off-the-land-attacks-lotl/)

---

## Windows Logon Types and reasons
* Reference: [Microsoft Docs - Logon Types](https://learn.microsoft.com/en-us/windows-server/identity/securing-privileged-access/reference-tools-logon-types)
* Additional Reference: [FR Secure - RDP Connection Event Logs](https://frsecure.com/blog/rdp-connection-event-logs/)

### Column Definitions
* **Logon - type**: Identifies the logon type initiated by the connection.
* **Reusable - credentials on destination**: Indicates that the following credential types will be stored in LSASS process memory on the destination computer where the specified account is logged on locally:
  * LM and NT hashes
  * Kerberos TGTs
  * Plaintext password (if applicable)

**Symbol Definitions:**
* `(-)` denotes when credentials are not exposed.
* `(v)` denotes when credentials are exposed.

For management applications that are not in this table, you can determine the logon type from the logon type field in the audit logon events. For more information, see Audit logon events.

In Windows-based computers, all authentications are processed as one of several logon types, regardless of which authentication protocol or authenticator is used. This table includes most common logon types and their attributes relative to credential theft:

| Logon type | # | Authenticators accepted | Reusable credentials in LSA session | Examples |
| :--- | :---: | :--- | :--- | :--- |
| **Interactive** (also known as, Logon locally) | 2 | Password, Smartcard, other | Yes | Console logon; RUNAS; Hardware remote control solutions (such as Network KVM or Remote Access / Lights-Out Card in server); IIS Basic Auth (before IIS 6.0) |
| **Network** | 3 | Password, NT Hash, Kerberos ticket | No (except if delegation is enabled, then Kerberos tickets present) | NET USE; RPC calls; Remote registry; IIS integrated Windows auth; SQL Windows auth; PSExec, RDP (if NLA (Network Level Authentication) is enabled) |
| **Batch** | 4 | Password (stored as LSA secret) | Yes | Scheduled tasks |
| **Service** | 5 | Password (stored as LSA secret) | Yes | Windows services |
| **Screen Unlock** | 7 | Password, Smartcard, etc | | Reconnecting after screenlock has been activated |
| **NetworkCleartext** | 8 | Password | Yes | IIS Basic Auth (IIS 6.0 and newer); Windows PowerShell with CredSSP |
| **NewCredentials** | 9 | Password | Yes | RUNAS /NETWORK |
| **RemoteInteractive** | 10 | Password, Smartcard, other | Yes | Remote Desktop (formerly known as "Terminal Services") |

### Event Log Definitions:
* **Logon - type**: The type of logon requested.
* **#**: The numeric identifier for the logon type that is reported in audit events in the Security event log.
* **Authenticators - accepted**: Indicates which types of authenticators are able to initiate a logon of this type.
* **Reusable - credentials in LSA session**: Indicates whether the logon type results in the LSA session holding credentials, such as plaintext passwords, NT hashes, or Kerberos tickets that could be used to authenticate to other network resources.
* **Examples**: List of common scenarios in which the logon type is used.<br><br>
<img width="55%" alt="image" src="https://github.com/user-attachments/assets/79ee52ea-c9a3-459b-8b41-8b103a008fb3" /><br>


---

## Using Sudo with options
Below is a listing of options that can be used with Sudo.<br>
<img width="50%" alt="image" src="https://github.com/user-attachments/assets/dc47eb7b-d728-4a22-8709-34a5d4595afb" /><br>
---

## Mounting shares in Linux
Reference: [How-To Geek - Mounting Removable Drives and Network Locations in WSL](https://www.howtogeek.com/331053/how-to-mount-removable-drives-and-network-locations-in-the-windows-subsystem-for-linux/)

Let’s say you have an external drive `G:` that represents either a USB drive or an optical disc drive. To mount it, you’d run the following commands:
```bash
sudo mkdir /mnt/g
sudo mount -t drvfs G: /mnt/g
```

You don’t actually have to mount the drive at `/mnt/g`, of course. You can mount it wherever you want. Just replace both instances of `/mnt/g` in the commands with your desired path.

To unmount the drive later so you can safely remove it, run the standard `umount` command:
```bash
sudo umount /mnt/g/
```

When working with an external device formatted with a FAT file system or any network file system, there are some limitations. The file system will not be case sensitive and you can’t create symbolic links or hard links on it.

---

## Checking free space on a mapped drive using Linux
```bash
du -d 1 -h /mnt/sandbox/_Engagements | sort -hr
```
This will show the disk usage and sort it by largest to smallest.

---

## Mounting E01 files using Linux/WSL (EWFMount)
1. `cd /mnt`
2. `sudo mkdir ewf_mount`
3. Change the permissions so that we can mount to it without needing to do a `sudo`. This is needed because `/mnt` is a restricted area: `sudo chmod 777 /mnt/ewf_mount`
4. Next, is `ewfmount imagename.E01 /mnt/ewf_mount`
5. If the mount was successful you’ll see a message similar to “ewfmount 20140807”.
6. This will mount the image to a raw image mount similar to the one below. The “ewf1” is the raw mount of the image itself.
7. The next part is to make a place to mount the logical images: `sudo mkdir /mnt/logical_mount`
8. Set the permissions correct: `sudo chmod 777 /mnt/logical_mount`
9. Now, we need to view the partition table to see which partition we’re going to be mounting. To do this we’re going to use `mmls`. So the command is `mmls /mnt/ewf_mount/ewf1` - That should give us a result like what’s below.
10. Optionally, we could run `fdisk` instead of `mmls` this will give us a little different format but still much of the same info. However, now we also know the disklabel and disk identifier. The partition sizes are also listed in a friendlier format.

---

## Image Conversion using QEMU-IMG
* Website: [www.qemu.com](https://www.qemu.com)
* **QEMU:** A generic and open source machine emulator and virtualizer.

Simple conversion from QEMU qcow2 (`qc2`) file format to RAW:
```bash
qemu-img convert ./Diskimagefile.qc2 ./Diskimagefile.raw
```

---

## Open-VM-Tools
Reference: [Kali Linux Documentation - Install VMware Guest Tools](https://www.kali.org/docs/virtualization/install-vmware-guest-tools/)

Should you decide to create your own VMware installation of Kali Linux (rather than using our pre-made VMware images), and you want to force a manual reinstall of `open-vm-tools` (as something has gone wrong), first make sure you are fully updated, then enter the following:

```bash
sudo apt update
sudo apt install -y --reinstall open-vm-tools-desktop fuse
sudo reboot -f
```

### Adding Support for Shared Folders When Using OVT
Unfortunately, shared folders will not work out of the box, some additional scripts are needed. Those can be installed easily with `kali-tweaks`.

```bash
kali-tweaks
```
In the Kali Tweaks menu, select **Virtualization**, then **Install additional packages and scripts for VMware**. Congratulations, you now have two additional tools in your toolbox!

The first one is a little script to mount the VMware Shared Folders. Invoke it with:
```bash
sudo mount-shared-folders
```
And with a bit of luck, checking `/mnt/hgfs/` you should see your shared folders.

The second script is a helper to restart the VM tools. Indeed, it’s not uncommon for OVT to stops functioning correctly (e.g. such as copy/paste between the host OS and guest VM stops working). In this case, running this script can help to fix the issues:
```bash
sudo restart-vm-tools
```

---

## How to Mount Network Locations
You can also mount network locations. Any network location you can reach from within Windows, you can mount from the Linux shell.

Network locations can be mounted in one of two ways. If you map a network drive to a drive letter, you can mount it using the same options as above. This would give you an opportunity to easily sign into the network share and enter your credentials in File Explorer. For example, if your mapped network drive is `F:`, you could run the following commands to mount it:
```bash
sudo mkdir /mnt/f
sudo mount -t drvfs F: /mnt/f
sudo mount -t cifs //192.168.1.66/NTDS /mnt/c -o username=dma-user
```

You can also specify a drive using its UNC (Universal Naming Convention) path. For example, if the path to the network share is `\\server\folder`, you’d run the following command. Again, use whatever mount point you like in place of `/mnt/folder`.
```bash
sudo mkdir /mnt/folder
sudo mount -t '\\server\folder' /mnt/folder
```

---

## Setting Time In Linux
Setting Linux time:
```bash
sudo date -s "19 APR 2012 11:14:00"
```

---

## Using ClamAV for Linux to scan folder
```bash
sudo freshclam              # Updates the AV Signatures
clamscan -r /mnt/i/[root]/  # Recursively scans a folder path
```

---

## Changing / Setting the Default User for WSL Ubuntu
The “ubuntu” part of the command references the Linux distro that’s installed, depending on the version that’s installed this could be `ubuntu2004.exe` or even `ubuntu2204.exe`. You can identify the version by typing `wsl --list`.

> **NOTE:** This is useful when you don’t know the `su` password. You can change to the root user and then change the `su` password and then change back.

```bash
ubuntu config --default-user <username>
```

---

## Creating a Triage Image Using KAPE
* Reference: SANS 508 Workbook Exercise 3.2-7 or 508.3 Page 22

**Targeted Collection:**
```cmd
kape --tsource F --target RegistryHives --vhdx localhost --tdest G:\cdrive
```

**Full Triage Collection:**
```cmd
kape --tsource I --target !All --vhdx Server2013_TriageImage --tdest e:\Chem27\Server2012
```

**Collection Examples:**
```cmd
.\kape.exe --tsource E: --tdest F:\KapeTest-USG87 --tflush --target MiniTimelineCollection --vss --zip
kape.exe --tsource J: --tdest "F:\USG204\Disk Images\CITIAMFS3_files\kape" --target !BasicCollection --vss --zip
```

`Kape87Test –gui` (`MFT`, `Registry` and `EventLogs`) - QuickTimeline

**Processing:**
```cmd
.\kape.exe --msource F:\KapeTest-USG87\2020-06-10T202820_Kape87Test3 --mdest F:\KapeTest-USG87\MiniTimeline --mflush --module Mini_Timeline,Mini_Timeline_Slice_by_Daterange --mvars computerName:ARSHIHIL4059095^dateRange:01/01/2020-03/03/2020 --gui
```

---

## Listing Volume Shadow Copies
Good for listing the dates and times of copies to determine which VSS could hold information about an incident.

```cmd
vssadmin list shadows /for=D: > VSSAdminList_win7-c-drive.txt
```

---

## Looking for WMI Object Persistence
* Reference: [GitHub - davidpany/WMI_Forensics](https://github.com/davidpany/WMI_Forensics)

This repository contains scripts used to find evidence of persistence in WMI repositories, specifically `OBJECTS.DATA` files located at:
* `C:\WINDOWS\system32\wbem\Repository\OBJECTS.DATA`
* `C:\WINDOWS\system32\wbem\Repository\FS\OBJECTS.DATA`

### `CCM_RUA_Finder.py`
`CCM_RUA_finder.py` extracts SCCM software metering RecentlyUsedApplication logs from `OBJECTS.DATA` files. Then it outputs related rules.

**Usage:**
```bash
CCM_RUA_Finder.py -i path\to\OBJECTS.DATA -o path\to\output.xls
```
The output file will be TSV formatted. Excel will automatically parse TSV files with `.xls` extensions.

### `PyWMIPersistenceFinder.py`
`PyWMIPersistenceFinder.py` is designed to find WMI persistence via FilterToConsumerBindings solely by keyword searching the `OBJECTS.DATA` file without parsing the full WMI repository.

In testing, this script has found the exact same data as python-cim's `show_FilterToConsumerBindings.py` without requiring the setup. Only further testing will indicate if this script misses any data that python-cim can find.

In theory, this script will detect FilterToConsumerBindings that are deleted and remain in unallocated WMI space, but I haven't had a chance to test yet.

**Usage:**
```bash
PyWMIPersistenceFinder.py <OBJECTS.DATA file>
```
The output is text based in the following format for each binding:
```text
<consumer name>-<filter name>
             <optional notes>
        Consumer: <consumer name><consumer execution details>
        Filter: <filter name><filter listener details>
```

---

## Password Cracking using John the Ripper
The password cracking below is from an exercise that I did for the Network+ class, but I thought I’d include the steps as a good overview of the process. I’m pretty sure that the dictionary used had the passwords “added” to it to make sure that the exercise was a success. 

### 1. Crack the root password on Support
1. From the Favorites bar, select **Terminal**  
2. At the prompt, type `cd /usr/share/john` and press **Enter** to change directories to the folder containing the John the Ripper password file.  
3. Type `ls` and press **Enter** to list the files in the directory.  
4. Type `cat password.lst` and press **Enter** to view the password list. This is an abbreviated list.  
5. Type `cd` and press **Enter** to go back to root.  
6. Type `john /etc/shadow` and press **Enter** to crack the Linux passwords. Notice that the root password of `1worm4b8` was cracked.  
7. Type `john /etc/shadow` and press **Enter** to attempt to crack the Linux passwords again. Notice that it does not attempt to crack the password again. The cracked password is already stored in the `john.pot` file.  
8. Type `cat ./.john/john.pot` and press **Enter** to view the contents of the `john.pot` file.  
9. Type `john /etc/shadow --show` and press **Enter** as an alternate method of viewing the previously cracked password.  

### 2. Crack the password of the `protected.zip` file
1. From the Terminal prompt, type `ls` and press **Enter** to view the contents of the home directory. Notice the `protected.zip` file you wish to crack.  
2. Type `zip2john protected.zip > ziphash.txt` and press **Enter** to copy the hashes to a text file.  
3. Type `cat ziphash.txt` and press **Enter** to confirm that the hashes have been copied.  
4. Type `john --format=pkzip ziphash.txt` and press **Enter** to crack the password.  
5. Type `john ziphash.txt --show` and press **Enter** to show the password.  

---

## Repairing Zip Files and Checking Integrity
* Integrity check on zip files with:
  ```bash
  zip -T filename.zip
  ```
* Repairing a Zip file works great for Plaso files, not so good for Iris files:
  ```bash
  zip -FF broken.plaso --out fixed.plaso
  ```

---

## Password brute forcing with Hydra
Here you will leverage the hydra to try and break into the webserver using the `taugratin` account. You will also feed the tool a password list and tell it to target the webserver for SSH access. See how the password list is very “tator”-oriented.

**Execute the following commands:**
1. Make sure the directory is on Desktop:
   ```bash
   cd /home/kali/Desktop
   ```
2. Now find the password list:
   ```bash
   hydra -l taugratin -P pwlist.txt 172.16.193.171 ssh
   ```

---

## Linux Changing File Permissions using Chmod
*(Notes section placeholder in original document)*

---

## Linux Changing File Ownership using Chown
*(Notes section placeholder in original document)*
