# Day 1 — Cybersecurity Lab Setup

## Objective

Build an isolated Windows-Kali virtual laboratory that can be used for
authorized cybersecurity testing and security monitoring exercises.

The lab provides separate attacker and target environments while keeping
the testing environment isolated from the external network.

---

## Lab Architecture

```text
┌─────────────────────┐
│     Kali Linux      │
│      Attacker       │
│                     │
│ eth0: NAT           │
│ eth1: Lab Network   │
│ 192.168.100.10      │
└──────────┬──────────┘
           │
           │ 192.168.100.0/24
           │
┌──────────▼──────────┐
│       Windows       │
│       Target        │
│                     │
│ 192.168.100.20      │
└─────────────────────┘
```

## Virtual Machines

| Machine | Role | Lab IP |
|---|---|---|
| Kali Linux | Attacker | `192.168.100.10` |
| Windows | Target | `192.168.100.20` |

Kali uses two network interfaces:

- `eth0` — NAT/Internet connectivity
- `eth1` — isolated cybersecurity lab network

Windows uses the lab network interface.

---

## Network Configuration

### Kali Linux

The lab interface was configured with:

```text
Interface: eth1
IP Address: 192.168.100.10
Subnet Mask: 255.255.255.0
```

The lab interface does not require a default gateway because it is used
for direct communication with the Windows target.

### Windows

The Windows lab interface was configured with:

```text
IP Address: 192.168.100.20
Subnet Mask: 255.255.255.0
```

---

## Connectivity Testing

Connectivity between the two virtual machines was tested using ICMP.

From Windows:

```powershell
ping 192.168.100.10
```

The Windows machine successfully reached the Kali lab interface.

The Kali-to-Windows ping was initially blocked by Windows Firewall,
which was later investigated during Day 2.

---

## Isolation

The cybersecurity activities are performed inside the virtual laboratory
rather than against external systems.

The lab network uses the:

```text
192.168.100.0/24
```

address range.

This separation allows offensive-security techniques to be practiced
against an intentionally configured target without targeting unrelated
systems.

---

## VM Snapshots

Snapshots were created to provide recovery points before making further
changes to the virtual machines.

Recommended snapshot points include:

```text
Day1-Clean-Lab
Day2-Before-Sysmon
```

Snapshots allow the laboratory environment to be restored if a later
experiment changes or breaks the system configuration.

---

## Tools

The initial laboratory setup uses:

- VirtualBox
- Kali Linux
- Windows
- PowerShell
- Nmap
- SMB utilities

Additional security-monitoring tools will be introduced in later stages.

---

## Outcome

At the end of Day 1, the basic cybersecurity laboratory was operational:

- [x] Kali Linux VM configured
- [x] Windows VM configured
- [x] Isolated lab network established
- [x] Static IP addressing configured
- [x] Kali and Windows connectivity verified
- [x] VM snapshots created

The environment was then ready for Day 2 reconnaissance and enumeration.

---

## Next

**Day 2 — Network Reconnaissance & SMB Enumeration**

The next stage investigates Windows network exposure using Nmap,
SMB enumeration, and Windows Firewall analysis.
