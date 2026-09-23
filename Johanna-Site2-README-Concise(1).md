# Site 2 — Johanna Salibongcogon

**Unit:** COIT12202 Network Security Concepts  
**Assessment:** A2 — Secured Multi-Site Network Project  
**Student:** Johanna Salibongcogon  
**Student ID:** 12306262  
**Site:** Site 2  
**Advertised Route:** `10.20.0.0/16`

---

## 1. Site Overview

This folder documents my Site 2 GNS3 network, its addressing, current configuration and testing evidence. I built the LAN, DMZ and Transit networks, configured OPNsense and connected VPNRouter1 to Tailscale. Firewall rules and full cross-site integration are still in progress.

### 1.1 Address Plan

| Segment | Subnet | Gateway | Purpose |
|---|---|---|---|
| LAN | `10.20.1.0/24` | `10.20.1.1` | PC1, CA and Target |
| DMZ | `10.20.2.0/24` | `10.20.2.1` | HTTPS and SSH servers |
| Transit | `10.20.3.0/24` | `10.20.3.1` | OPNsense WAN to VPNRouter1 |
| Advertised | `10.20.0.0/16` | — | Covers my Site 2 networks |

### 1.2 Device Inventory

| Device | Interface | IP Address | Role |
|---|---|---|---|
| OPNsense | em0 / vtnet0 (WAN) | `10.20.3.1/24` | Transit firewall interface |
| OPNsense | em1 / vtnet1 (LAN) | `10.20.1.1/24` | LAN gateway |
| OPNsense | em2 / vtnet2 (OPT1) | `10.20.2.1/24` | DMZ gateway |
| VPNRouter1 | eth0 | `10.20.3.2/24` | Tailscale subnet router |
| VPNRouter1 | eth1 | DHCP (observed `192.168.122.144/24`) | GNS3 NAT/Internet |
| PC1 | eth0 | `10.20.1.10/24` | Client |
| CA | eth0 | `10.20.1.20/24` | Certificate Authority host |
| Target | eth0 | `10.20.1.30/24` | Internal host |
| HTTPS Server | eth0 | `10.20.2.10/24` | DMZ web-server host |
| SSH Server | eth0 | `10.20.2.20/24` | DMZ SSH-server host |

---

## 2. Topology

My current GNS3 topology has three network zones. Switch1 connects PC1, CA and Target to the LAN; Switch2 connects the HTTPS and SSH servers to the DMZ; Switch3 connects OPNsense to VPNRouter1, which reaches the GNS3 NAT node.

