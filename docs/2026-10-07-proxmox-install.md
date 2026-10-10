# 2026-10-07 Proxmox install on the Triton 300

## Goal

Turn the Acer Predator Triton 300 into the lab's hypervisor by installing Proxmox VE over Windows, and get it reachable from my main PC so it can run all the lab VMs.

## What I did

1. Cleared off a 256GB thumb drive I had laying around.  
2. On my main PC, downloaded the Proxmox ISO and put it in my Downloads folder.  
3. Downloaded Rufus 4.15, which is what writes the ISO to the thumb drive, and pointed it at the Proxmox ISO so it knew what to write.  
4. Went into the BIOS on the Triton 300 (PT315-52-7337).  
5. Noticed the Secure Boot options were greyed out, so I set a Supervisor Password, which unlocked them.  
6. Turned on the F12 Boot Menu, which were off for some reason.  
7. Disabled Secure Boot and and sure that Intel VTX and VTD were on.  
8. Saw on the Information tab that SATA Mode was set to Optane with RAID. It was read only, with no toggle anywhere in the menus.  
9. Rebooted, pressed F12 repeatedly, and then chose the "Linpus lite" boot entry, which was the file on the thumb drive.  
10. The Proxmox installer launched but it failed with "No Hard Disk found."  
11. I Rebooted back into the BIOS and used the hidden SATA Mode option to change it from Optane with RAID to AHCI.  
12. Ran the installer again. This time it saw the drive.  
13. Set the timezone, root password, and email, then set the hostname.  
14. Installed, rebooted, and pulled the USB.  
15. From my main PC (hogpc), reached the Proxmox web interface on port 8006\.  
16. Set HandleLidSwitch and HandleLidSwitchExternalPower to ignore in `/etc/systemd/logind.conf` and restarted systemd-logind.  
17. Put the laptop in a vertical dock with the lid closed.  
18. The next day, installed lm-sensors and checked temperatures, checked uptime, removed the enterprise and ceph repos, added pve-no-subscription, ran apt update and dist-upgrade on 111 packages, then rebooted.

## Configuration

**Host:** Acer Predator Triton 300 PT315-52-7337, BIOS V1.09

| Setting | Value |
| :---- | :---- |
| Proxmox version | VE 9.2-1 |
| Kernel after updates | 7.0.14-23-pve |
| Target disk | /dev/nvme0n1, 953.87GiB, WDC PC SN730 |
| SATA Mode | Changed from Optane with RAID to AHCI |
| Secure Boot | Disabled |
| Intel VTX / VTD | Enabled |
| Network interface | nic0 |
| IP address | 192.168.1.185/24 |
| Gateway / DNS | 192.168.1.1 |
| Hostname | pve01 |
| Web interface | Port 8006 |

Lid behavior, in `/etc/systemd/logind.conf`:

HandleLidSwitch=ignore

HandleLidSwitchExternalPower=ignore

Repositories: removed the enterprise and ceph sources, added pve-no-subscription.

Temperatures after a few days in the dock with the lid closed: CPU package 60°C, cores 37 to 60°C, NVMe 24.9°C, against a critical threshold of 100°C. Uptime at that check was 2 days 23 hours.

## Problems hit

1. **Saved the ISO to the USB instead of the PC.** Rufus needs to read the ISO from one drive and write to another, and it wipes the target. Moved the file back to Downloads.  
2. **Secure Boot options greyed out.** Fixed by setting a Supervisor Password, which unlocks them on this Acer BIOS.  
3. **F12 Boot Menu was disabled.** Turned it on so I could pick the USB at startup.  
4. **"No Hard Disk found" in the installer.** Fixed with Ctrl \+ S in the BIOS to reveal the hidden SATA Mode option, then switching it from Optane with RAID to AHCI.  
5. **apt update failing with 401 Unauthorized.** Fixed by swapping to the pve-no-subscription repo and deleting the ceph one.

## What I learned

The installer couldn’t see the drive because of the SATA mode. It was set to Optane with RAID, which put the NVMe behind Intel's RST controller instead of showing it directly. Windows could read it because Acer preinstalls Intel's driver, but the Proxmox installer had no driver for that, so to Linux the drive basically didn’t exist. Switching to AHCI takes that layer out and shows the drive directly, which any OS can read. That is also why Windows wouldn’t boot afterward, since I pulled the controller out from it.

The other thing is that my laptop suspended when I close the lid, and a suspended server is a dead server so I changed that behavior in logind.conf because this machine is not a laptop anymore. Putting it in a vertical dock with the lid closed saved space in my office, and I checked the temperatures after to make sure the airflow was still fine.

## Next

- Phase 4: build the lab networks inside Proxmox as internal bridges with no physical port  
- Consider a DHCP reservation on the router so 192.168.1.185 does not get reassigned