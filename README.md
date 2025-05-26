# Local Network Reconnaissance Using Nmap & Wireshark

## Objective
To scan a local Wi-Fi network (`192.168.1.0/24`) using Nmap to detect live hosts and open ports, and analyze network-level behavior using Wireshark. The goal is to identify potential vulnerabilities, particularly exposed services.

---

## Tools & Environment

| Tool       | Purpose                       |
|------------|-------------------------------|
| **Kali Linux** | OS for running all scans (in VirtualBox, Bridged mode) |
| **Nmap 7.95** | Network scanner for port discovery |
| **Wireshark** | Packet analyzer to inspect traffic |
| **xsltproc** | Convert Nmap XML to HTML for report |

---

## Network Details

- **Scanned Range**: `192.168.1.0/24`
- **Hosts Detected**: 6
- **Interface Used**: `eth0` (Bridged Adapter)
- **Internal IP (Kali)**: `192.168.1.6`

---

## Commands Used

### Nmap Scan
```bash
sudo nmap -sS 192.168.1.0/24 -oX scan.xml
xsltproc scan.xml -o scan.html
```
- **-sS**: TCP SYN (stealth) scan

- **-oX**: Output in XML format

- **scan.html**: Human-readable report

### Wireshark
- Interface: eth0

- Filter applied: tcp.port == 3306

- File saved as: capture.pcapng

## Scan Results Summary (Nmap)

| IP Address     | Open Ports | Services    | Risk Level | Notes                                      |
| -------------- | ---------- | ----------- | ---------- | ------------------------------------------ |
| `192.168.1.1`  | 80, 443    | HTTP, HTTPS |  Medium  | Router interface accessible via HTTP/HTTPS |
| `192.168.1.2`  | 80         | HTTP        | Medium  | Possible IoT/printer — needs validation    |
| `192.168.1.3`  | None       | —           | Low      | All ports closed                           |
| `192.168.1.6`  | 3306       | MySQL       | High    | MySQL DB exposed — major security risk     |
| `192.168.1.8`  | None       | —           | Low      | All ports closed                           |
| `192.168.1.15` | None       | —           | Low      | All ports filtered (possibly firewalled)   |


## Packet Analysis (Wireshark)
### Filter Used
```bash
tcp.port == 3306
```
### Observations
- 192.168.1.6 responded with SYN-ACK to incoming SYNs on port 3306, confirming MySQL is open and reachable.
- Other hosts like 192.168.1.2 and 192.168.1.15 returned RST or did not respond — indicating the port is closed or filtered.

### Interpretation
- The presence of SYN → SYN-ACK → RST confirms an open port.
- All other patterns (SYN → RST) indicate closed ports.

# Security Recommendations
## Router (192.168.1.1)
- Disable HTTP, enforce HTTPS only
- Set strong admin credentials
- Turn off remote web access

## MySQL (192.168.1.6)
- Block external access to port 3306
- Bind MySQL to localhost
- Require strong passwords and TLS
- Disable remote root login

## Other Hosts
- Confirm ownership of unknown MAC addresses
- Disable unused services
- Apply firmware and OS updates

# Project Files Included
| File                  | Description                            |
| --------------------- | -------------------------------------- |
| `scan.html`           | Formatted Nmap report for web viewing  |
| `capture.pcapng`      | Wireshark capture of Nmap scan traffic |
| `README.md`           | This documentation                     |

# Conclusion
This project successfully mapped the internal network, revealing multiple active devices and one critical exposure: an open MySQL service on 192.168.1.6. The use of both Nmap and Wireshark allowed for thorough service discovery and traffic-level verification. The findings demonstrate the importance of closing unused ports, limiting remote access, and hardening network-facing services.
