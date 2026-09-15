# SecureCorp Cybersecurity Assessment Lab

A controlled cybersecurity assessment project covering **network discovery, web application security testing, authentication attacks, centralized monitoring with Wazuh, incident investigation, risk assessment, and remediation recommendations**.

> ⚠️ **Disclaimer:** This project was conducted only in an authorized, isolated lab environment. The techniques and attacks documented here must not be performed against systems without explicit authorization.

## 📌 Project Overview

This project simulates a security assessment for a fictional organization, **SecureCorp**.

The assessment follows the workflow:

```text
Build
  ↓
Discover
  ↓
Attack
  ↓
Log
  ↓
Detect
  ↓
Investigate
  ↓
Assess
  ↓
Remediate
  ↓
Report
```

The primary objective was to demonstrate how security weaknesses can be identified through controlled attacks and how a SIEM/HIDS platform such as **Wazuh** can be used to detect and investigate malicious activity.

---

## 🏗️ Lab Architecture

```text
                         Isolated Lab Network
                         172.20.10.0/24

        ┌──────────────────────────────┐
        │      Attacker Environment    │
        │          WSL / Linux         │
        │                              │
        │        172.20.10.5           │
        └──────────────┬───────────────┘
                       │
                       │ Security Testing
                       │
                       ▼
        ┌──────────────────────────────┐
        │       Metasploitable2        │
        │                              │
        │        172.20.10.3           │
        │                              │
        │  Apache / DVWA               │
        │  SSH / FTP / Other Services  │
        └──────────────┬───────────────┘
                       │
                       │ Logs / Security Events
                       │
                       ▼
        ┌──────────────────────────────┐
        │        Wazuh Server          │
        │                              │
        │        172.20.10.2           │
        │                              │
        │     Wazuh Manager            │
        └──────────────────────────────┘
```

### Systems

| System          | IP Address    | Role                                      |
| --------------- | ------------- | ----------------------------------------- |
| Attacker        | `172.20.10.5` | Discovery and controlled security testing |
| Metasploitable2 | `172.20.10.3` | Vulnerable target / DVWA                  |
| Wazuh Server    | `172.20.10.2` | Centralized monitoring and detection      |

The Metasploitable2 machine uses the legacy **OSSEC HIDS v2.8.3** agent because of its old operating system and architecture.

---

## 🔍 Assessment Scope

The assessment covered:

* Network and service discovery
* Web application mapping
* DVWA security testing
* Web brute force
* SQL Injection
* Reflected XSS
* Command Injection
* FTP brute force
* SSH brute force
* Wazuh monitoring
* Log correlation
* Incident investigation
* Risk assessment
* Security recommendations

All testing was restricted to the authorized lab target:

```text
172.20.10.3
```

---

## 🛠️ Tools Used

### Discovery

* **Nmap**

### Web Application Testing

* **DVWA**
* Web browser
* **Hydra**

### Monitoring & Investigation

* **Wazuh**
* **OSSEC HIDS v2.8.3**
* Apache logs
* SSH authentication logs
* vsftpd logs

### Operating Environment

* WSL/Linux attacker environment
* Metasploitable2
* Wazuh server VM

---

## 🔎 Network Discovery

Service enumeration was performed using:

```bash
nmap -sV 172.20.10.3
```

The scan identified services including:

```text
21/tcp    FTP
22/tcp    SSH
23/tcp    Telnet
25/tcp    SMTP
53/tcp    DNS
80/tcp    HTTP
139/tcp   NetBIOS
445/tcp   SMB
3306/tcp  MySQL
5432/tcp  PostgreSQL
5900/tcp  VNC
8180/tcp  Apache Tomcat
```

The HTTP service hosted the DVWA application used for controlled web-security testing.

---

## 💥 Attack Simulations

Six required attack scenarios were performed.

