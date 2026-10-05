# 2026-10-04 Laptop teardown and inspection

## Goal

Open the Acer Predator Triton 300 that will become the Proxmox node, find out what RAM and storage are actually installed, and clean it before it starts running 24/7.

## What I did

1. Shut the laptop down and removed all of the bottom panel screws.
2. The panel would not come off at first. Got it off eventually (method not recorded).
3. Photographed the internals and identified the RAM, the storage, and an empty M.2 slot.
4. Located the battery connector (PJP201, the white plug below the RAM) and disconnected it before working inside.
5. Cleaned both fans with compressed air.
6. Decided not to repaste the CPU and GPU.
7. Reconnected the battery, reinstalled the panel, and booted back into Windows.
8. Checked Task Manager > Performance > Memory to confirm the RAM configuration.
9. Photographed the model sticker and looked up the board's RAM and M.2 support.

## Configuration

**Model:** Acer Predator Triton 300 PT315-52-7337, manufactured 2020/07/25

| Component | Found |
|---|---|
| RAM | 32GB DDR4 SODIMM, 2 of 2 slots used, running at 2933 MT/s |
| Stick 1 | SK hynix 16GB 1Rx8 PC4-3200AA (HMAA2GS6AJR8N) |
| Stick 2 | Label reads KN16G0, 16GB |
| Storage | Western Digital PC SN730 1TB NVMe, in the slot labeled 1.PCIE |
| Free M.2 slot | Yes, JSSD2, directly below the RAM. Screw and standoff present: unknown |
| WiFi | Killer 1650i |
| Storage mode | Task Manager reports the disk as "SSD (RAID)", which suggests Intel RST is enabled |
| BIOS | Not opened. Nothing checked or changed |

Unknown or not inspected: whether a third M.2 slot exists, heatsink fin condition, whether the battery shows any swelling.

## Problems hit

The bottom panel would not separate after every screw was out. It came off eventually, but I did not record how, so I cannot repeat it next time.

## What I learned

The laptop already has 32GB with both slots full, so there is no cheap upgrade path. Going to 64GB would mean replacing both sticks, and current prices for a 2x32GB kit are around $500. Sources also disagree about whether this board supports 64GB at all, with most listing 32GB as the maximum. Staying at 32GB is the right call, which means Wazuh rather than Security Onion for the detection phase later.

Task Manager reporting the drive as RAID means the storage controller is in Intel RST mode. That has to change to AHCI in the BIOS before the Proxmox install, or the installer may not see the NVMe drive at all.

## Next

- Back up anything worth keeping off the laptop, since the Proxmox install will erase Windows
- Phase 0: set up the GitHub repo, .gitignore, and README
- Phase 2: switch BIOS to AHCI, confirm virtualization is enabled, then install Proxmox
