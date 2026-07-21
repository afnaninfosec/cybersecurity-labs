# Metasploitable2 - FTP Service Assessment

## Scope
- Target: Metasploitable2 VM (isolated lab environment, own VMware host-only/bridged network)
- Attacker box: Kali Linux VM (same isolated lab network)
- Authorization: Self-owned lab environment, no external systems involved
- Objective: Practice enumeration and exploitation of a deliberately vulnerable FTP service as part of home-lab / diploma coursework

## Tools Used
- nmap - port/service enumeration
- Metasploit Framework (msfconsole) - exploitation
- Manual ftp client - credential validation
- netcat / socat - attempted manual session catch

---

## Finding 1 - Weak / Default FTP Credentials (Confirmed)

Service: FTP (port 21/tcp)
Vulnerability: Default account credentials (msfadmin:msfadmin) accepted without restriction

### Steps
1. Enumerated open ports: nmap -p21 -sV <target_IP>
2. Connected via FTP client and authenticated with known default Metasploitable2 credentials (msfadmin:msfadmin) - received "230 Login successful"
3. Confirmed read access to the filesystem via ls / get

### Evidence
- Screenshot: successful FTP login (230 Login successful)
- Screenshot: file listing / file transfer confirming access

### Impact
Default or weak credentials on a network-facing service allow any attacker who knows or guesses the default account to gain authenticated access without needing to exploit a software vulnerability at all. This is one of the most common findings in real-world assessments.

### Remediation
- Remove or disable default accounts before deployment
- Enforce strong, unique credentials per service
- Disable anonymous and plaintext-credential FTP
- Restrict FTP access to trusted network segments only

---

## Finding 2 - vsftpd 2.3.4 Backdoor / CVE-2011-2523 (Attempted, Inconclusive)

Service: FTP (port 21/tcp), vsftpd 2.3.4
Vulnerability: Known intentionally-backdoored build of vsftpd 2.3.4, distributed briefly in 2011. A crafted login sequence spawns a root shell listener on port 6200/tcp

### Steps
1. Confirmed vulnerable banner via nmap -p21 -sV <target_IP>: "220 (vsFTPd 2.3.4)"
2. Ran Metasploit module: use exploit/unix/ftp/vsftpd_234_backdoor, set RHOSTS <target_IP>, run
3. Module confirmed target vulnerable and reported backdoor spawning on port 6200/tcp each run, but reported "Exploit completed, but no session was created"
4. Attempted to manually catch the spawned shell via netcat and socat against port 6200. Port consistently showed open, but no interactive shell output was returned across multiple timing attempts

### Evidence
See screenshots/vsftpd_backdoor_attempt.png - terminal log showing module loaded, exploit run, backdoor spawned, followed by repeated session attempts returning "Invalid session identifier"

### Outcome
The vulnerability was confirmed present, but a stable interactive session was not obtained in this attempt. This is a documented limitation of this specific Metasploit module against certain listener timing conditions, not an indication the target was unaffected.

### Remediation
- Never deploy software from unverified/unofficial build sources
- Verify package checksums/signatures before installation
- Patch to a clean, non-backdoored vsftpd release
- Monitor for unexpected listening ports as an indicator of compromise

---

## Lessons Learned
- Confirmed that default credentials remain a fully viable attack path even on services with no other apparent vulnerabilities
- Learned that vulnerability confirmation and successful exploitation are not always the same outcome
- Practiced distinguishing between commands run inside a tool's console versus a genuine remote/interactive shell
