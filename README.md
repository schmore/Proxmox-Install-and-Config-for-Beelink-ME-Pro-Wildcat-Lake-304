# Proxmox-Install-and-Config-for-Beelink-ME-Pro-Wildcat-Lake-304
Documenting my setup for running Home Assistant OS, a Debian 13 VM (primarily for Docker (jellyfin+*arr, Immich, Komodo, ...) with iGPU/NPU passthrough, and OpenMediaVault 

Lots of issues to deal with related to drivers and some hardware limitations. 

I used Gemini quite a bit in the process of troubleshooting so much of the scripts/code are written partially or entirely by AI but guided by a real human (and linux/homelab noob) the whole way.

##Driver issues: Wildcat Lake iGPU/NPU 
The Wildcat lake CPU defaulted to the i915 drivers that didn't work correctly for me when trying igpu/npu passthrough. Installed the Xe series drivers manually and was successful with pcie passthrough afterwards (transcoding for jellyfin, reencoding with handbrake, Immich machine learning features, etc). See separate doc for install details and igpu/npu passthrough setup.

## Driver Issues: Realtek Semiconductor Co., Ltd. RTL8127 10GbE Controller
This controller was not working at all when I first checked it, which was after I had already converted everything to proxmox. There are 3 network interfaces:

lspci -nnk
...
57:00.0 Ethernet controller [0200]: Intel Corporation Ethernet Controller I226-V [8086:125c] (rev 04)  
        Subsystem: Intel Corporation Device [8086:0000]  
        Kernel driver in use: igc  
        Kernel modules: igc  
58:00.0 Ethernet controller [0200]: Realtek Semiconductor Co., Ltd. RTL8127 10GbE Controller [10ec:8127] (rev 05)  
        Subsystem: Realtek Semiconductor Co., Ltd. Device [10ec:0123]  
        *Kernel driver in use: r8169*  
        *Kernel modules: r8169*  
59:00.0 Network controller [0280]: MEDIATEK Corp. MT7922 802.11ax PCI Express Wireless Network Adapter [14c3:7922]  
        Subsystem: AzureWave Device [1a3b:5911]  
        Kernel driver in use: mt7921e  
        Kernel modules: mt7921e

The 2.5GbE port (57:00.0) was working fine but note the mismatched driver for the realtek controller (58:00.0). Manually installing the correct drivers directly downloaded from Realtek fixed the issue and let me connect to the 10GbE card at full speed on my larger NAS directly. See separate document for details on the install and setting up the network bridge and openmediavault VM connection to Unraid in proxmox using the realtek controller and 10G network card on the Unraid box.

##ASMedia 1062 SATA Controller does not support PCIe passthrough (lacks function level reset)
I wanted to run Unraid in a VM to manage my 2x8TB SATA drives and then point my other VMs/etc. towards Unraid. I had schemes to add more disks (JBOD setup) with an M.2 to SATA port expander but gave up on the idea after going through this process. Ultimately I used Proxmox's native ZFS features instead and it worked fine but I needed more than 2 bays to store my collection of linux isos. 

I spent a long time troubleshooting the passthrough of the ASMedia 1062 SATA controller so Unraid could properly manage the drives. I found the controller is ultimately not compatible with passthrough in this hardware configuration. As I understand it, the controller needs to be reset when handed off to the VM from the host and this chip does not support this operation. I tried a number of workarounds and nothing was successful but I'm leaving some of my troubleshooting notes for reference.

#
