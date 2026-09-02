Security Controls

Purpose

This document summarizes the security controls implemented in the AWS
Secure Web Infrastructure & Incident Response Lab.

Network Security

VPC Segmentation

The lab uses a dedicated VPC:

SecureShield-VPC
10.0.0.0/16

The web workload is placed in:

Public-Web-Subnet
10.0.1.0/24

The public subnet is associated with a route table containing an
Internet Gateway route.

Security Group

WEB-SG follows a limited inbound-access model.

HTTP  TCP/80  0.0.0.0/0
SSH   TCP/22  ADMIN-IP/32

HTTP is public because the server provides a public web service. SSH is
restricted to one trusted administrator source address.

Identity Security

EC2 IAM Role

WEB-SERVER-01 uses:

SecureShield-EC2-SecurityReadOnly

The workload obtains temporary AWS credentials through its IAM role
rather than storing long-lived access keys locally.

Least Privilege

When CloudTrail lookup access was required, the role received only:

cloudtrail:LookupEvents

Broad policies such as AdministratorAccess were intentionally avoided.

Audit Logging

AWS CloudTrail is configured to record management activity.

Trail:

SecureShield-Audit-Trail

The lab uses CloudTrail to investigate changes to AWS resources,
including Security Group modifications.

Log-file validation was enabled during trail configuration.

Log Storage

CloudTrail logs are stored in a dedicated S3 security-log bucket.

Controls include:

Block Public Access

Server-side encryption

ACLs disabled

Versioning, if enabled in the deployed environment

The real bucket name should be sanitized in public documentation.

Incident Response

A controlled Security Group misconfiguration was used to test
investigation and remediation.

The workflow was:

Baseline
   ↓
Controlled Misconfiguration
   ↓
CloudTrail Evidence
   ↓
Identity / Source Investigation
   ↓
Remediation
   ↓
Verification

Credential Security

The following must never be committed to the repository:

*.pem
.env
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_SESSION_TOKEN

AWS account IDs, personal IP addresses, and sensitive CloudTrail data
should also be redacted from public screenshots.

GuardDuty

GuardDuty was evaluated but should not be described as implemented
unless it is successfully enabled and verified in the AWS account.

Future Security Enhancements

Private application subnet/workload

Security Group-to-Security Group access

CloudWatch metric filters and alarms

SNS security notifications

AWS Config

GuardDuty when appropriate

Terraform Infrastructure as Code
