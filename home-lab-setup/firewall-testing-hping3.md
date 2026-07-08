# Firewall Detection & Fingerprinting with hping3

**Lab environment:** Kali Linux (attacker) ↔ Metasploitable2 (target), both bridged on local LAN
**Tools used:** `hping3`, `iptables`
**Goal:** Identify how different firewall rule types (ACCEPT, REJECT, DROP) appear from an external scanner's perspective, and understand why this matters for defensive/SOC analysis.

---

## Background

When scanning a host, the response (or lack of one) reveals what kind of filtering is in place:

| Scenario | Expected Response |
|---|---|
| Open port, no firewall | SYN → SYN/ACK |
| Closed port, no firewall | SYN → RST/ACK |
| Filtered (DROP) | SYN → *no response* |
| Filtered (REJECT) | SYN → RST/ACK or ICMP unreachable |

The key distinction for a defender/analyst: **DROP vs REJECT is a policy choice with real tradeoffs.** REJECT looks identical to a genuinely closed port, while DROP is a clear signal that a firewall is present and actively filtering.

---

## Step 1 — Baseline (no firewall)

Metasploitable2's `iptables` was confirmed to be fully open (default ACCEPT on all chains, no rules):

```
Chain INPUT (policy ACCEPT)
Chain FORWARD (policy ACCEPT)
Chain OUTPUT (policy ACCEPT)
```

Scan against port 80 (web service):

```
sudo hping3 -S -p 80 -c 3 192.168.0.102
```

**Result:**
```
flags=SA seq=0 win=5840 rtt=1.0 ms
flags=SA seq=1 win=5840 rtt=1.7 ms
flags=SA seq=2 win=5840 rtt=1.2 ms
```

`SA` (SYN-ACK) on every packet confirms port 80 is open with no filtering.

> 📸 **Screenshot placeholder:** Kali terminal showing the `hping3 -S -p 80` command and output above.

---

## Step 2 — REJECT rule

Added on Metasploitable2:
```
sudo iptables -A INPUT -p tcp --dport 8080 -j REJECT --reject-with tcp-reset
```

Scan:
```
sudo hping3 -S -p 8080 -c 3 192.168.0.102
```

**Result:**
```
flags=RA seq=0 win=0 rtt=1.8 ms
flags=RA seq=1 win=0 rtt=3.5 ms
flags=RA seq=2 win=0 rtt=2.4 ms
```

`RA` (RST-ACK) with `win=0` — active reset. **Indistinguishable from a plain closed port with no firewall at all.**

> 📸 **Screenshot placeholder:** Kali terminal showing the `hping3 -S -p 8080` command and `flags=RA` output.
>
> 📸 **Screenshot placeholder:** Metasploitable2 terminal showing `sudo iptables -L INPUT -n -v` with the REJECT rule visible.

---

## Step 3 — DROP rule

Added on Metasploitable2:
```
sudo iptables -A INPUT -p tcp --dport 9090 -j DROP
```

Scan:
```
sudo hping3 -S -p 9090 -c 3 192.168.0.102
```

**Result:**
```
3 packets transmitted, 0 packets received, 100% packet loss
```

Zero replies of any kind — the textbook silent-firewall signature.

> 📸 **Screenshot placeholder:** Kali terminal showing the `hping3 -S -p 9090` command with `100% packet loss`.
>
> 📸 **Screenshot placeholder:** Metasploitable2 terminal showing `sudo iptables -L INPUT -n -v` with all three rules (REJECT + DROP) visible together.

---

## Summary Table

| Port | Rule | hping3 Signature | Interpretation |
|---|---|---|---|
| 80 | ACCEPT (default) | `flags=SA` | Open, service responding |
| 8080 | REJECT (tcp-reset) | `flags=RA`, win=0 | Closed *or* firewall-rejected — cannot be distinguished by the scanner |
| 9090 | DROP | 100% packet loss, no reply | Actively filtered by a firewall |

---

## Key Takeaway

From an attacker's/scanner's point of view:
- **REJECT** rules blend in with normal closed ports — a stealthier defensive choice in some contexts.
- **DROP** rules are actually *more* revealing on close inspection, since consistent unexplained timeouts (versus RSTs elsewhere on the same host) are a recognizable firewall fingerprint. Tools like `nmap` explicitly report this distinction as `closed` vs `filtered`.

Understanding this tradeoff is directly relevant to SOC work: recognizing DROP vs REJECT patterns in scan logs or IDS alerts helps identify reconnaissance activity and assess how a target's firewall posture might be tuned (and how it might unintentionally reveal information despite being "more restrictive").
