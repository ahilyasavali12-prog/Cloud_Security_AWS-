# AWS Secure Web Infrastructure & Incident Response Lab

A demonstrable, production-aligned cloud security project featuring a hardened public-facing web server, least-privilege IAM management, centralized audit logging, and a simulated incident response lifecycle using AWS CloudTrail telemetry.

---

## Technical Overview

* **Target Roles:** Cloud Support Engineer, Junior Cloud Security Engineer, SOC Analyst, Infrastructure Support
* **Cloud Provider:** Amazon Web Services (AWS)
* **Region:** `eu-west-1` (Ireland)
* **Core Services:** AWS VPC, EC2, IAM, STS, CloudTrail, S3, AWS CLI, Amazon Linux 2023, Nginx

This lab demonstrates practical implementation of cloud network isolation, instance hardening, non-custodial credential practices via IAM Roles, least-privilege policy iteration, and forensic investigation of an unauthorized network exposure.

---

## Architecture

```text
                         INTERNET
                            │
                            │ HTTP :80
                            ▼
                    Internet Gateway
                            │
                            ▼
              ┌─────────────────────────┐
              │   SecureShield-VPC      │
              │     10.0.0.0/16        │
              │                         │
              │   Public-Web-Subnet     │
              │     10.0.1.0/24        │
              │           │             │
              │           ▼             │
              │       ┌─────────┐       │
Admin IP ───SSH:22───►│ WEB-SG  │       │
              │       └────┬────┘       │
              │            ▼            │
              │    ┌────────────────┐   │
              │    │ WEB-SERVER-01  │   │
              │    │ Amazon Linux   │   │
              │    │ Nginx          │   │
              │    └───────┬────────┘   │
              └────────────┼────────────┘
                           │
                     IAM Instance Role
                           │
                           ▼
                  Temporary Credentials

                     AUDIT / SECURITY
                           │
              ┌────────────┴────────────┐
              ▼                         ▼
          CloudTrail                    S3
              │                  Security Logs
              ▼
       Event Investigation
              │
              ▼
       Incident Response
