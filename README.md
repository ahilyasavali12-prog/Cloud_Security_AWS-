# Secure WordPress Deployment on AWS — Cloud Architecture & Security Assessment

> A production-grade, security-hardened WordPress application deployed on AWS EC2, assessed against industry-standard frameworks including NIST CSF, OWASP ASVS, CIS Benchmarks, and the AWS Well-Architected Framework.

---

## Overview

This project demonstrates the end-to-end design, deployment, and security assessment of a WordPress-based note-taking application hosted on **AWS EC2 (Amazon Linux 2023)**. The application supports user authentication, user registration, and per-user content isolation — representative of a public-sector use case involving PII and access control requirements.

A full **black-box penetration testing** cycle was conducted post-deployment, achieving an **87% reduction in total vulnerability count** and eliminating all Critical and High severity findings.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Cloud | AWS EC2 (t3.micro), VPC, Security Groups, KMS, Elastic IP |
| OS | Amazon Linux 2023 (AL2023) |
| Web Server | Apache (mod_rewrite, HTTPS via Let's Encrypt) |
| Application | WordPress (LAMP stack) |
| Monitoring | Datadog Agent, Splunk Universal Forwarder |
| Security | Wordfence WAF, Fail2Ban, WP 2FA, firewalld, SELinux |

---

## Security Architecture — Defence in Depth

Controls were applied across five independent layers:

```
┌─────────────────────────────────────────────────┐
│  Network Perimeter                               │
│  AWS Security Groups — ports 22, 80, 443 only   │
│  SSH locked to specific IP                       │
├─────────────────────────────────────────────────┤
│  Operating System                                │
│  SSH hardening, non-root user, firewalld,        │
│  Fail2Ban brute-force protection                 │
├─────────────────────────────────────────────────┤
│  Web Server                                      │
│  Apache virtual host, mod_rewrite, HTTPS/TLS,   │
│  security header injection                       │
├─────────────────────────────────────────────────┤
│  Application                                     │
│  Wordfence WAF, login URL obfuscation,           │
│  XML-RPC disabled, MFA via WP 2FA               │
├─────────────────────────────────────────────────┤
│  Monitoring & Detection                          │
│  Datadog (metrics/logs), Splunk SIEM alerting   │
└─────────────────────────────────────────────────┘
```

---

## Penetration Testing

Testing followed a **black-box methodology** using industry-standard tools:

| Tool | Version | Purpose |
|---|---|---|
| Nmap | 7.94 | Network port scanning and service enumeration |
| Nessus Essentials | 10.7 | Comprehensive vulnerability scanning |
| WPScan | 3.8.25 | WordPress-specific security scanning |
| SQLMap | — | SQL injection testing |
| Hydra | — | Brute-force resistance testing |
| Burp Suite | — | HTTP traffic interception and analysis |

---

## Results

| Assessment | Pre-Hardening | Post-Hardening | Outcome |
|---|---|---|---|
| Nmap | Open ports including TRACE enabled | HTTPS redirect only; TRACE disabled | Hardened |
| Nessus | 23 findings (Critical/High/Medium) | 0 findings | 100% remediation |
| WPScan | Directory listing, public registration, XML-RPC exposed | All issues resolved | 100% WordPress hardening |
| HTTP Headers | F rating | A rating | Full implementation |

**Overall: 87% reduction in total vulnerability count. All Critical and High severity findings eliminated.**

---

## Compliance Frameworks

- **NIST Cybersecurity Framework (CSF)**
- **OWASP Application Security Verification Standard (ASVS)**
- **CIS Benchmarks for Linux**
- **AWS Well-Architected Framework — Security Pillar**

---

## Infrastructure Setup

### EC2 Provisioning
```bash
# Instance: t3.micro | AMI: Amazon Linux 2023
# Storage: 10GB gp3 EBS (KMS encrypted)
# Network: Custom VPC, public subnet, Elastic IP

ssh -i wordpress.pem ec2-user@<your-elastic-ip>
```

### LAMP Stack Installation
```bash
# Update system
sudo dnf update -y

# Install Apache, MariaDB, PHP
sudo dnf install -y httpd mariadb105-server php php-mysqlnd php-fpm

# Start and enable services
sudo systemctl start httpd mariadb
sudo systemctl enable httpd mariadb
```

### HTTPS Configuration
```bash
# Install Certbot for Let's Encrypt TLS
sudo dnf install -y certbot python3-certbot-apache
sudo certbot --apache -d yourdomain.com
```

---

## Key Hardening Steps

```bash
# Disable XML-RPC (WordPress)
# Add to .htaccess:
<Files xmlrpc.php>
  Order Deny,Allow
  Deny from all
</Files>

# SSH hardening (/etc/ssh/sshd_config)
PermitRootLogin no
PasswordAuthentication no
AllowUsers ec2-user

# Fail2Ban for brute-force protection
sudo dnf install -y fail2ban
sudo systemctl enable --now fail2ban
```

---

## Project Structure

```
aws-wordpress-security/
├── docs/
│   ├── architecture-diagram.png
│   ├── penetration-testing-report.pdf
│   └── hardening-checklist.md
├── scripts/
│   ├── lamp-install.sh
│   ├── wordpress-setup.sh
│   └── hardening.sh
└── README.md
```

---

## Academic Context

**Module:** Cloud Architecture and Security — MSc Cybersecurity
**Institution:** National College of Ireland, Dublin
**Submission:** March 2026
**Team:** Group E (Ahilya Sarnaik, Ravi Rameshkumar Mishra, Varun Poojari)

---

## Disclaimer

This project was conducted in a controlled academic environment on infrastructure owned by the project team. All penetration testing was performed with explicit authorisation. Do not use these techniques on systems without written permission.
