# Prerequisites
Recording prerequisites to enable SR-IOV on an ***Intel x86 platform with Linux system***, including supporting for the VM and the container scenarios.

## SR-IOV Introductions

### What it is
Single root I/O virtualization (SR-IOV) is an extension to the PCI Express (PCIe) specification. SRIOV enables a single PCIe device to appear as multiple, separate devices. Traditionally in a virtualized environment, a packet has to go through an extra layer of the hypervisor, that results in multiple CPU interrupts per packet. These extra interrupts cause a bottleneck in high traffic environments. SR-IOV enabled devices, have the ability to dedicate isolated access to its resources among various PCIe hardware functions. These functions are later assigned to the virtual machines which allow direct memory access (DMA) to the network data. This enables efficient sharing of PCIe devices, optimization of performance and reduction in hardware costs.

### Architecture
![](https://www.servethehome.com/wp-content/uploads/2015/11/Intel-SR-IOV-Diagram.jpg)

**Physical Function (PF):**

Physical Function is a full-featured PCIe function of a network adapter that is SR-IOV capable. Physical Functions are discovered, managed, and configured as normal PCIe devices. PCIe devices have a set of registers known as configuration space. Physical Function behaves like an L2 switch and performs traffic forwarding between physical port and Virtual Functions.

**Virtual Function (VF):**

Virtual Functions (VFs) are simple PCIe functions that lack the configuration resources and only have the ability to move data in and out. Each Virtual Function is associated with a PCIe Physical Function. Each VF represents a virtualized instance of the network adaptor and has a separate PCI Configuration space. These VFs are assigned to virtual machines later.

**I/O MMU:**

Input–Output Memory Management Unit (IOMMU) connects a Direct-Memory-Access–capable (DMAcapable) I/O bus to the main memory. The IOMMU maps device addresses or I/O addresses to physical addresses. IOMMU helps in accessing physical devices directly from virtual machines.

**Hypervisor:**

A hypervisor is a software that allows running multiple virtual machines to share a single underlying hardware platform. In case of SR-IOV, either the hypervisor or the guest OS must be aware that they are not using full PCIe devices. Hypervisor maps VF’s configuration space to the configuration space presented to the guest using IOMMU.

**Virtual Machine:**

A VF can only be assigned to one virtual machine at a time however, a virtual machine may have multiple virtual functions associated with it. VF appears as a single network interface card inside of a virtual machine. The virtual machine must be aware of using VF in case the hypervisor is not aware of using an SR-IOV enabled adapter.

### Features

**Wire-speed performance:**

SR-IOV takes a physical port (Physical Function or PF) and logically slices it into Virtual Functions (VFs). Each VF can be programmed to provide a certain amount of bandwidth to the Virtual Network Function (VNF) connected to it. It is the primary objective of the SR-IOV facility provided in this solution to enable
"end-to-end" wire-speed performance to the extent possible.

**Fail-over support:**

Sometimes referred to as high-availability (HA), the SR-IOV implementation in this solution supports physical port, wire, NIC, and first-level switching element failover. If any of these elements fails, connectivity to the Virtual Functions (a pair of VFs is always exposed to a VNF) is not compromised. Application-level HA models are overlayed on top of the SR-IOV connectivity and can be independently implemented based on the needs of the VNFs and the service function chains within which they might be individual links.

**Co-existence with OpenStack platform native networks:**

The SR-IOV implementation does not attempt to merge with OpenStack networks. The built-in tenant and external networks supported by the OpenStack are untouched. Individual VNF instances are created, managed, and lifecycle-managed by the OpenStack Platform. The SR-IOV networks are additional networks that co-exist with the OSP networks and provide a direct external path into and out of the VNFs. This makes them an excellent choice for functions typically placed at traffic ingress/egress points of an NFV solution, such as a firewall, a session-border controller, or an application-delivery controller.

**Separate network resources:**

The SR-IOV implementation requires separate NICs, SPF+ connectors, cables and switch ports to carry SR-IOV enabled traffic. The SR-IOV networks are not routed through the Master Controller of the OpenStack Platform. This requires the use of a shared resource (the virtual network router) in the Master Controller, and disrupts the end-to-end wire-speed performance to and from a VNF.

## Requirements
**Notice:** Contents in this and all the following sections are referring to 


- [https://software.intel.com/en-us/articles/configure-sr-iov-network-virtual-functions-in-linux-kvm](https://software.intel.com/en-us/articles/configure-sr-iov-network-virtual-functions-in-linux-kvm)
- [https://www.intel.com/content/dam/www/public/us/en/documents/technology-briefs/xl710-sr-iov-config-guide-gbe-linux-brief.pdf](https://www.intel.com/content/dam/www/public/us/en/documents/technology-briefs/xl710-sr-iov-config-guide-gbe-linux-brief.pdf).

### Supported Intel NICs
A complete list of Intel Ethernet Server Adapters and Intel® Ethernet Controllers that support SR-IOV can be found by ***Which Intel® Ethernet Adapters and Controllers support SR-IOV?*** at [https://www.intel.com/content/www/us/en/support/articles/000005722/network-and-i-o/ethernet-products.html](https://www.intel.com/content/www/us/en/support/articles/000005722/network-and-i-o/ethernet-products.html).

### Supported Hypervisors on Intel Ethernet Adapters
- Microsoft Hyper-V* (Windows Server 2012*)
- VMware Sphere* 5.1
- Xen Hypervisor*
- KVM* (Kernel Based Virtual Machine)


### Guest OS with Available VF Drivers
- Windows Server 2012*
- Windows Server 2012* R2
- Windows Server 2008* R2
- Windows Server 2008*
- Linux* 2.6.30 kernel or later
- Red Hat Enterprise Linux 6.0* and later
- SUSE Linux Enterprise Server 11* SP1 and later

### Hardware Requirements
- Supported Intel Ethernet Adapters.
- A server platform that supports Intel® Virtualization Technology for Directed I/O (VT-d) and the PCI-SIG* Single Root I/O Virtualization and Sharing (SR-IOV) specification.
- A server platform with an available PCI Express*: x8 5.0Gb/s (Gen2) or x8 8.0Gb/s (Gen3) slot.

### Software Requirements
- OS supporting SR-IOV (RHEL 7).
- Drivers supporting PF and VF of corresponding network adapters. Drivers can be found by ***Where can I get the virtual function (VF) drivers?*** at [https://www.intel.com/content/www/us/en/support/articles/000005722/network-and-i-o/ethernet-products.html](https://www.intel.com/content/www/us/en/support/articles/000005722/network-and-i-o/ethernet-products.html).


## Installation and Configuration

### Server Setup
**Notice:** Only key steps are listed here.

- Enable IOMMU support by appending `intel_iommu=on` to the `GRUB_CMDLINE_LINUX` entry in `/etc/default/grub` configuration file.
- Update grub configuration using `grub2-mkconfig` command and reboot the server for the `iommu` change to take effect.
- Create VF, for linux kernel version > 3.8.x, by writing an appropriate value to the `sriov_numvfs` parameter via `sysfs` interface. For example: `echo 4 > /sys/class/net/<device name>/device/sriov_numvfs` which means creating 4 VFs on the port indicated by the `device name`.
- By `echo 0 > /sys/class/net/<device name>/device/sriov_numvfs` the created VFs could be deleted.
- Write the VF creation command to the `/etc/rc.d/rc.local` file to make sure the VFs are created each time the server reboots.
- The maximum number of VFs supported by the adapter can be queried by reading the `sriov_totalvfs` parameter via `sysfs` interface. For example: `cat /sys/class/net/device name/device/sriov_totalvfs`.
- Use the `lspci` command to confirm that the VF was successfully created.
- Manually set valid MAC addresses to created VFs if automatically VF MAC addresses assignment by such as `LibVirt` or `Virtual Machine Manager` is not wanted. For example: `ip link set <device name> vf 0 mac aa:bb:cc:dd:ee:ff`.
- Write the VF MAC addresses assigning commands to the `/etc/rc.d/rc.local` file to ensure each VF carries the same MAC address assignment from one boot to the next.




