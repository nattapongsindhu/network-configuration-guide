# Operations Guide

This guide defines the operational model for the Network Configuration Manual. It is intended for administrators, network engineers, and systems engineers who maintain the logical network state, configuration standards, and incident response workflow.

## 1. System Overview

The network follows a layered path from public access to internal compute resources:

```text
ISP / Internet
  -> NTU Router
  -> FortiGate Firewall
  -> Ruijie Core Switch
  -> HPE Virtualization Host
  -> Virtual Servers and Client VLANs
```

### 1.1 Network Role Summary

| Layer | Component | Operational Purpose |
| --- | --- | --- |
| Internet Edge | ISP / NT FTTx | Provides public connectivity, PPPoE service, and dynamic public IP routing |
| Customer Premises Edge | NTU Router | Terminates ISP access and forwards WAN traffic toward the firewall |
| Security Boundary | FortiGate FG-81E | Provides NAT, policy enforcement, SSL VPN, segmentation, and failover-ready security control |
| Core Switching | Ruijie RG-S2910 | Provides VLAN gateway services, trunking, LAG aggregation, DHCP support, and internal routing |
| Compute Layer | HPE DL360 Gen 8 | Hosts the hypervisor platform for virtual servers and infrastructure workloads |
| Out-of-Band Management | iLO | Provides hypervisor-independent remote management for server recovery and provisioning |

### 1.2 Traffic Logic

- Inbound traffic enters through the ISP and reaches the NTU Router.
- The NTU Router forwards permitted WAN traffic to the FortiGate WAN interface.
- The FortiGate applies firewall policy, NAT, and VPN access control.
- Internal routed traffic is handed to the Ruijie Core Switch.
- The Core Switch separates workloads by VLAN segmentation and forwards traffic to servers, users, and management networks.
- The virtualization host receives tagged VLANs through a trunk or bonded uplink for virtual machine provisioning.

## 2. Infrastructure-as-Code Documentation Concept

This web portal represents the physical and logical network as documentation that can be versioned, reviewed, and deployed.

### 2.1 IaC Principles Used

- The Mermaid topology diagram represents the current intended network state.
- Hardware tables represent inventory and management access references.
- VLAN/IP tables represent segmentation, routing, and gateway assignments.
- Configuration snippets represent repeatable provisioning commands.
- Maintenance logs represent the audit history of physical and logical changes.

### 2.2 Source of Truth Rules

- [ ] Update the web portal before or during approved network changes.
- [ ] Keep VLAN names, subnet ranges, and gateway addresses consistent with the live configuration.
- [ ] Record every firewall policy, trunk, and LAG change in the maintenance log.
- [ ] Treat copied CLI snippets as templates; verify device firmware and syntax before execution.
- [ ] Use Git commits as change records for documentation updates.

## 3. Configuration Standards

### 3.1 Naming Conventions

Use short, uppercase names for VLANs and operational labels.

| Object Type | Standard | Example |
| --- | --- | --- |
| VLAN Name | Uppercase with underscores | `HR_FIN`, `WIFI_GUEST`, `IOT_CCTV` |
| Interface Description | Directional and device-based | `LAG_TO_FORTIGATE`, `LAG_TO_HPE_DL360_ESXI` |
| Firewall Policy Name | Source-to-destination format | `SSLVPN_to_LAN` |
| Server Hostname | Site-role-number format | `bug-srv01-ilo` |
| VPN Group | Purpose-based uppercase name | `VPN_USERS` |

### 3.2 VLAN and IP Standards

| VLAN | Name | Subnet | Gateway | Purpose |
| --- | --- | --- | --- | --- |
| 5 | MGMT | `192.168.5.0/24` | `192.168.5.254` | Network and device management |
| 10 | OFFICE | `192.168.10.0/23` | `192.168.10.254` | Office client access |
| 20 | HR_FIN | `192.168.20.0/24` | `192.168.20.254` | HR and finance systems |
| 30 | SERVERS | `192.168.30.0/24` | `192.168.30.254` | Virtual server workloads |
| 40 | WIFI_GUEST | `192.168.40.0/23` | `192.168.40.254` | Guest wireless access |
| 90 | IOT_CCTV | `192.168.90.0/23` | `192.168.90.254` | Cameras and IoT devices |
| 100 | DMZ | `192.168.100.0/24` | `192.168.100.254` | Public-facing or isolated services |

