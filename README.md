#  Migrate VMware machine to Proxmox VE 
![](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTfl4UoFvHn9M4mdhpcJL_uAXgQ4WHNLbVNRkBRS8V0LDq0jITBZC12xwXaYbQ1TzTOOD8&usqp=CAU)

[MITT NSA](https://mitt.ca/programs/post-secondary-programs/2385/network-and-systems-administrator-diploma) Winter 24P2\
130 Henlow Bay, Winnipeg, MB R3Y 1G4\
(204) 989-6500\
*Version #1 of document*\
Written by:  **Ching-An Hu** **Ching-Chuan Hu**\
Approved by: **Rogelio Villaver Jr**\
Date: 03/05/2025

## Purpose
This Standard Operating Procedure (SOP) is designed  to provide a clear and repeatable process for migrating **Virtual Machines (VMs)** from an existing **VMware ESXi** or **local VMware** to **Proxmox Virtual Environment (VE)**. 
This ensures minimal downtime, proper configuration, and compatibility with the target system while maintaining best practices for academic or lab environments.

The procedure aims to facilitate:
  * The procedure ensures VMs are migrated from a source VMware to Proxmox VE with minimal errors or data loss, maintaining operational continuity.
  * By offering options like live-import and efficient manual methods (e.g., disk cloning), the procedure minimizes the time VMs are offline, which is critical for production-like lab environments.
  * Verification steps and post-migration tasks ensure the migrated VMs function correctly, facilitating a stable and reliable outcome.
    
## Application
This SOP applies to students, lab assistants, or IT trainees tasked with migrating VMs in a controlled environment, such as a lab or a virtualized testing setup. 
It covers both manual and automatic migration methods for VMs running Linux or Windows operating systems.

It is relevant in circumstances such as:
 * Upgrading Virtualization Platforms in a Lab Environment.
 * Replacing End-of-Life Hypervisors.
 * Testing New Virtualization Software.

## Definitions
**VM (Virtual Machine)** : A software-based emulation of a physical computer running an operating system and applications.\
**Proxmox VE** : An open-source virtualization platform based on QEMU/KVM and LXC.\
**Hypervisor** : Software that creates and manages VMs (e.g., VMware ESXi or Proxmox VE).\
**VirtIO** : Paravirtualized drivers that improve VM performance on compatible systems.\
**Live-Import** : A migration method that starts the VM during the import process to reduce downtime.\
**OVF/OVA** : Open Virtualization Format, a standard for packaging and distributing VMs.\
**ESXi** : VMware’s type-1 hypervisor used as a common migration source.


## Hardware requirements for Proxmox Virtual Environment
|Components     |Minimum Requirements    |
|---------------|------------------------|
|CPU            |Intel 64 or AMD64 with Intel VT/AMD-V CPU flag. | 
|RAM            |Minimum 2 GB for OS and Proxmox VE services. Plus designated memory for guests. For Ceph or ZFS additional memory is required, approximately 1 GB memory for every TB used storage. |
|OS Storage        |Fast and redundant storage, best results with SSD disks. Hardware RAID with batteries protected write cache (“BBU”) or non-RAID with ZFS and SSD cache.|
|VM Storage        |For local storage use a hardware RAID with battery backed write cache (BBU) or non-RAID for ZFS. Neither ZFS nor Ceph are compatible with a hardware RAID controller. Shared and distributed storage is also possible.|
|Network        |Redundant Gbit NICs, additional NICs depending on the preferred storage technology and cluster setup – 10 Gbit and higher is also supported. |
|Connect|For PCI(e) passthrough a CPU with VT-d/AMD-d CPU flag is needed.|

[More informations from Proxmox offical website](https://www.proxmox.com/en/products/proxmox-virtual-environment/requirements).

## Procedure steps
### Download the Windows Server 2022 ISO
1. Download the Windows Server 2022 ISO from the [Microsoft Evaluation Center](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2022).  
2. Click on the “Download the ISO” button.
3. Fill in your information in the form and click “Download”.
4. Select “Windows Server 2022 Standard” or “Windows Server 2022 Datacenter” base on purpose.
5. Click the “Confirm” button. The ISO download will be start.

Table for the directly download links:

|Language                | ISO Downloads                                                                                                | VHD download                                                                                                 |
|------------------------|--------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------|
|English (United States) | ISO [64-bit edition](https://go.microsoft.com/fwlink/p/?LinkID=2195280&clcid=0x409&culture=en-us&country=US) | VHD [64-bit edition](https://go.microsoft.com/fwlink/p/?linkid=2195166&clcid=0x409&culture=en-us&country=us) |
|French                  | ISO  [64-bit edition](https://go.microsoft.com/fwlink/p/?LinkID=2195280&clcid=0x40c&culture=fr-fr&country=FR) |                                                                                                             |

### Install Windows Server 2022 on VMWare Workstation
1.  [Download](https://www.techspot.com/downloads/downloadnowfile/189/?evp=4c50cf08866937ea246522b86f4d4286&file=241), install and open the VMWare workstation.
2.   Click on **Create a New Virtual Machine** button.
3.   VMWare Workstation will be greeted with two options, Typical and Custom. Choose **Typical** with the default recommended settings.
4. Chose the third option, install in interactive mode (More options to choose during the installation process.)
5. Choose **Microsoft Windows Server 2022** as the guest operating system.
![](https://www.datocms-assets.com/104397/1708651058-choose-the-microsoft-windows-server-2022.png?auto=format&dpr=4)
6. Give a name for the VM and choose the location to store the VM.
7. Specify the disk capacity according to requirements, and choose the options to save the disk as split storage or single storage.
8. That completes the VM configuration.  Take a review and go for customization if needed or click on the **Finish** to complete the VM setup process.
9. Click on the **CD Rom** and select the Windows Server 2022 ISO file.
10. Power on the VM.
11. When VM booted, it asked to press any key to boot from the CR-ROM. Hit **Enter**.
![](https://www.datocms-assets.com/104397/1708651104-boot-from-the-cd-rom.png?auto=format&dpr=3)
12. Windows will boot from the CD-ROM with ISO and ask for the installation. Click **Next**.
13. Click **Install now**.
14. Choose either **Standard** or **Datacenter** editions. Click **Next**.  ※Desktop Experience with GUI and CLI version with powershell.
![](https://www.datocms-assets.com/104397/1708651127-choose-windows-version-to-install.png?auto=format&dpr=3)
15. Read and **Accept the License Agreement** to continue. Click **Next**.
16. Windows asks for the installation type and installation drive. Choose the second option **Custom**.
17. Choose the drive to install, Click **Next**
18. Installation will begin start.
![](https://www.datocms-assets.com/104397/1708651164-installation-is-in-progress.png?auto=format&dpr=3)
19. The VM reboots upon the installation process. Click **Reboot now**
![](https://www.datocms-assets.com/104397/1708651172-reboot-windows-server-2022-in-few-seconds.png?auto=format&dpr=3)
20. After the reboot process. Windows will ask to create an Administrator account. Enter the Password and Click the **Finish** to finish the installation.
![](https://www.datocms-assets.com/104397/1708651179-create-administrator-account.png?auto=format&dpr=3)
21. The installation process completes. Hit the **Ctrl+Alt+Delete** keys to log in.
22. Windows Server 2022 has been successfully installed on VMware.
![](https://www.datocms-assets.com/104397/1708651196-windows-server-2022-desktop.png?auto=format&dpr=3)

### Change the Time Zone
1. Open **Settings**.
2. Go to **Time & Language**.
3. Select **Date & time**.
4. Turn off **Set time zone automatically**.
![](https://anakage.com/blog/wp-content/uploads/2022/01/image2-1.jpg)
5. Go to the **Time zone** drop-down menu and choose correctly time zone.
![](https://anakage.com/blog/wp-content/uploads/2022/01/image3-1.jpg)

### Change the hostname
1. Open **Server Manager** from the **Start Menu**. 
2. In Server Manager, select Local Server in the left-hand menu followed by selecting the current name of your server Next to **Computer name**.
3. In the System Properties window, under the Computer Name tab, **select Change**.
![](https://s3.amazonaws.com/cdn.freshdesk.com/data/helpdesk/attachments/production/11120332911/original/eNZbU-hRYSk_Fr983dmaI921GoOMhStwSw.png?1728339765)
4. Enter your desired server name in the Computer name field, then select **More**.
![](https://s3.amazonaws.com/cdn.freshdesk.com/data/helpdesk/attachments/production/11120332914/original/uooywpVvXjcUNFgM4ultSjLxpj3SalFr8A.png?1728339798)
5. If required, enter your Primary DNS for your server and select **OK**.
6. Restart the server to apply the changes. Select **OK**.
7. Once your system restarts, open Server Manager again to verify that the server name has been successfully updated.
![](https://s3.amazonaws.com/cdn.freshdesk.com/data/helpdesk/attachments/production/11120333033/original/LhpfOZN4R_GaWTLOUH0bsg_cph8MIB6SmQ.png?1728340022)

## References
1. [Windows Server 2022 | Microsoft Evaluation Center](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2022)
2. [Hardware requirements for Windows Server](https://learn.microsoft.com/en-us/windows-server/get-started/hardware-requirements?tabs=cpu&pivots=windows-server-2025)
3. [How to Create a Windows Server 2022 Virtual Machine With VMware](https://www.youtube.com/watch?v=II-a79HFQtQ)
4. [How to Change Hostname/Server Name in Windows Server 2022](https://support.binarylane.com.au/support/solutions/articles/11000129152-how-to-change-hostname-server-name-in-windows-server-2022)
5. [Windows Server 2022 Change Time Zone Greyed Out](https://www.ichi.co.uk/blog/windows-server-2022-change-time-zone-greyed-out)