![Current Site 2 GNS3 topology](https://github.com/user-attachments/assets/a508e0f3-8bff-4b45-bf27-08e0520ebeb8)

*The screenshot shows my current build. Kerberos and Suricata devices have not yet been added.*

---

## 3. Configuration and Services

### 3.1 LAN and Assessment 1 Hosts

- Configured PC1 (`10.20.1.10`), CA (`10.20.1.20`) and Target (`10.20.1.30`) with gateway `10.20.1.1`.
- All three LAN hosts successfully pinged OPNsense and one another.
- CA and Target are the hosts carried over for the Assessment 1 services; application-level verification will be documented separately.

**Evidence:** [PC1 tests](https://github.com/user-attachments/assets/a40fd7c6-47e3-4cf4-abb9-bfedbb36df69) · [CA tests](https://github.com/user-attachments/assets/708adb54-afc3-4575-ad51-34f3bf510417) · [Target tests](https://github.com/user-attachments/assets/9d9cca4e-7afe-4818-a71d-664e257b6586)

### 3.2 DMZ

- Configured HTTPS Server (`10.20.2.10`) and SSH Server (`10.20.2.20`), both using `10.20.2.1` as their gateway.
- Both servers successfully pinged each other; OPNsense successfully pinged the HTTPS Server.
- DMZ-initiated ping to OPNsense is blocked with the firewall enabled. Permanent DMZ firewall rules and final tests are **in progress**.

**Evidence:** [HTTPS IP](https://github.com/user-attachments/assets/bd2321a4-200e-4b19-b3bd-cdf7c4fb914d) · [SSH IP](https://github.com/user-attachments/assets/a9cc9244-4d52-41a1-8ba0-642631c0acbf) · [Firewall-enabled test](https://github.com/user-attachments/assets/76e4b1e5-0b9f-4381-92c4-013ec9db56ca)

### 3.3 OPNsense Firewall

- Configured LAN `10.20.1.1/24`, DMZ `10.20.2.1/24` and WAN/Transit `10.20.3.1/24`.
- Confirmed the interface addresses and tested connectivity across the three zones.
- Diagnosed blocked DMZ and Transit traffic: the relevant pings succeeded when packet filtering was temporarily disabled for testing. The firewall should remain enabled during normal operation.
- **In progress:** permanent rules allowing only the required traffic, followed by allowed/blocked test evidence.

**Evidence:** [OPNsense interface configuration](https://github.com/user-attachments/assets/4b30da8a-fc9b-4aa6-b27b-edbd3c60fbbd) · [DMZ diagnostic test](https://github.com/user-attachments/assets/2d441fff-12bc-4ccd-bf35-d79a6414d397)

### 3.4 Transit and Internet Access

- Configured VPNRouter1 eth0 as `10.20.3.2/24` and eth1 for DHCP through the NAT node.
- Confirmed its default route through `192.168.122.1` and successful pings to `8.8.8.8` and `google.com`.
- The initial VPNRouter1 → OPNsense transit ping succeeded. After reboot, that direction was blocked by OPNsense firewall policy; the reverse direction still worked.

**Evidence:** [Transit IP](https://github.com/user-attachments/assets/a0b9e19b-254a-4793-87ec-14b002219702) · [Routing table](https://github.com/user-attachments/assets/c18cf968-0512-418c-8d5c-a952f9991669) · [Internet test](https://github.com/user-attachments/assets/8682df09-2f2b-4abd-94bf-8c6275d9422e)

---

## 4. Federation Over Tailscale

- Connected VPNRouter1 to my Tailscale tailnet (`100.112.249.125`).
- Advertised and approved `10.20.0.0/16` in the Tailscale Admin Console.
- Enabled IPv4 forwarding and added routes for `10.20.1.0/24` and `10.20.2.0/24` via OPNsense `10.20.3.1`.
- Before reboot, VPNRouter1 successfully pinged PC1, the HTTPS Server and the SSH Server.
- Made eth0, eth1 DHCP, the LAN/DMZ routes, IPv4 forwarding and Tailscale startup persistent. After reboot, Tailscale, routing and Internet/DNS worked; the remaining internal ping issue was traced to OPNsense firewall policy.

**Verification commands:**

```bash
tailscale status
ip route
sysctl net.ipv4.ip_forward
ping -c 4 google.com
```

**Evidence to insert from my GNS3 screenshots:** Tailscale status; Tailscale Admin Console showing **approved** `10.20.0.0/16`; routes and successful internal pings; persistence and post-reboot checks. Verify each screenshot before linking it.

**Group integration:** Cross-site tests with Chelsea's `10.10.0.0/16` and Abiral's `10.30.0.0/16` will be recorded once the shared tailnet and firewall rules are ready.

---

## 5. Testing and Evidence

| Test | Result | Evidence |
|---|---|---|
| LAN devices ↔ one another and LAN gateway | Passed | [LAN ping tests](https://github.com/user-attachments/assets/a40fd7c6-47e3-4cf4-abb9-bfedbb36df69) |
| DMZ servers ↔ each other | Passed | Add matching screenshot |
| OPNsense → HTTPS Server | Passed | [Ping test](https://github.com/user-attachments/assets/4c71fce2-42f0-41d4-9216-0c7e8d6bc046) |
| DMZ → OPNsense with firewall enabled | Blocked; rule review pending | [Diagnostic test](https://github.com/user-attachments/assets/76e4b1e5-0b9f-4381-92c4-013ec9db56ca) |
| VPNRouter1 → OPNsense before reboot | Passed | [Transit ping](https://github.com/user-attachments/assets/387fab3d-352f-48c1-a444-6fa99ad4f1aa) |
| VPNRouter1 → LAN and DMZ before reboot | Passed | Add matching screenshots |
| Internet and DNS after VPNRouter1 restart | Passed | Add post-reboot screenshot |
| Permanent OPNsense rules and final retest | In progress | Pending |
| Cross-site Tailscale, separate site-to-site VPN, Kerberos and Suricata | Not yet documented as complete | Pending |

### 5.1 Packet Captures

Required VPN before/after captures and additional Kerberos, Suricata and HTTPS captures will be added to the group's `captures/` folder after those tests have been performed. No packet captures are claimed as completed here.

---

## 6. Remaining Work

- Finish OPNsense DMZ and Transit rules and repeat the allowed/blocked tests.
- Integrate and verify the Assessment 1 services at application level.
- Add and configure the Kerberos KDC and Suricata sensor for Site 2.
- Work with the group on cross-site Tailscale access and the separate IPsec/WireGuard site-to-site VPN.
- Collect packet captures, final screenshots and the GNS3 project export for submission.