| Task | Attack               | Target      | Result                           | Wazuh Detection            |
| ---- | -------------------- | ----------- | -------------------------------- | -------------------------- |
| 2.3  | DVWA Web Brute Force | HTTP / DVWA | `admin:password` recovered       | Manual                     |
| 2.4  | SQL Injection        | HTTP / DVWA | Injection attempt detected       | **Automatic — Rule 31104** |
| 2.5  | Reflected XSS        | HTTP / DVWA | JavaScript executed              | Manual                     |
| 2.6  | Command Injection    | HTTP / DVWA | Module accessed/tested           | Manual                     |
| 2.7  | FTP Brute Force      | TCP/21      | Repeated authentication failures | **Automatic — Rule 11451** |
| 2.8  | SSH Brute Force      | TCP/22      | Repeated authentication failures | **Automatic — Rule 5760**  |

---

## 🛡️ Wazuh Detection

Wazuh was used to centralize and investigate security events generated during testing.

The assessment confirmed automatic detection for:

### SQL Injection

```text
Rule ID: 31104
```

Detected SQL injection-related HTTP input.

### FTP Brute Force

```text
Rule ID: 11451
```

Detected rapid consecutive FTP authentication failures.

### SSH Brute Force

```text
Rule ID: 5760
```

Detected repeated SSH authentication failures.

Other application-level attacks were identified through manual correlation of Apache logs, browser evidence, and attack-tool output.

---

## 📊 Incident Timeline

```text
02:58 AM  ── DVWA Web Brute Force
              │
03:01 AM  ── SQL Injection
              │
03:04 AM  ── Reflected XSS
              │
03:06 AM  ── Command Injection
              │
03:11 AM  ── FTP Brute Force
              │
03:22 AM  ── SSH Brute Force
```

All activities were authorized security-testing events and were therefore treated as **alerts/test events rather than actual production incidents**.

---

## 🚨 Key Findings

### 1. Weak Web Authentication

The DVWA brute-force test successfully recovered:

```text
Username: admin
Password: password
```

**Risk:** High

**Recommended controls:**

* Strong password policy
* Rate limiting
* Progressive login delays
* MFA
* Protection against common/default passwords

---

### 2. SQL Injection

SQL injection-related input was submitted to the vulnerable application and detected by Wazuh Rule `31104`.

**Risk:** Critical

**Recommended controls:**

* Prepared statements
* Parameterized queries
* Server-side input validation
* Least-privilege database accounts
* Safe error handling

---

### 3. Reflected XSS

The following test payload executed successfully:

```html
<script>alert('XSS')</script>
```

**Risk:** High

**Recommended controls:**

* Context-aware output encoding
* Server-side input validation
* Content Security Policy
* Secure cookie configuration

---

### 4. Command Injection

The DVWA Command Injection module was accessed and tested using a harmless command chain.

**Risk:** Critical

**Recommended controls:**

* Avoid direct OS command execution with user input
* Use safe APIs
* Allowlist input validation
* Least-privileged application processes

> The available assessment evidence establishes that the module was accessed and tested; it does not independently establish successful OS-level command execution.

---

### 5. FTP Brute Force

Repeated authentication failures were generated against:

```text
FTP — TCP/21
```

Wazuh detected the activity using Rule `11451`.

**Risk:** High

**Recommended controls:**

* Disable FTP when unnecessary
* Use SFTP
* Strong authentication
* Access restrictions
* Brute-force protection
* Centralized monitoring

---

### 6. SSH Brute Force

Repeated authentication failures were generated against:

```text
SSH — TCP/22
```

Wazuh detected the activity using Rule `5760`.

**Risk:** High

**Recommended controls:**

* SSH keys
* Strong credentials
* Disable password authentication where appropriate
* Restrict SSH access
* Rate limiting
* Wazuh monitoring

---

## 🔐 Security Recommendations

The main remediation areas identified by the assessment are:

