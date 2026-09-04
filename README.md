# Network Service Discovery & Vulnerability Assessment Report

## Metadata
* **Target Host IP:** Metasploitable 2 VM (`192.168.6.132`)
* **Scan Source:** Kali Linux VM (`192.168.6.128`)
* **Tools Used:** Nmap 7.98
* **Scan Profile:** Service Enumeration (`nmap -sV 192.168.6.132`)

---

## 1. Executive Summary
A local area network discovery and comprehensive service enumeration scan was successfully conducted from the local testing host (`192.168.6.128`) against the target lab machine (`192.168.6.132`). The core operational objective of this assessment was to establish an active infrastructure baseline, map open perimeter communication vectors, and perform a structural gap analysis against hardened enterprise standards.

The analysis revealed a critical and broad attack surface consisting of **23 open TCP ports** running active services. The system configuration exhibits extreme security deviations, including legacy cleartext protocols, active backdoor software configurations, and unauthenticated administrative shells bound directly to the public-facing network interface.

---

## 2. Infrastructure Visualization

```text
 ┌──────────────────────┐               ┌──────────────────────────────┐
 │    Kali Linux VM     │  Scan Packet  │    Metasploitable 2 VM       │
 │   192.168.6.128      ├──────────────►│       192.168.6.132          │
 └──────────────────────┘               └──────────────┬───────────────┘
                                                       │
                                        ┌──────────────┴───────────────┐
                                        │ Enumerated Boundary Profile: │
                                        │  ► Cleartext (Ports 21, 23)  │
                                        │  ► DB Systems (Port 3306)    │
                                        │  ► Root Bindshell (Port 1524)│
                                        └──────────────────────────────┘
```

![Network Topology](images/Assignment 2.jpg)  
*Figure 1: Infrastructure Visualization*

![Ping Scan](images/sn1.png)  
*Figure 2: Verification of active network hosts running within the 192.168.6.0/24 subnet layer.*

---

## 3. Discovered Services & Port Mapping
The network service scan identified the host as active and running a diverse set of services. Below is the full table of the 23 open ports discovered by the Nmap version detection engine:

| Port / Protocol | State | Service | Software / Fingerprint |
| :--- | :--- | :--- | :--- |
| **21/tcp** | open | ftp | vsftpd 2.3.4 *(Malicious backdoor variant)* |
| **22/tcp** | open | ssh | OpenSSH 4.7p1 Debian 8ubuntu1 |
| **23/tcp** | open | telnet | Linux telnetd *(Cleartext administrative access)* |
| **25/tcp** | open | smtp | Postfix smtpd |
| **53/tcp** | open | domain | ISC BIND 9.4.2 |
| **80/tcp** | open | http | Apache httpd 2.2.8 (`(Ubuntu) DAV/2`) |
| **111/tcp** | open | rpcbind | 2 (`RPC #100000`) |
| **139/tcp** | open | netbios-ssn | Samba smbd 3.X - 4.X *(Workgroup: WORKGROUP)* |
| **445/tcp** | open | netbios-ssn | Samba smbd 3.X - 4.X *(Workgroup: WORKGROUP)* |
| **512/tcp** | open | exec | netkit-rsh rexecd |
| **513/tcp** | open | login | OpenBSD or Solaris rlogind |
| **514/tcp** | open | tcpwrapped | Generic security TCP wrap handler |
| **1099/tcp** | open | java-rmi | GNU Classpath grmiregistry |
| **1524/tcp** | open | bindshell | **Metasploitable root shell** *(No authentication)* |
| **2049/tcp** | open | nfs | 2-4 (`RPC #100003`) |
| **2121/tcp** | open | ftp | ProFTPD 1.3.1 |
| **3306/tcp** | open | mysql | MySQL 5.0.51a-3ubuntu5 |
| **5432/tcp** | open | postgresql | PostgreSQL DB 8.3.0 - 8.3.7 |
| **5900/tcp** | open | vnc | VNC *(Protocol 3.3 connection layer)* |
| **6000/tcp** | open | X11 | X11 graphical environment |
| **6667/tcp** | open | irc | UnrealIRCd platform |
| **8009/tcp** | open | ajp13 | Apache Jserv *(Protocol v1.3)* |
| **8180/tcp** | open | http | Apache Tomcat/Coyote JSP engine 1.1 |

---

## 4. Gap Analysis & Unexpected Exposure

![Comprehensive Service Enumeration Scan](images/sn2.png)  
*Figure 3: Complete Nmap 7.98 system service version detection execution output.*

Assuming a typical intended baseline of running a standard, secure production web server with isolated management capabilities, we find critical architectural deviations:

* **Severe Accidental Root Exposure (Port 1524):** A running `bindshell` exposes an unauthenticated raw root terminal over the network. Connecting via Netcat provides complete system takeover immediately without authentication.
* **Malicious Application Backdoor (Port 21):** The operational `vsftpd 2.3.4` service software contains an intentional backdoor that spawns a command shell on Port 6200 when a username string containing a smiley face `:)` is input.
* **Database Network Exposure (Ports 3306 & 5432):** MySQL and PostgreSQL are bound globally across network layers rather than terminating strictly on localhost loopback interfaces (`127.0.0.1`). This enables raw external network-level brute-forcing vectors.
* **Legacy Remote Protocol Stack (Ports 23, 512, 513):** Telnet and r-services transfer operational session details via unencrypted cleartext frames. A passive attacker monitoring traffic streams can easily extract active credentials.

---

## 5. Remediation Plan

### Short-Term Strategic Remediations
* **Disable Cleartext Networks:** Shut down the Telnet engine, `rexecd`, and `rlogind` instances immediately to eliminate cleartext credential harvesting vectors.
* **Isolate Administrative Shells:** Terminate the process layout binding the unauthenticated root shell to port 1524.
* **Reconfigure Database Listeners:** Enforce strict loopback interface bindings inside `my.cnf` and `postgresql.conf` to lock data platforms to `127.0.0.1` exclusively.

### Long-Term Security Engineering Lifecycles
* **Patch Optimization:** Upgrade obsolete software system distributions (Apache, vsftpd, Samba) to modern, actively supported vendor security baselines.
* **Host-Based Firewall Hardening:** Deploy a default-deny ingress packet filtering policy using `iptables` or `ufw` to protect open internal platform services from unauthorized routing ranges.
