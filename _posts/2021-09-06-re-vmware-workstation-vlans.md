---
title: "RE: VMWare Workstation VLANs"
date: 2021-09-06 23:05:00 -0500
classes: wide
categories:
  - linux
  - vmware
tags:
  - linux
  - vmware
  - networking
  - vlans
  - virtualization
---

[Last year](/linux/vmware/using-vlans-with-vmware-on-a-linux-desktop/) I wrote about using VLANs with VMWare Workstation with systemd-networkd.<br>
A year later, I'm still actively using this process in my day to day workflow but I'm using a much improved configurating leveraging NetworkManager and nmcli.

Placing my VMs within specific VLANs ensures my network traffic has restrictions commensurate with its level of risk.<br>
For example, capture the flag VMs have unrestricted internet access but zero access to the LAN, meanwhile my administrative VM is allowed to access my network infrastructure and servers, but can't reach the internet.

## Configuration

### Pre-Requisites

Before doing this, you'll need to already have a network set up that has fully functioning VLANs in place on the routers and switches.<br>
Additionally, the connection to the desktop you're working on will need to have a VLAN trunk link.<br>
This link must support all the tags you anticipate having virtual interfaces for on the desktop.<br>
For example, if you anticipate the desktop host operating on VLAN 10 and your virtual machines in VLAN 20 and 30, you'll need to ensure your desktop's network link is a trunk link supporting VLANs 10, 20, and 30.

### Creating Virtual Desktop Inferfaces

We'll need to configure a VLAN-tagged virtual interface for each VLAN we wish to be able to bridge to VMWare Workstation.

```bash
# Syntax:
nmcli con add type vlan ifname vlan<vlan_id> con-name vlan<vlan_id> dev <trunk_iface> id <vlan_id>
# Example:
nmcli con add type vlan ifname vlan10 con-name vlan10 dev enp0s12a3 id 10
```

Next we'll disable IP addressing on the new virtual interface.<br>
Because the interface will be bridged, the attached virtual machines can pull an IP address from the network without needing one on the host.<br>
Also, not assigning an address to your host reduces your host attack surface by not directly exposing it on the same network as the bridged VMs.

```bash
nmcli con mod vlan<vlan_id> ipv4.method disabled ipv6.method disabled
```

Bounce the interface to drop the address.

```bash
nmcli con down vlan<vlan_id>
nmcli con up vlan<vlan_id>
```

### Reconfigure Trunk Interface

This part is optional, but you can place your host machine in a specific VLAN as well by disabling IP addressing on the trunk interface, and leaving addressing on for a VLAN-tagged virtual interface. (Process above, without the second step)

```bash
nmcli dev mod <trunk_iface> ipv4.method disabled ipv6.method disabled
```

### Creating VMWare Virtual Networks

Start by opening up the VMWare Virtual Network Editor.<br>
You can open it through the desktop GUI or via `sudo vmware-netcfg`.<br>
If you're using VMWare Player, check out the [Caveats](#caveats) section for a workaround.

If you haven't edited anything, you'll probably have three `vmnet#` interfaces already made.<br>
`vmnet0` is an interface that is bridged to all other interfaces on your host.

You'll want to start by selecting `vmnet0`, and changing the device that it is bridged to from `Automatic` to your host interface.<br>
Next, begin adding networks (e.g. `vmnet20`) for each of your vlans, setting them to `Bridged` and bridging them to the appropriate virtual interface (e.g. `vlan20`).

![VMWare Virtual Network Editor](/assets/images/vmware-netcfg-vlan-2.png)

### Working With Virtual Machines

To assign a VM to one of your new virtual networks, just assign a network adapter to one of the VMNets you created in the `custom` section.

![Virtual Machine VLAN Assignment](/assets/images/vmware-vm-netdev-2.png)

## Caveats

> What does this do for security?

I'll be honest, this is a bit of a double-edged sword.<br>
For your virtual machines, it allows you greater control over network resources by ensuring the VMs are placed into an appropriate VLAN.<br>
This can be very helpful in the event your virtual machine is compromised, restricting, maybe even isolating the attacker in your desired network space.<br>
However, your host machine is now operating on a network trunk link.<br>
If your host were to ever be compromised or (less likely) an attacker were able to escape a VM up to the host machine, they now have access to every VLAN on your trunk, unimpeded by any semblence of network segmentation.<br>
This may be an acceptable trade-off for some, as the risk of a VM compromise may be higher than a host compromise.

> What if I have VMWare Player instead of VMWare Workstation?

```bash
cd /usr/lib/vmware/bin

ln -s /usr/lib/vmware/bin/appLoader vmware-netcfg

ln -s /usr/lib/vmware/bin/vmware-netcfg /usr/bin/vmware-netcfg
```

After creating these links, you should be able to access the VMWare Virtual Network Editor. [[1](#references)]

## References

1. Michael Gr. contributed a clever fix to the [Debian documentation](https://wiki.debian.org/VMware#Running_vmware-netcfg_.28Virtual_Network_Editor.29_with_VMware_Player).
This should allow users of VMWare Player to use the vmware-netcfg utility.