| Area                     | Recommendation                                          | Priority     |
| ------------------------ | ------------------------------------------------------- | ------------ |
| Web Application Security | Fix SQL Injection and Command Injection vulnerabilities | **Critical** |
| Password Policy          | Enforce strong passwords and brute-force protection     | **High**     |
| MFA                      | Enable MFA for sensitive and administrative accounts    | **High**     |
| XSS Protection           | Implement output encoding, validation and CSP           | **High**     |
| SSH                      | Harden SSH and restrict access                          | **High**     |
| FTP                      | Disable FTP or migrate to SFTP                          | **High**     |
| Firewall                 | Restrict unnecessary exposed services                   | **High**     |
| Patch Management         | Regularly patch and remove legacy services              | **High**     |
| Logging & Monitoring     | Expand Wazuh application-level detection                | **High**     |
| RBAC                     | Apply least privilege                                   | **Medium**   |
| Backups                  | Maintain protected and tested backups                   | **Medium**   |
| Security Awareness       | Train users on credential and phishing risks            | **Medium**   |

---

## 📁 Project Structure

A suggested evidence/report structure:

```text
SecureCorp-Security-Assessment/
│
├── README.md
│
├── Report/
│   └── SecureCorp_Security_Assessment.pdf
│
├── Week1/
│   ├── Asset_Inventory/
│   ├── CIA_Analysis/
│   └── Risk_Assessment/
│
├── Week2/
│   ├── 2.1_Network_Discovery/
│   ├── 2.2_Web_Mapping/
│   ├── 2.3_DVWA_BruteForce/
│   ├── 2.4_SQL_Injection/
│   ├── 2.5_XSS/
│   ├── 2.6_Command_Injection/
│   ├── 2.7_FTP_BruteForce/
│   └── 2.8_SSH_BruteForce/
│
├── Week3/
│   ├── Detection/
│   ├── Investigation/
│   └── Incident_Timeline/
│
├── Week4/
│   └── Recommendations/
│
└── Evidence/
    ├── Screenshots/
    ├── Logs/
    ├── Wazuh/
    └── Attack_Output/
```

---

## 📚 Assessment Workflow

```text
                    SECURECORP ASSESSMENT
                            │
                            ▼
                    Asset Identification
                            │
                            ▼
                     CIA Analysis
                            │
                            ▼
                    Risk Assessment
                            │
                            ▼
                   Network Discovery
                            │
                            ▼
                   Web Application Map
                            │
                            ▼
                  Controlled Attack Tests
                    /    /    |    \    \
                   ▼    ▼     ▼     ▼    ▼
                Brute  SQLi   XSS   CMD  FTP/SSH
                   \    \     |     /    /
                    ▼    ▼    ▼    ▼    ▼
                       Log Collection
                            │
                            ▼
                     Wazuh Detection
                            │
                            ▼
                      Investigation
                            │
                            ▼
                       Risk Review
                            │
                            ▼
                    Remediation Plan
                            │
                            ▼
                         Report
```

---

## 🎯 Learning Outcomes

This project demonstrates practical understanding of:

* Network reconnaissance and service enumeration
* Web application vulnerability testing
* Authentication security
* Common web vulnerabilities
* Security logging
* Wazuh-based detection
* Log correlation and investigation
* Risk assessment
* Security remediation
* Professional security reporting

---

## ⚠️ Responsible Use

The attacks documented in this repository were performed in a **controlled and authorized cybersecurity laboratory** using intentionally vulnerable systems.

Do not reproduce these tests against systems, accounts, networks, or applications without explicit authorization.

---

## 👤 Project Context

**Project:** SecureCorp Cybersecurity Assessment
**Assessment Type:** Controlled Vulnerability Assessment & Detection Exercise
**Primary Focus:** Web Security, Network Security, Authentication, Wazuh Monitoring & Incident Investigation
**Environment:** Isolated Virtualized Lab

Available next action: Create a downloadable DOCX file here in this chat containing the editable prose above
