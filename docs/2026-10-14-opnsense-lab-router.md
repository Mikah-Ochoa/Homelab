# 2026-10-14 OPNsense as the lab router

## Goal

Build OPNsense as a VM with one interface on the house network and one on each lab bridge, making it the only route between the lab segments, then write firewall rules that enforce the access table from the network plan.

## What I did

1. Downloaded the OPNsense 26.7 dvd amd64 ISO on hogpc and extracted the .bz2 with 7-Zip, which I also had to install.  
2. Uploaded the ISO to Proxmox under local (pve01), ISO Images.  
3. Created the VM as VM 100, named opnsense, with Guest OS type set to Other since OPNsense is built on FreeBSD rather than Linux.  
4. Changed the disk bus from IDE to SCSI.  
5. Created it with one network card on vmbr0, then added three more for vmbr60, vmbr70, and vmbr80.  
6. Wrote down the interface mapping before starting the installer, since the installer does not tell you which bridge each interface connects to.  
7. Installed with UFS rather than ZFS.  
8. Set the root password and completed the install.  
9. Assigned interfaces from the console, then set addresses on each one.  
10. Created the firewall rules and disabled the default allow-everything LAN rules.  
11. Took snapshots at each working state and downloaded an encrypted config backup.

## Configuration

**Interface mapping**

| Proxmox device | Bridge | OPNsense interface | Address |
| :---- | :---- | :---- | :---- |
| net0 | vmbr0 | WAN | DHCP from the house router |
| net1 | vmbr60 | LAN | 10.10.60.1/24 |
| net2 | vmbr70 | LAB\_ATTACKER  | 10.10.70.1/24 |
| net3 | vmbr80 | LAB\_HONEYPOT  | 10.10.80.1/24 |

Only WAN has an upstream gateway. The three lab networks are local and route through it.

LAN runs a DHCP server handing out .100 to .200, matching the convention in the network plan.

**VM settings**

| Setting | Value |
| :---- | :---- |
| Guest OS type | Other (FreeBSD) |
| BIOS | SeaBIOS |
| Disk | 32GB, SCSI bus |
| CPU | 2 cores, type host |
| Memory | 2048 MB |
| Filesystem | UFS |

**Firewall alias**

`PrivateNetworks-` containing 10.0.0.0/8, 172.16.0.0/12, and 192.168.0.0/16.

**Firewall rules**

| Interface | Source | Destination | Log | Purpose |
| :---- | :---- | :---- | :---- | :---- |
| LAB\_ATTACKER | LAB\_ATTACKER net | LAN net | Yes | Attacker reaches the lab servers |
| LAB\_ATTACKER | LAB\_ATTACKER net | LAB\_HONEYPOT net | Yes | Attacker can trigger honeypot alerts |
| LAB\_ATTACKER | LAB\_ATTACKER net | \! PrivateNetworks | No | Internet only, no internal access |
| LAN | LAN net | \! PrivateNetworks | No | Internet only, no internal access |
| LAB\_HONEYPOT | LAB\_HONEYPOT net | \! PrivateNetworks | No | Internet only, no internal access |
| WAN | hogpc only, /32 | WAN address, HTTPS | Yes | Temporary management access |

The two default "allow LAN to any" rules were disabled after the replacement rule was in place.

**Snapshots**

opnsense-base-working, before-firewall-rules, and opnsense-rules-complete, all taken with RAM.

An encrypted config backup is stored outside the repo.

## Problems hit

**1\. The VM wouldn’t boot.** It dropped to the UEFI boot device menu instead of starting the installer. I reordered the boot priority and it did not help. The cause was Secure Boot: the Pre-Enroll keys option had loaded Microsoft's certificates into the virtual firmware, and I found out that it was because FreeBSD's bootloader is not signed by Microsoft. There was no Detach option available for the EFI disk, so I switched the BIOS from OVMF (UEFI) to SeaBIOS instead and the installer booted immediately.

**2\. Repeating `vm_fault: pager read error, pid 1 (init)` after install.** The CD error was because it  was still in the virtual drive. Setting the CD/DVD drive to "Do not use any media" fixed it.

**3\. IP address conflict with the house router.** The first console view showed LAN assigned to vtnet0, the house-facing interface, with the Spectrum router's own address on it. So I had two devices claiming the same address on one network which caused a conflict. Fixed with console option 1 to reassign WAN to vtnet0, LAN to vtnet1, OPT1 to vtnet2, and OPT2 to vtnet3.

**4\. First ping to 1.1.1.1 failed with "No route to host."** DHCP on WAN hadn’t finished yet. The second attempt came back with 0% packet loss.

**5\. Locked out of the web interface, twice.** The OPNsense web UI bound to LAN at 10.10.60.1, which sits on an isolated bridge that hogpc can’t reach. I disabled the packet filter with `pfctl -d` to reach the UI on the WAN address, then created a WAN rule allowing HTTPS from hogpc only. Re-enabling the filter with `pfctl -e` locked me out again. Block private networks and Block bogon networks were both already unchecked, so that wasn’t the problem. The actual cause was reply-to, which forced reply traffic out the WAN gateway and broke the return path when the client was on the same subnet as WAN. I fixed it by enabling "Disable reply-to on WAN rules" under Firewall, Settings, Advanced.

## What I learned

I went with UFS instead of ZFS because ZFS wanted more RAM, and with RAM prices the way they are right now I am better off keeping it small with UFS at least for now.

1\. I learned that the better way to make sure that areas of the network cant talk to anything on my privatenetworks I had to invert the destination and then set the destination to Private networks so it is a more broad umbrella of blocking anything going to privatenetworks. The other option would've had to be creating BLOCKING rules which is less efficient and time consuming and something could get missed versus me just setting an allow but inverting it to not let it speak to privatenetworks. 

2\. I found out that rule order matters alot more than I thought, and QUICK is what made it matter. I knew that normally pf reads through every rule and the last match wins. But I didn't know that with QUICK enabled, it stops at the first match and never looks at anything below it. I had set a new LAN rule that wasn’t taking effect and I couldn’t figure out why, until I saw the two default allow rules sitting above mine with QUICK on. They were matching first and ending the search before my rule was read. I disabled the two default rules by unchecking them, and left my rule as the only active one on that interface. I disabled them instead of deleting them so I can turn them back on if something breaks. 

3\. Why reply-to broke management access, and why that problem only exists when WAN shares a subnet with the machine managing the firewall.\] The problem took me a while to figure out because I kept getting locked out of the network and I had to keep using pfctl \-d to get back in to check what other problems it could be. After trying a bunch of solutions I decided to look at the firewall advanced settings and saw the cause was reply-to, which forced reply traffic out the WAN gateway and broke the return path when the client was on the same subnet as WAN. The fix ended up being easy. I just had to  hit the enable the "Disable reply-to on WAN rules" toggle and I was good to go.

4\. The firewall being stateful is what made my one directional rules possible. I did it so that I could allow attacker to servers and still block servers to attacker, and both still work. A stateless firewall would need a rule for every direction of every conversation and I would lose the asymmetry. Its kind of the reason my phone will be able to reach a Ring camera later and the camera can’t reach my phone. 

## Next

- Replace the temporary WAN management rule once there is a VM on the lab network to manage from  
- Phase 6: Windows Server domain controller and a workstation VM on the lab servers network