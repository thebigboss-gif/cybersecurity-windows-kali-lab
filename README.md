# Windows-Kali Cyber Lab

A hands-on cybersecurity lab built with **Kali Linux** and **Windows** to practice
authorized offensive security testing, network reconnaissance, service enumeration,
security controls, and — in later stages — detection and incident investigation.

---
# Project Objective

The goal of this project is to build an isolated virtual environment where
attacker activity can be safely simulated against a Windows target and then
analyzed from a defensive/SOC perspective.

The project follows an attack-to-detection workflow:

Reconnaissance
      ↓
Enumeration
      ↓
Controlled Attack Simulation
      ↓
Windows Security Telemetry
      ↓
Detection
      ↓
Investigation
      ↓
Mitigation

---

## Lab Environment

| Machine | Role | IP Address |
|---|---|---|
| Kali Linux | Attacker | `192.168.100.10` |
| Windows | Target | `192.168.100.20` |

The machines communicate through an isolated VirtualBox lab network
(`192.168.100.0/24`).

Kali uses a separate interface for Internet connectivity while the
`192.168.100.0/24` network is used for the cybersecurity lab.

---

## Project Progress

### Day 1 — Lab & Network Setup

- Configured Kali Linux and Windows virtual machines
- Configured the isolated lab network
- Assigned static IP addresses
- Verified connectivity between the virtual machines
- Created VM snapshots for recovery

### Day 2 — Network Reconnaissance & SMB Enumeration

- Performed Nmap reconnaissance against the Windows VM
- Investigated `open`, `closed`, and `filtered` port states
- Identified TCP/445 as SMB
- Compared Nmap results against Windows' local listening-port state
- Investigated Windows Firewall filtering
- Created a narrowly scoped firewall rule allowing TCP/445 from Kali
- Enumerated SMB protocol support
- Investigated Windows SMB shares
- Created a dedicated `CyberLab` SMB share
- Tested unauthenticated SMB access
- Confirmed anonymous access was denied

**Detailed Day 2 documentation:**  
`docs/day-02-smb-recon.md`

### Upcoming

- [ ] Install Sysmon
- [ ] Generate controlled attack activity
- [ ] Collect Windows security telemetry
- [ ] Investigate security events
- [ ] Develop detection logic
- [ ] Integrate a SIEM
- [ ] Create SOC-style alerts
- [ ] Document incident investigation
- [ ] Document mitigation and hardening

---

## Day 2 Highlights

The reconnaissance phase demonstrated an important distinction between
a service being **locally available** and being **network-accessible**.

For example:

```text
Windows:
TCP/445 → LISTENING

        ↓ Windows Firewall

Kali:
TCP/445 → FILTERED
```

After creating a firewall rule scoped specifically to the Kali VM:

```text
Windows:
TCP/445 → LISTENING

        ↓ Firewall allows 192.168.100.10

Kali:
TCP/445 → OPEN
```

SMB was then identified on TCP/445 and its protocol support was enumerated.

An unauthenticated SMB connection to the dedicated lab share was rejected:

```text
NT_STATUS_ACCESS_DENIED
```

This demonstrated that an **open port does not automatically mean
unauthorized access is possible**.

---

## Key Concepts

### Enumeration vs. Exploitation

**Enumeration** is the process of gathering information about a target,
such as:

- Open or filtered ports
- Running services
- Protocols
- Network shares
- Access controls

**Exploitation** is the subsequent attempt to take advantage of a
vulnerability or misconfiguration to perform an unintended action.

This project intentionally separates these stages.

```text
Reconnaissance
      ↓
Enumeration
      ↓
Identify potential weaknesses
      ↓
Controlled attack simulation
      ↓
Detection & investigation
```

---

## Security Concepts Demonstrated

- Network reconnaissance
- Port scanning
- Service enumeration
- Windows Firewall
- SMB security
- Authentication and access control
- Attack surface identification
- Security telemetry
- Detection engineering
- Incident investigation
- System hardening

---

## Repository Structure

```text
windows-kali-cyber-lab/
│
├── README.md
│
├── docs/
│   ├── day-01-lab-setup.md
│   └── day-02-smb-recon.md
│
├── screenshots/
│   ├── day-01/
│   └── day-02/
│
└── scans/
    └── day-02/
```

The detailed technical write-ups contain the commands, results,
screenshots, observations, and findings from each stage.

---

## Roadmap

The final goal is to demonstrate a complete security workflow:

```text
             ATTACKER
                │
                ▼
        Reconnaissance
                │
                ▼
          Enumeration
                │
                ▼
     Controlled Attack
        Simulation
                │
                ▼
             WINDOWS
                │
                ▼
      Security Telemetry
                │
                ▼
           Detection
                │
                ▼
       SOC Investigation
                │
                ▼
           Mitigation
```

Each stage will be documented as the project progresses.

---

## Disclaimer

All testing is performed against intentionally configured and authorized
systems inside an isolated virtual laboratory.

No systems outside the lab are intended to be scanned, accessed, or tested.
