# Homelab Plan and Roadmap

Living plan for my homelab. When a phase is finished, change its status to Done and upload this file to the project again.

## Goal

Build a segmented network, run a Windows AD domain, attack it from Kali, and detect the attacks with Wazuh or Security Onion. Document everything on GitHub as a portfolio for an entry level Security Analyst role with the State of Texas.

## Hardware

| Device | Details | Role in the lab | Status |
|---|---|---|---|
| Acer Predator Triton 300 | PT315-52-7337, i7-10750H, RTX 2070 Max-Q, 32GB DDR4 (2 of 2 slots, 2933 MT/s), WD SN730 1TB NVMe, free M.2 slot JSSD2, Killer 1650i WiFi | Proxmox node, runs all VMs | Owned, opened and cleaned 2026-10-04 |
| Raspberry Pi 3 B+ | 1GB RAM, currently running Kali | Linux practice now, honeypot later | Owned |
| Flipper Zero with WiFi dev board | Marauder firmware for WiFi testing | Wireless attack practice (own gear only) | Owned |
| Spare iPhones: SE 2nd gen, 13, XR | Unused, no plan attached | Test client devices on the lab wireless network for Phase 10, and traffic to capture and analyze | Owned |
| Microsoft Surface Go | Low powered | Portable terminal for working at the laptop, or a second small Linux box for practice | Owned |
| Main desktop | Ryzen 7 5700X, 32GB RAM | Workstation for Proxmox web UI, GitHub, docs | Owned |
| Spectrum modem and house router | Home internet | Stays separate from the lab | In use |

## Standing rules

- The house network stays on its own router. The lab is a separate segment behind it. Double NAT is fine.
- Do not make OPNsense the house router until I've run it for months and I'm confident.
- No passwords, keys, tokens, WiFi passwords, raw config exports, or my public IP in the repo. Ever.
- No scanning or attack tools against the house network. Offensive tools only inside the segmented lab.
- Flipper and wireless attacks only against networks and devices I own.
- Snapshot in Proxmox or export the OPNsense config before any risky change.
- After each phase: write the doc entry, commit, push.

## Doc template for each build entry

Goal, What I did, Configuration, Problems hit, What I learned, Next.

## Roadmap overview

| Phase | Task | Status |
|---|---|---|
| 0 | GitHub setup and README purpose section | Not started |
| Ongoing | Linux practice on the Pi | Not started |
| 1 | Laptop teardown and upgrades | Done 2026-10-04 |
| 2 | Proxmox install on the Triton 300 | Not started |
| 3 | Physical setup and cabling | Not started |
| 4 | Lab network segmentation inside Proxmox | Not started |
| 5 | OPNsense as lab router | Not started |
| 6 | Windows AD domain controller and workstation | Not started |
| 7 | Kali attacker VM | Not started |
| 8 | Wazuh or Security Onion, traffic mirroring, first detections | Not started |
| 9 | Raspberry Pi honeypot with OpenCanary | Not started |
| 10 | Wireless attack and detection: Flipper vs Kismet | Not started |

## Phase details

### Phase 0: GitHub setup

- Do GitHub Skills "Introduction to GitHub," then "Communicate using Markdown"
- Install Git for Windows (for terminal use) and GitHub Desktop (to see changes visually)
- Install Obsidian and point it at the repo folder for writing docs
- Create a repo called `homelab`
- Write `.gitignore` before the first commit (`.obsidian/`, `*.env`, `secrets/`)
- Turn on secret scanning and push protection in repo settings
- Write the README purpose section in my own words
- Done when: first commit pushed with README and .gitignore

### Ongoing: Linux practice on the Pi

- SSH into the Pi from the desktop
- Practice navigating folders, editing files, installing packages
- No scanning the house network
- Done when: comfortable moving around a Linux command line without looking everything up

### Phase 1: Laptop teardown (Done 2026-10-04)

