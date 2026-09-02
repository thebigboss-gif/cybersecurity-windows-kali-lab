# Day 2 — Network Reconnaissance & SMB Enumeration

A hands-on lab demonstrating SMB service enumeration methodology, host-based firewall
behavior, and secure SMB configuration validation on an isolated Windows/Kali network.

## Objective

Investigate how a host-based firewall affects external reconnaissance, and practice
validating scanner output against ground truth on the target itself — rather than
trusting a single tool's perspective. The lab also examines how a properly hardened
SMB configuration resists anonymous enumeration even when the port is reachable.

## Environment

- **Attacker:** Kali Linux (VirtualBox VM)
- **Target:** Windows (VirtualBox VM)
- **Network:** Isolated host-only/internal VirtualBox network (`192.168.100.0/24`) —
  no internet-facing exposure
- **Tools:** `nmap`, `smbclient`, PowerShell (`Get-NetTCPConnection`,
  `Get-SmbServerConfiguration`, `Get-NetFirewallRule`, `New-SmbShare`)

## Methodology

### 1. Initial reconnaissance

```
sudo nmap -Pn -sV 192.168.100.20
```

`-Pn` skips host-discovery pings (Windows commonly drops ICMP), `-sV` attempts
service/version detection.

**Result:** all 1000 scanned ports reported as `filtered` — no response at all, not
even a reset. This pattern is a strong indicator of a host-based firewall silently
dropping unsolicited traffic, rather than the ports genuinely being closed.

![nmap filtered results](screenshots/nmap-filtered.png)

### 2. Validating against ground truth on the target

```
Get-NetTCPConnection -State Listen | Sort-Object LocalPort | Format-Table LocalAddress,LocalPort,OwningProcess
```

Confirms Windows *is* listening on ports 135, 139, 445, 5040, and several dynamic RPC
ports (49664–49669). This proves the earlier "filtered" result was a firewall
artifact, not an absence of services — an important distinction when interpreting
scan output.

### 3. Confirming the block from Kali

```
sudo nmap -Pn -p 135,139,445,5040,49664-49669 192.168.100.20
```

Re-scanning specifically the ports Windows reports as listening still shows every
one as `filtered`, confirming the block is enforced consistently against external
traffic.

### 4. Scoped firewall exception

```powershell
New-NetFirewallRule -DisplayName "CyberLab SMB Inbound" -Direction Inbound `
  -Protocol TCP -LocalPort 445 -RemoteAddress 192.168.100.10 -Action Allow
```

Rather than disabling the firewall, this opens the minimum necessary surface:
TCP/445 only, inbound only, restricted to the Kali host's specific IP. A rescan
confirms port 445 now shows `open`.

![firewall rule scoped to single host](screenshots/firewall-rule.png)

### 5. Service enumeration

```
sudo nmap -Pn -sV -p 445 192.168.100.20
```

Confirms `microsoft-ds`, the service name associated with SMB over TCP 445.

### 6. Passive SMB probing (no exploitation)

```
smbclient -L //192.168.100.20 -N
```

`-N` attempts a null/anonymous session with no credentials, purely to see what the
server advertises.

**Result:** `NT_STATUS_CONNECTION_RESET` — the server actively tore down the
connection during negotiation.

### 7. Diagnosing the reset, on the target

```powershell
Get-SmbServerConfiguration | Select-Object EnableSMB1Protocol,EnableSMB2Protocol,RejectUnencryptedAccess
Get-NetFirewallRule -DisplayName "CyberLab SMB Inbound" | Format-List DisplayName,Enabled,Direction,Action,Profile
```

Shows SMB1 disabled, SMB2 enabled, and `RejectUnencryptedAccess = True`. The
firewall rule itself is confirmed enabled and correctly scoped — so the reset traces
to SMB-layer hardening, not the firewall.

### 8. Confirming supported dialects

```
sudo nmap -Pn -p 445 --script smb-protocols 192.168.100.20
```

Confirms the server negotiates modern dialects (SMB 2.0.2 through 3.1.1), ruling out
a protocol-support gap as the cause of the earlier reset.

### 9. Standing up a real share to test against

```powershell
New-Item -ItemType Directory -Path "C:\CyberLab" -Force
"CyberSecurity Lab Test File" | Out-File "C:\CyberLab\text.txt"
New-SmbShare -Name "CyberLab" -Path "C:\CyberLab" -Description "Cybersecurity training lab share"
```

### 10. Testing anonymous access to the share

```
smbclient //192.168.100.20/CyberLab -N
```

**Result:** `NT_STATUS_ACCESS_DENIED` — the connection now succeeds at the protocol
level, confirming the share exists and is reachable, but anonymous/null-session
access is explicitly rejected.

![access denied to CyberLab share](screenshots/smb-access-denied.png)

## Findings

| Area | Observation | Assessment |
|---|---|---|
| Network exposure | TCP 445 filtered by default, opened intentionally and narrowly for this lab | Correct default posture |
| SMB hardening | SMB1 disabled, SMB2/3 enabled, `RejectUnencryptedAccess = True` | Aligned with current best practice |
| Anonymous enumeration | Null-session listing and share access both rejected | Share-level ACLs enforcing least privilege |

**Recommendation:** maintain current configuration — SMB1 remains disabled, and
share access requires authentication. No remediation needed based on this lab's
scope.

## Key takeaways

- **Filtered ≠ closed.** A scanner's "filtered" result only describes what's visible
  from the outside; validating against the host's own listening state
  (`Get-NetTCPConnection`) is necessary before concluding a service is absent.
- **Open port ≠ accessible service.** Port 445 being reachable didn't mean SMB was
  exploitable — protocol-level hardening and share ACLs are independent, stacked
  layers of defense.
- **Scope firewall exceptions tightly.** Restricting the lab's allow-rule to a single
  protocol, port, and source IP kept the test environment realistic rather than
  just disabling protections wholesale.

## Next steps

- Authenticated SMB enumeration with valid lab credentials
  (`smbclient -U labuser //192.168.100.20/CyberLab`)
- Compare `NT_STATUS_ACCESS_DENIED` vs. `NT_STATUS_LOGON_FAILURE` responses and what
  each implies about share vs. account misconfiguration
- Deliberately misconfigure a share (e.g., `Everyone: Read`) to document the
  contrast with the hardened baseline above

## Disclaimer

All testing was performed against a personally owned, isolated lab environment
(VirtualBox host-only network) with no internet-facing exposure. No systems outside
this lab were scanned or accessed.

