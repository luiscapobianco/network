# Network Documentation

## Overview
Home network with 4 VLANs, 5 WiFi networks, and approximately 19 devices.

## Internet Connection
- **ISP:** Fiber connection
- **Speed:** 600Mb duplex

## Network Equipment

### Router
- **Model:** EdgeRouter X SFP
- **OS:** OpenWRT
- **Management IP:** 10.0.0.1
- **Role:** Main router, DHCP server for all VLANs

### Access Points
- **cocina**
  - Model: Unifi AP Pro
  - IP: 10.0.0.150
  - Connected: EdgeRouter port 1
  - Channels: 2.4GHz ch 11 (20 MHz), 5GHz ch 40 (80 MHz)

- **pasillo**
  - Model: Unifi AP Pro
  - IP: 10.0.0.205
  - Connected: Via switch
  - Channels: 2.4GHz ch 6 (20 MHz), 5GHz ch 48 (80 MHz)

### Switch
- **Type:** Unmanaged (dumb) switch
- **Connected:** EdgeRouter port 4
- **Devices on switch:**
  - pasillo AP
  - Raspberry Pi
  - Mac mini

### Raspberry Pi
- **OS:** Ubuntu Server
- **IP Addresses:** 10.0.0.5, 10.0.10.5 (reserved via DHCP)
- **Services:** Unifi Network Software v9.5.21 (manages APs)

## Physical Topology

```
ISP Fiber (600Mb)
    |
    | (port 0)
    |
EdgeRouter X SFP (10.0.0.1)
    |
    +-- port 1 → cocina AP (10.0.0.150)
    |
    +-- port 2 → [unused]
    |
    +-- port 3 → [unused]
    |
    +-- port 4 → Unmanaged Switch
                    |
                    +-- pasillo AP (10.0.0.205)
                    +-- Raspberry Pi (10.0.0.5/10.0.10.5)
                    +-- Mac mini (10.0.0.224/10.0.10.193)
```

## VLANs

### VLAN 0 - Administrative
- **Network:** 10.0.0.0/24
- **Purpose:** Network infrastructure management
- **Devices:**
  - EdgeRouter X (10.0.0.1)
  - cocina AP (10.0.0.150)
  - pasillo AP (10.0.0.205)
  - Raspberry Pi (10.0.0.5)
  - Mac mini (10.0.0.224)

### VLAN 10 - Office
- **Network:** 10.0.10.0/24
- **Purpose:** Primary workspace for homeowners
- **Devices:**
  - Mac mini (10.0.10.193)
  - 2x MacBook Air
  - 2x iPad
  - 2x iPhone
  - 1x Android device
  - 2x Apple Watch
  - Raspberry Pi (10.0.10.5)

### VLAN 20 - Everyone
- **Network:** 10.0.20.0/24
- **Purpose:** Family members, sons, and friends
- **Devices:**
  - 1x MacBook Air
  - 1x iPad
  - 1x iPhone
  - 4x Gaming consoles
  - 1x Printer
  - 1x Apple TV
  - 3x TVs
  - 1x Roku
  - Various roaming Android/iPhone devices

### VLAN 30 - IoT
- **Network:** 10.0.30.0/24
- **Purpose:** IoT devices and guest network isolation
- **Devices:**
  - 1x Washing machine

## WiFi Networks

| SSID | VLAN | Network | Frequencies | Purpose |
|------|------|---------|-------------|---------|
| admin | 0 | 10.0.0.0/24 | 2.4GHz + 5GHz | Network administration |
| office | 10 | 10.0.10.0/24 | 2.4GHz + 5GHz | Primary workspace |
| cachapa | 20 | 10.0.20.0/24 | 2.4GHz + 5GHz | Family and friends |
| IdiOT | 30 | 10.0.30.0/24 | 2.4GHz only | IoT devices |
| guest | 30 | 10.0.30.0/24 | 5GHz only | Guest access |

## DHCP Configuration
- **Server:** EdgeRouter X SFP
- **Reserved IPs:**
  - Raspberry Pi: 10.0.0.5, 10.0.10.5

## Static/Reserved IP Assignments

### VLAN 0 (10.0.0.0/24)
- 10.0.0.1 - EdgeRouter X SFP
- 10.0.0.5 - Raspberry Pi (reserved)
- 10.0.0.150 - cocina AP
- 10.0.0.205 - pasillo AP
- 10.0.0.224 - Mac mini

### VLAN 10 (10.0.10.0/24)
- 10.0.10.5 - Raspberry Pi (reserved)
- 10.0.10.193 - Mac mini

## Recent Changes
- 2024-11-15: Changed pasillo 2.4GHz channel width from 40MHz to 20MHz (reduces interference)

## Known Issues
- **Zoom Connectivity Problems:** Wife experiencing poor connectivity during concurrent Zoom meetings (10am-5pm weekdays)
  - Both using MacBook Airs on 5GHz "office" WiFi
  - Issue occurs when both have simultaneous video calls
  - See `recommendations.md` for detailed analysis and fixes

## Notes
- Mac mini has dual network interfaces (one on VLAN 0 for admin, one on VLAN 10 for work)
- Raspberry Pi also has dual interfaces (VLAN 0 and VLAN 10)
- Guest and IoT devices are isolated on VLAN 30
- Unifi Network Software runs on Raspberry Pi to manage both APs
- EdgeRouter ports 2, 3, and SFP port are currently unused
- Both MacBook Airs confirmed using 5GHz band on "office" SSID