- Opened the bottom panel, disconnected the battery, cleaned both fans
- Found 32GB already installed, both slots used, so no RAM purchase
- Found a free M.2 slot (JSSD2) if I ever want a second NVMe
- Decided not to repaste
- Still open: back up the laptop before Phase 2, check the battery for swelling, inspect the heatsink fins
- Done when: laptop opened, cleaned, reassembled, and boots

### Phase 2: Proxmox install

- Back up anything I want off the laptop first, the Proxmox install erases Windows
- Check virtualization is enabled in BIOS
- Switch storage mode from RST/RAID to AHCI in BIOS, otherwise the installer may not see the NVMe drive
- Flash the Proxmox installer to USB and install over Windows (no factory reset needed)
- Set lid close to ignore in `/etc/systemd/logind.conf` so it doesn't suspend
- Cap battery charge if possible, check for swelling every few months
- Reach the Proxmox web interface from the desktop
- Learn snapshots before building anything
- Known limits: one NIC, no GPU passthrough, 32GB RAM total so plan on Wazuh rather than Security Onion
- Done when: Proxmox running, reachable from desktop, first snapshot taken

### Phase 3: Physical setup

- Laptop somewhere with airflow, not sealed in or stacked under things
- Wired Ethernet from the laptop to the house router for the lab's internet uplink
- Label cables
- No rack. Laptop and Pi sit on a shelf or desk for now
- Done when: laptop placed, wired, and stable running 24/7

### Phase 4: Lab network segmentation

- Plan lab networks (management, servers, AD, attacker, honeypot) and write the plan down first
- Build them as internal Proxmox bridges or VLANs with no physical port, so lab traffic never touches the house network
- Keep the Proxmox management interface on the house side so I can always reach it from the desktop
- Snapshot before every change
- Done when: lab networks exist, are documented, and the Proxmox web UI is still reachable
- Later option: a managed switch if I get one, for physical devices like the Pi

### Phase 5: OPNsense

- OPNsense VM on Proxmox as the lab router only
- WAN side on the bridge connected to the house network, LAN side on the internal lab networks
- Firewall rules between lab networks
- Lab DNS through OPNsense's built in Unbound
- Export config before every change
- Done when: lab networks route through OPNsense and rules block what they should

### Phase 6: Windows AD

- Windows Server evaluation ISO for the domain controller
- Set up AD, DNS, users, groups, a few GPOs
- Windows 10 or 11 evaluation VM as a workstation joined to the domain
- Done when: workstation logs in with a domain account

### Phase 7: Kali

- Kali VM on the attacker network
- Confirm what it can and can't reach based on firewall rules
- Done when: Kali runs and network access matches the plan

### Phase 8: Detection

- Wazuh, not Security Onion, because the node only has 32GB total
- Agents on the domain controller and workstation
- Mirror lab traffic to the network sensor from the Proxmox bridge (or switch port mirroring if I add a managed switch)
- First detections: port scan, password brute force, other attacks from Kali
- Done when: an attack from Kali shows up as an alert and is documented

### Phase 9: Raspberry Pi honeypot

- Use a second SD card so the Kali card stays intact
- Raspberry Pi OS Lite 64 bit, good quality SD card or USB boot
- Solid 5V 2.5A micro USB power supply
- Install OpenCanary, nothing else legitimate on this Pi
- Install the Wazuh agent so honeypot hits become alerts
- Connect it to the lab side so Kali can reach it but the house network can't (needs a second NIC on the laptop or a managed switch, decide when I get here)
- Test by scanning from Kali
- Done when: a Kali scan triggers a honeypot alert in Wazuh

### Phase 10: Wireless attack and detection

- Kismet on the Pi with a USB WiFi adapter that supports monitor mode
- Test access point that I own
- Connect a spare iPhone to that test access point as the client device, so there's real traffic to watch and a client to knock off
- Flipper with Marauder attacks only that test access point
- Done when: Kismet detects the Flipper's attack and it's documented
