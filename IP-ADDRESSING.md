# Network IP Addressing Table

Addressing plan for all devices in the lab network (HQ + Branch, connected through an ISP and a GRE tunnel).

## Topology Overview

```
                         Net
                          |
                       [ ISP ]
                  ens4 /     \ ens5
                      /       \
                 [HQ-RTR] ═GRE═ [BR-RTR]
                     |             |
                  [HQ-SW]       [BR-FW]
                  /     \          |
            HQ-SRV     HQ-CLI   BR-SRV
```

## Device IP Addresses

| Device | Interface | VLAN / Role | IP Address | Prefix | Subnet Mask | Gateway |
|--------|-----------|-------------|------------|--------|-------------|---------|
| **ISP** | ens3 | Uplink (Internet) | 10.0.137.201 | /24 | 255.255.255.0 | - |
| | ens4 | Link to HQ-RTR | 172.16.1.1 | /28 | 255.255.255.240 | - |
| | ens5 | Link to BR-RTR | 172.16.2.1 | /28 | 255.255.255.240 | - |
| **HQ-RTR** | ge0 | ISP link | 172.16.1.2 | /28 | 255.255.255.240 | 172.16.1.1 |
| | ge1 | VLAN 100 (Servers) | 10.10.100.1 | /27 | 255.255.255.224 | - |
| | ge1 | VLAN 200 (Clients) | 10.10.200.1 | /28 | 255.255.255.240 | - |
| | ge1 | VLAN 999 (Management) | 10.10.30.1 | /29 | 255.255.255.248 | - |
| | tunnel.1 | GRE tunnel to BR-RTR | 10.10.10.1 | /30 | 255.255.255.252 | - |
| **HQ-SW** | ens3 | Trunk to HQ-RTR | - | - | - | - |
| | ens4 | Access to HQ-SRV | - | - | - | - |
| | ens5 | Access to HQ-CLI | - | - | - | - |
| **HQ-SRV** | ens3 | VLAN 100 | 10.10.100.2 | /27 | 255.255.255.224 | 10.10.100.1 |
| **HQ-CLI** | ens3 | VLAN 200 (DHCP) | 10.10.200.2 | /28 | 255.255.255.240 | 10.10.200.1 |
| **BR-RTR** | ge0 | ISP link | 172.16.2.2 | /28 | 255.255.255.240 | 172.16.2.1 |
| | ge1 | Link to BR-FW | 10.20.10.1 | /30 | 255.255.255.252 | - |
| | ge2 | SSH interface | 169.254.2.10 | - | - | - |
| | tunnel.1 | GRE tunnel to HQ-RTR | 10.10.10.2 | /30 | 255.255.255.252 | - |
| **BR-FW** | oif0 | Outside (to BR-RTR) | 10.20.10.2 | /30 | 255.255.255.252 | 10.20.10.1 |
| | iif0 | Inside (to BR-SRV) | 10.20.20.1 | /28 | 255.255.255.240 | - |
| **BR-SRV** | ens3 | Branch LAN | 10.20.20.2 | /28 | 255.255.255.240 | 10.20.20.1 |

## Subnet Summary

| Subnet | Purpose | Usable Range | Gateway / Router |
|--------|---------|--------------|------------------|
| 10.0.137.0/24 | ISP uplink to the Internet | 10.0.137.1 - 10.0.137.254 | - |
| 172.16.1.0/28 | ISP <-> HQ-RTR | 172.16.1.1 - 172.16.1.14 | ISP (172.16.1.1) |
| 172.16.2.0/28 | ISP <-> BR-RTR | 172.16.2.1 - 172.16.2.14 | ISP (172.16.2.1) |
| 10.10.100.0/27 | HQ VLAN 100 (Servers) | 10.10.100.1 - 10.10.100.30 | HQ-RTR (10.10.100.1) |
| 10.10.200.0/28 | HQ VLAN 200 (Clients) | 10.10.200.1 - 10.10.200.14 | HQ-RTR (10.10.200.1) |
| 10.10.30.0/29 | HQ VLAN 999 (Management) | 10.10.30.1 - 10.10.30.6 | HQ-RTR (10.10.30.1) |
| 10.10.10.0/30 | GRE tunnel HQ-RTR <-> BR-RTR | 10.10.10.1 - 10.10.10.2 | - |
| 10.20.10.0/30 | BR-RTR <-> BR-FW | 10.20.10.1 - 10.20.10.2 | BR-RTR (10.20.10.1) |
| 10.20.20.0/28 | Branch LAN | 10.20.20.1 - 10.20.20.14 | BR-FW (10.20.20.1) |

## Notes

- **HQ-CLI** gets its address via DHCP. Its default gateway is the HQ-RTR VLAN 200 interface (`10.10.200.1`).
- **HQ-SW** is a Layer 2 switch, so it has no IP address in the plan above.
- **BR-RTR ge2** (`169.254.2.10`) is a link-local address used for SSH access. No prefix length was provided.
- **HQ-RTR ge1** is a single physical interface carrying VLANs 100, 200 and 999 (router-on-a-stick).
