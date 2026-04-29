# Maintenance Handbook

This handbook is intended for field technicians and support staff responsible for routine inspection, hardware maintenance, software updates, backups, and change recording for the Network Configuration Manual environment.

## 1. Hardware Maintenance

### 1.1 Routine Check-Up Schedule

| Component | Interval | Maintenance Actions |
| --- | --- | --- |
| FortiGate FG-81E | Monthly | Inspect power, link status, interface errors, firmware status, and configuration backup age |
| Ruijie RG-S2910 | Monthly | Check fan status, temperature, port errors, LAG state, VLAN table, and uplink throughput |
| HPE DL360 Gen 8 | Monthly | Check iLO health, power supply state, RAID status, disk health, memory status, and hypervisor management access |
| Cabling and Patch Panel | Quarterly | Verify labels, cable seating, bend radius, trunk links, and unused port documentation |
| UPS / Power | Quarterly | Confirm battery status, load percentage, runtime estimate, and alarm history |

### 1.2 FortiGate Physical Inspection

- [ ] Confirm power LED is normal.
- [ ] Confirm WAN and LAN link LEDs are active.
- [ ] Check for overheating or blocked ventilation.
- [ ] Verify management access.
- [ ] Confirm recent configuration backup exists.
- [ ] Record firmware version and uptime.

### 1.3 Ruijie Core Switch Physical Inspection

- [ ] Confirm power and system LEDs are normal.
- [ ] Inspect uplink ports and LAG member ports.
- [ ] Check interface counters for errors, drops, and abnormal throughput.
- [ ] Confirm VLAN trunk ports are documented.
- [ ] Verify fan and temperature status if supported.
- [ ] Confirm configuration has been saved to startup configuration.

### 1.4 HPE Server Physical Inspection

- [ ] Confirm server health LED is normal.
- [ ] Check iLO for hardware alerts.
- [ ] Verify power supply redundancy.
- [ ] Check disk and RAID health.
- [ ] Confirm hypervisor management access.
- [ ] Review recent host events for latency, storage, or network warnings.

## 2. Software Updates

### 2.1 General Update Policy

Perform firmware and software updates only during an approved maintenance window.

- [ ] Confirm business approval.
- [ ] Confirm backup completion.
- [ ] Confirm rollback path.
- [ ] Confirm console or out-of-band access.
- [ ] Notify impacted users or stakeholders.
- [ ] Record planned start and end time.

### 2.2 FortiGate Firmware Upgrade Procedure

1. Review the target FortiOS release notes.
2. Confirm the supported upgrade path from the current version.
3. Export the current configuration.
4. Confirm administrator access and console access.
5. Upload the firmware image through the approved management interface.
6. Start the upgrade during the maintenance window.
7. After reboot, verify:
   - [ ] WAN connectivity
   - [ ] Default route
   - [ ] NAT policy
   - [ ] SSL VPN listener
   - [ ] Firewall policy behavior
   - [ ] Logs and system status

### 2.3 Ruijie Switch Firmware Upgrade Procedure

1. Confirm switch model and supported firmware image.
2. Back up the startup configuration.
3. Verify available storage for the firmware image.
4. Upload the firmware image using the approved method.
5. Set the boot image if required.
6. Reload during the maintenance window.
7. After reboot, verify:
   - [ ] VLAN table
   - [ ] SVI gateway addresses
   - [ ] LAG and trunk status
   - [ ] Interface status
   - [ ] Routing table
   - [ ] Management access

### 2.4 Hypervisor Host Patching Procedure

Use the vendor-approved update method for ESXi or Proxmox.

1. Confirm the hypervisor version and patch target.
2. Verify backups for critical virtual machines.
3. Migrate or shut down workloads as required.
4. Confirm iLO access before host reboot.
5. Apply the update.
6. Reboot the host if required.
7. Verify:
   - [ ] Hypervisor management access
   - [ ] VM network connectivity
   - [ ] Port group VLAN tagging
   - [ ] Storage visibility
   - [ ] Host health and logs

## 3. Backup and Disaster Recovery

### 3.1 Backup Frequency

