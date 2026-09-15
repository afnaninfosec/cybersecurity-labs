# CH4IN4RULZ V1 Penetration Test Summary

## Scope and authorization

This assessment was completed solely against the CH4IN4RULZ V1 vulnerable virtual machine from VulnHub in an isolated training lab. The work was performed for a CEH Diploma final project. No systems outside the lab environment were targeted.

## Objective

The objective was to apply a structured penetration-testing methodology to identify exposed services, enumerate the web application, obtain an initial foothold, and demonstrate the impact of weak application and system controls.

## Methodology

The assessment followed a practical sequence:

1. **Reconnaissance and enumeration** - Identified the target host, exposed TCP services, service versions, and web application entry points.
2. **Service assessment** - Reviewed the anonymous FTP configuration and assessed the exposed HTTP and development services for accessible content and misconfigurations.
3. **Web application discovery** - Used content discovery and web-server scanning to identify API endpoints, backup content, authentication prompts, and an upload function.
4. **Credential analysis** - Identified a credential record from exposed application content and validated it in the authorized lab after offline password-hash analysis.
5. **Initial access** - Assessed the upload workflow and file-path handling, then used the identified weakness to obtain a controlled shell in the lab.
6. **Privilege escalation** - Performed local enumeration after initial access, identified the operating-system context, and demonstrated escalation to administrative access in the target VM.

## Tools used

| Tool | Purpose in the lab |
| --- | --- |
| Nmap | Host discovery, port scanning, service detection, and version enumeration |
| FTP client | Validation of anonymous FTP access and available content |
| Web browser and developer tools | Manual inspection of HTTP services and application behaviour |
| DirBuster and Gobuster | Discovery of directories, files, and upload-related paths |
| Nikto | Web-server configuration and content scanning |
| cURL | Request testing and endpoint interaction |
| John the Ripper | Offline analysis of the recovered password hash |
| Burp Suite | Interception and modification of authorized HTTP requests |
| Netcat | Controlled listener for the lab shell connection |
| Metasploit Framework and msfvenom | Payload handling and controlled post-exploitation validation |
| Kali Linux | Assessment platform and tooling environment |

## Key findings

The lab machine demonstrated how multiple lower-level weaknesses can combine into a full compromise path:

- Unnecessary or weakly protected exposed services increased the attack surface.
- Web content and backup artefacts exposed information useful for further enumeration.
- Weak credential protection enabled reuse of recovered credentials in the development environment.
- Insufficient upload validation and unsafe file-path handling enabled unauthorized code execution in the lab.
- Post-exploitation configuration weaknesses allowed privilege escalation from an initial low-privilege shell to root-level access.

## Lessons learned

This exercise reinforced the value of thorough enumeration before exploitation. Small observations, such as an exposed development service, backup file, or upload function, can become meaningful when connected in sequence. It also highlights defensive priorities: minimize exposed services, remove development artefacts, store passwords using strong salted hashes, validate uploads server-side, restrict file access, and apply least-privilege controls.

## Supporting report

The full project report, including screenshots and supporting evidence, is available in [CH4IN4RULZ V1 Penetration Test Report](ch4in4rulz-v1-penetration-test-report.docx).

## Disclaimer

This write-up is for educational purposes and documents activity performed only in an isolated, intentionally vulnerable virtual lab. Do not use these techniques against systems without explicit authorization.
