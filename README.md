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

## Manual migration procedure steps
### Source VM Preparation
1. Document the test VM’s configuration (CPU, memory, disk, network) for reference.
2. Record the VM’s network settings (IP address, MAC address, DHCP details)..  
3. For Windows VMs, remove static IP configurations to avoid conflicts post-migration.
4. If full-disk encryption is used, ensure manual decryption keys are available (vTPM migration is not supported).
5. In the **File** section, click **Export to OVF** and save the .OVF file
![image](https://github.com/user-attachments/assets/2e6c0342-0dc4-4832-ac88-c45e9ffbe217)

### Target Environment Check
1.Verify that Proxmox VE is version 8 or higher and fully updated (via the web console interface: Datacenter > Updates).
2.Ensure sufficient storage space is available on the target Proxmox node.

### Configure Target VM 
1. Go to the Proxmox web console interface, select a node, and click **Create VM**.
2. Assign a unique VMID and name.
![image](https://github.com/user-attachments/assets/8610ea7f-4fa6-443d-b2b5-ef9497ad1cbd)
3. In the **OS** section, click **Do not use any media**. 
4. In the **System** section, keep that all options are set as default.
5. In the **Disks** section, keep that all options are set as default.
6. In the **CPU** section, make sure the cpu type choose **x86-64-v2-AES** , the other settings can be adjusted based on the VM to be migrated.
![image](https://github.com/user-attachments/assets/04ab5fa7-89aa-43b6-8e3f-6b24daaf6132)
7. In the **Memory** section, the settings can be adjusted based on the VM to be migrated. Enable the **Ballooning Device** in the **Advanced** settings to monitor memory usage.
![image](https://github.com/user-attachments/assets/3da3123d-14ad-45fa-9229-4084868b8ec8)
8. Network Configuration, Add a network device with the **VirtIO** model for best performance. Connect it to the appropriate network bridge (e.g vmbr0).
9. Click **Finish** and the VM will be ready to use.
10. Go to the VM you just create, click **Hardware**.
11. Select **Hard disk(xxx)** and click **Detach**, click **Yes**.
![image](https://github.com/user-attachments/assets/38efc4e1-9750-4d4f-8251-f0323768f175)








## References
1. [Windows Server 2022 | Microsoft Evaluation Center](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2022)
2. [Hardware requirements for Windows Server](https://learn.microsoft.com/en-us/windows-server/get-started/hardware-requirements?tabs=cpu&pivots=windows-server-2025)
3. [How to Create a Windows Server 2022 Virtual Machine With VMware](https://www.youtube.com/watch?v=II-a79HFQtQ)
4. [How to Change Hostname/Server Name in Windows Server 2022](https://support.binarylane.com.au/support/solutions/articles/11000129152-how-to-change-hostname-server-name-in-windows-server-2022)
5. [Windows Server 2022 Change Time Zone Greyed Out](https://www.ichi.co.uk/blog/windows-server-2022-change-time-zone-greyed-out)
