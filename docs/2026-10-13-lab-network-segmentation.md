# 2026-10-13 Lab network segmentation in Proxmox

## Goal

Build isolated virtual networks inside Proxmox so lab traffic wont be able to reach the house network, and so the domain controller, the attacker box, and the honeypot each have their their own segment.

## What I did

1. Decided on 10.10.x.0/24 addressing for everything I build, with the third octet matching the VLAN ID.
2. Kept the house network separate on 192.168.1.0/24.
3. Put together exactly how I wanted to segment the network and came up with eight segments: management, trusted, IoT, guest, services, lab servers, lab attacker, and lab honeypot.
4. Wrote out a draft of the access rule table, default deny, with asymmetric rules like trusted can reach IoT but IoT cannot reach trusted.
5. Decided to start with only the three lab segments, since I do not have the hardware for house segments yet.
6. Wrote the plan as `docs/network-plan.md` and linked it in the README.
7. Went to pve01, System, Network in Proxmox to look at the starting state.
8. Found four interfaces: nic0 active as vmbr0's slave, nic1 inactive, vmbr0 holding the management IP with the house gateway, and wlp0s20f3 as the inactive WiFi card.
9. Created three Linux bridges with no IP, no gateway, and no bridge ports.
10. Left vmbr0 completely untouched.
11. Read the diff of `/etc/network/interfaces` before applying and confirmed the vmbr0 lines had no plus or minus in front of them.
12. Applied the configuration.

## Configuration

|Bridge|Comment|IP|Bridge ports|
|-|-|-|-|
|vmbr60|Lab servers VLAN 60|none|none|
|vmbr70|Lab attacker VLAN 70|none|none|
|vmbr80|Lab honeypot VLAN 80|none|none|

All three came Active with Autostart enabled. The Ports/Slaves column was empty for all three, since none of them have a physical path out. vmbr0 kept its address and the web interface stayed reachable the whole time.

Planned segments, from the network plan:

|VLAN|Name|Subnet|
|-|-|-|
|10|Management|10.10.10.0/24|
|20|Trusted|10.10.20.0/24|
|30|IoT|10.10.30.0/24|
|40|Guest|10.10.40.0/24|
|50|Services|10.10.50.0/24|
|60|Lab servers|10.10.60.0/24|
|70|Lab attacker|10.10.70.0/24|
|80|Lab honeypot|10.10.80.0/24|

## Problems hit

None. Thankfully the process went smoothly. Proxmox staged the network changes as pending instead of applying them, so I was able to read the diff and confirm nothing touched my management interface before committing.

## What I learned

1. A bridge in Proxmox is a virtual switch, and that a switch can only forward traffic out of ports that are attached to it. When I created vmbr60, vmbr70, and vmbr80 I left the bridge ports field blank, so none of them have a physical NIC attached. That meant there was no port a packet could exit through to reach my router or the house network. VMs plugged into the same bridge can still talk to each other, because they are on the same switch, but nothing can get off that switch until something routes between them.
2. I kept management on vmbr0 because that is the only interface with an IP address and a route to the house network, which would make it the only way my desktop could reach the Proxmox web interface. If I moved management onto a lab bridge or broke vmbr0, the web UI would go dark and I would be fixing it from the laptop's physical console with no desktop to work in.

## Next

* Phase 5: build OPNsense as a VM with an interface on vmbr0 for internet and one on each lab bridge, which would make it the only route between segments
* Write the firewall rules from the access rule table in the network plan