| Asset | Backup Type | Minimum Frequency |
| --- | --- | --- |
| FortiGate | Full configuration export | Before every change and monthly |
| Ruijie Core Switch | Startup configuration backup | Before every change and monthly |
| HPE iLO | Management settings record | After management changes |
| Hypervisor | Host configuration and VM inventory | Before patching and quarterly |
| Documentation Portal | Git commit and repository push | Every documentation change |

### 3.2 Configuration Backup Procedure

1. Create a dated backup folder.

```text
backups/YYYY-MM-DD/
```

2. Export device configurations.

- [ ] FortiGate running configuration
- [ ] Ruijie startup configuration
- [ ] Hypervisor networking summary
- [ ] iLO network settings
- [ ] Current `index.html` and `docs/` state from Git

3. Label each backup file using this standard:

```text
YYYY-MM-DD_device_scope_version.ext
```

Example:

```text
2026-04-29_fortigate_fg81e_full-config_pre-change.conf
2026-04-29_ruijie_rg-s2910_startup-config_pre-change.txt
2026-04-29_hpe-dl360_esxi-networking_pre-patch.txt
```

### 3.3 Disaster Recovery Procedure

Use this procedure when a failed change or outage requires restoration.

1. Stabilize the environment.
   - [ ] Stop non-essential changes.
   - [ ] Confirm management access.
   - [ ] Identify the last known good configuration.

2. Restore network edge service.
   - [ ] Confirm ISP service and NTU Router status.
   - [ ] Restore FortiGate configuration if policy or routing is corrupt.
   - [ ] Verify WAN route and NAT.

3. Restore core switching.
   - [ ] Restore Ruijie startup configuration if VLAN, trunk, or LAG state is invalid.
   - [ ] Verify gateway reachability.
   - [ ] Verify VLAN segmentation.

4. Restore virtualization access.
   - [ ] Confirm hypervisor management access.
   - [ ] Confirm port group VLAN IDs.
   - [ ] Confirm virtual server reachability.

5. Close recovery.
   - [ ] Record incident timeline.
   - [ ] Record failed configuration or failed component.
   - [ ] Record restored backup version.
   - [ ] Update documentation if the network state changed.

## 4. Field Technician SOP

### 4.1 Before Arriving On Site

- [ ] Confirm ticket number and requested scope.
- [ ] Review latest topology diagram.
- [ ] Review latest maintenance log.
- [ ] Confirm required access credentials are available through approved channels.
- [ ] Bring console cable, laptop, Ethernet patch cables, labels, and power tester.

### 4.2 On-Site Work Rules

- [ ] Do not remove unlabeled cables without approval.
- [ ] Photograph cable state before moving uplinks.
- [ ] Label new connections before closing the work order.
- [ ] Confirm redundancy before removing any LAG member.
- [ ] Avoid touching hypervisor management networks unless the change request includes them.
- [ ] Record every physical port change.

### 4.3 Post-Work Verification

- [ ] Ping default gateways for affected VLANs.
- [ ] Confirm internet access from an approved test client.
- [ ] Confirm VPN access if firewall policy was changed.
- [ ] Confirm virtual server reachability if server trunking changed.
- [ ] Confirm no new critical alerts in firewall, switch, iLO, or hypervisor management logs.

## 5. Change Log Template

| Date | Ticket | Change Type | Device / Port | VLAN / IP Impact | Technician | Verification Result | Rollback Plan |
| --- | --- | --- | --- | --- | --- | --- | --- |
| YYYY-MM-DD |  | Physical / Logical / Firmware |  |  |  |  |  |
| YYYY-MM-DD |  | Physical / Logical / Firmware |  |  |  |  |  |
| YYYY-MM-DD |  | Physical / Logical / Firmware |  |  |  |  |  |

## 6. Required Terminology

| Term | Operational Meaning |
| --- | --- |
| Provisioning | Preparing and applying configuration for a device, VLAN, port, or service |
| Latency | Time delay between request and response across the network path |
| Throughput | Actual data transfer rate across a link or service |
| Failover | Automatic or manual transition to a backup path or device after failure |
| Redundancy | Additional path, component, or capacity used to reduce outage risk |
| Encapsulation | VLAN tagging or protocol wrapping used to carry traffic across a trunk |
| Segmentation | Separation of traffic by VLAN, policy, subnet, or security zone |
| Hypervisor Management | Administrative access plane for ESXi or Proxmox host operations |
