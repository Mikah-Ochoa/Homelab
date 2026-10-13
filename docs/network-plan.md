# Network Plan

Design document for network segmentation. Lab segments are built first as virtual networks inside Proxmox. House segments come later, once the hardware is in place.

## Addressing scheme

The house network stays on 192.168.1.0/24, which is the Spectrum default. Everything I build uses the 10.10.x.0/24 range so a lab or segmented address never looks like a house address at a glance.

Conventions:

- The third octet matches the VLAN ID, so VLAN 30 lives on 10.10.30.0/24
- The gateway is always .1 on each segment
- Static addresses for servers run from .10 to .50
- DHCP hands out .100 to .200

## Segments

| VLAN | Name | Subnet | Gateway | What lives here |
|---|---|---|---|---|
| 10 | Management | 10.10.10.0/24 | 10.10.10.1 | Switch and router admin interfaces |
| 20 | Trusted | 10.10.20.0/24 | 10.10.20.1 | My PC, phones, family devices |
| 30 | IoT | 10.10.30.0/24 | 10.10.30.1 | Smart TVs, Ring cameras |
| 40 | Guest | 10.10.40.0/24 | 10.10.40.1 | Visitors and untrusted devices |
| 50 | Services | 10.10.50.0/24 | 10.10.50.1 | Plex and other self hosted apps |
| 60 | Lab servers | 10.10.60.0/24 | 10.10.60.1 | Windows domain controller, workstation VM |
| 70 | Lab attacker | 10.10.70.0/24 | 10.10.70.1 | Kali |
| 80 | Lab honeypot | 10.10.80.0/24 | 10.10.80.1 | Raspberry Pi running OpenCanary |

Proxmox management stays on the house network at 192.168.1.185 so I can always reach the web interface, even if I break a lab network.

## Access rules

Default is deny. Anything not listed is blocked.

| From | To | Allowed | Why |
|---|---|---|---|
| Trusted | All segments | Yes | My devices need to manage everything |
| Trusted | Internet | Yes | Normal use |
| IoT | Internet | Yes | Cameras and TVs need to phone home |
| IoT | Any internal segment | No | A compromised camera should reach nothing |
| IoT | Services | Yes, Plex ports only | TVs need to find the Plex server |
| Guest | Internet | Yes | Visitors need working WiFi |
| Guest | Any internal segment | No | Untrusted by definition |
| Services | Internet | Yes | Updates and metadata |
| Lab segments | Any house segment | No | The lab never touches the house |
| Lab attacker | Lab servers | Yes | This is the point of the lab |
| Lab servers | Lab attacker | No | Keeps the attack direction one way |
| Lab attacker | Lab honeypot | Yes | Needed to trigger honeypot alerts |

The firewall is stateful, so an allow rule in one direction lets replies come back without opening the reverse direction. That is what makes "my phone can reach the camera but the camera cannot reach my phone" possible with a single rule.

## Known problems to solve

- Device discovery (mDNS and broadcast traffic) does not cross subnets. Chromecast, AirPlay, and Plex auto discovery will break between segments. Fix is an mDNS reflector on the router, or pointing clients at the server IP manually.
- Some Ring cameras behave badly when isolated. Test one before moving all of them to the IoT segment.
- Plex needs a specific allow rule from IoT to Services on its ports, not a blanket allow.

## Build order

1. Lab segments only (VLANs 60, 70, 80) as internal Proxmox bridges with no physical port
2. OPNsense as the lab router, firewall rules between lab segments
3. Buy a managed switch and a VLAN aware access point
4. Apply the same design to the house segments, keeping the Spectrum router in place until I am confident
5. Only then consider moving the house edge to OPNsense

## Hardware still needed for the house segments

- A managed switch that supports 802.1Q VLAN tagging
- An access point that can map multiple SSIDs to different VLANs
- A router that can route between VLANs and serve DHCP per VLAN
