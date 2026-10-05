# \# Homelab

# 

# \## Purpose

# 

# My purpose for building this homelab was to get an understanding of networking and network security, and to build toward an entry level security analyst role. I also want to be able to maintain and secure my own home network.

# 

# \## Goals

# 

# \- Build a segmented network with partitions that have their own VLAN and subnet

# \- Ensure only the secure areas of the network can have two way connections, and that IoT devices stay partitioned and cannot reach the rest of the network

# \- Test network security with the Flipper Zero and detect it using Wazuh

# \- Self host a streaming service and an ad blocker

# 

# \## Hardware

# 

# | Device | Specs | Role |

# |---|---|---|

# | Acer Predator Triton 300 (PT315-52-7337) | i7-10750H, 32GB DDR4 2933, WD SN730 1TB NVMe, RTX 2070 Max-Q | Proxmox node, runs all VMs |

# | Raspberry Pi 3 B+ | 1GB RAM | Linux practice, honeypot later |

# | Flipper Zero + WiFi dev board | Marauder firmware | Wireless attack testing, own gear only |

# | Microsoft Surface Go | Low powered | Portable terminal |

# | Spare iPhones (SE 2nd gen, 13, XR) | Unused | Test clients for wireless detection |

# | Desktop (hogpc) | Ryzen 7 5700X, 32GB RAM | Workstation for docs and repo |

# 

# No managed switch or rack yet. Network segmentation is handled with virtual networks inside Proxmox.

# 

# \## Build log

# 

# \- \[Laptop teardown and inspection](docs/2026-10-04-laptop-teardown.md)

# \- \[Roadmap](docs/homelab-roadmap.md)