### 3.3 Port Management Standards

- Use trunk ports for links that carry multiple VLANs.
- Use access ports for single-purpose endpoint connections.
- Use LACP-based LAG for uplinks requiring redundancy and additional throughput.
- Keep native VLAN usage documented and intentional.
- Avoid mixing management traffic with guest or DMZ traffic.
- Document encapsulation expectations for every trunk connection.

### 3.4 Configuration Review Checklist

- [ ] VLAN ID matches the approved VLAN table.
- [ ] Gateway IP is unique and reachable.
- [ ] Trunk allowed VLAN list is explicit.
- [ ] LAG members are consistent on both ends.
- [ ] Firewall policy source, destination, service, and NAT behavior are documented.
- [ ] SSL VPN address pool and access rules are reviewed.
- [ ] Hypervisor port group VLAN IDs match the Core Switch configuration.

## 4. Troubleshooting Flow

Use this SOP during network failure diagnosis. Work from physical layer to application layer.

### 4.1 Initial Triage

1. Confirm the reported impact.
   - [ ] Identify affected users, VLANs, servers, and services.
   - [ ] Determine whether the issue is isolated or site-wide.
   - [ ] Record the incident start time.

2. Check power and physical connectivity.
   - [ ] Verify FortiGate, Ruijie, and HPE server power status.
   - [ ] Inspect link LEDs and cable seating.
   - [ ] Confirm uplink ports are active.

3. Check WAN and firewall status.
   - [ ] Confirm ISP service status.
   - [ ] Test NTU Router connectivity.
   - [ ] Confirm FortiGate WAN IP and default route.

### 4.2 Diagnostic Command Flow

#### FortiGate

```text
get system status
get router info routing-table all
diagnose vpn ssl list
execute ping 192.168.5.254
execute traceroute 8.8.8.8
```

#### Ruijie Core Switch

```text
show vlan brief
show ip interface brief
show lacp neighbor
show ip route
show interfaces status
ping 192.168.5.1
```

#### Hypervisor Host

```text
Check management IP reachability.
Check vmkernel or management interface status.
Check port group VLAN tagging.
Check virtual switch or bond uplink status.
Review hypervisor management events.
```

### 4.3 Failure Domain Decision Tree

| Symptom | Likely Domain | Action |
| --- | --- | --- |
| No internet for all users | ISP, router, firewall WAN | Check PPPoE, WAN route, NAT, and ISP outage status |
| One VLAN cannot reach gateway | Core SVI or access/trunk path | Check VLAN presence, SVI status, and allowed VLAN list |
| VPN users cannot connect | Firewall SSL VPN | Check VPN listener port, certificate, group mapping, and policy |
| Virtual servers unreachable | Hypervisor, trunk, server VLAN | Check LAG, VLAN tagging, port group, and host uplinks |
| High latency or packet loss | WAN, overloaded link, LAG member failure | Check interface counters, throughput, drops, and provider status |
| Failover does not occur | Redundancy design or LACP issue | Check LAG state, member ports, and peer configuration |

### 4.4 Escalation Rules

- Escalate to ISP support when WAN loss is confirmed beyond the NTU Router.
- Escalate to security administration when firewall policy or VPN authentication is the failure point.
- Escalate to systems administration when hypervisor management or virtual switching is impacted.
- Escalate to field service when hardware fault indicators, power issues, or cabling damage are present.

### 4.5 Closure Requirements

- [ ] Root cause is recorded.
- [ ] Affected device, VLAN, or service is identified.
- [ ] Verification commands are documented.
- [ ] Any configuration change is added to the maintenance log.
- [ ] Documentation is updated if the intended network state changed.
