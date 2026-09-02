🛡️ AWS Secure Web Infrastructure & Incident Response Lab

A hands-on AWS cloud security project demonstrating secure
infrastructure deployment, least-privilege IAM, audit logging, and
investigation of a simulated cloud security misconfiguration.

📌 Project Overview

Imagine a small company has launched its first public-facing web
application in AWS.

The application must be reachable by customers over the Internet, but
the infrastructure team also needs to ensure that:

administrative access is tightly restricted,

AWS credentials are not stored on the server,

infrastructure changes can be audited,

security misconfigurations can be investigated,

and unnecessary permissions are avoided.

This project builds that environment and then introduces a controlled
security incident to demonstrate how AWS audit evidence can be used
during an investigation.

The project is designed as a practical portfolio lab for Cloud
Security, SOC, Cloud Support, Systems Administration, and Infrastructure
Security roles.

🎯 Project Objectives

The goal was not simply to launch an EC2 instance. The lab was designed
around a realistic security workflow:

DESIGN → DEPLOY → HARDEN → MONITOR → SIMULATE → INVESTIGATE → REMEDIATE

The project demonstrates:

AWS VPC networking

Public subnet design

Internet Gateway and routing

Amazon EC2 deployment

Linux and Nginx administration

Security Group hardening

IAM roles and temporary AWS credentials

Least-privilege IAM permissions

AWS CloudTrail auditing

Private S3 audit-log storage

AWS CLI investigation

Cloud incident response

🏢 Business Scenario

SecureShield Enterprise

SecureShield Enterprise is a fictional small organization migrating
a public web service to AWS.

The business requires a simple public website, but the security team has
established several requirements.

Business Requirements

Requirement                        Security Decision

Customers must access the website  Allow HTTP/80

Administrators need remote         Allow SSH only from trusted admin
management                         IP

AWS API access is required from    Use an IAM instance role
EC2

Long-lived credentials should not  Use temporary role credentials
exist on the server

Infrastructure changes must be     Enable CloudTrail
traceable

Audit records must be retained     Store CloudTrail logs in S3

The resulting architecture intentionally stays small enough to
understand completely while still demonstrating real AWS security
concepts.

🏗️ Architecture



Environment

Resource             Configuration

AWS Region           eu-west-1 --- Ireland
VPC                  SecureShield-VPC
VPC CIDR             10.0.0.0/16
Public Subnet        Public-Web-Subnet
Public Subnet CIDR   10.0.1.0/24
Internet Gateway     SecureShield-IGW
Route Table          Public-RT
EC2 Instance         WEB-SERVER-01
Operating System     Amazon Linux
Web Server           Nginx
Security Group       WEB-SG
EC2 IAM Role         SecureShield-EC2-SecurityReadOnly
CloudTrail           SecureShield-Audit-Trail
Audit Storage        Private encrypted S3 bucket

🌐 Network Security Design

The web server is deployed inside a dedicated AWS VPC.

                         INTERNET
                            │
                            │ HTTP :80
                            ▼
                    Internet Gateway
                            │
                            ▼
                 SecureShield-VPC
                    10.0.0.0/16
                            │
                            ▼
                  Public-Web-Subnet
                    10.0.1.0/24
                            │
                            ▼
                         WEB-SG
                            │
                            ▼
                     WEB-SERVER-01
                  Amazon Linux + Nginx

The public subnet has a route to the Internet Gateway:

0.0.0.0/0 → SecureShield-IGW

This provides Internet connectivity while the Security Group controls
which inbound connections can reach the server.

🔥 Security Group Hardening

WEB-SG acts as the instance-level virtual firewall.

The secure baseline is:

Protocol     Port Source          Reason

HTTP           80 0.0.0.0/0     Public website
SSH            22 ADMIN-IP/32   Restricted administration

Why SSH is restricted

A rule such as:

SSH :22 → 0.0.0.0/0

would unnecessarily expose the SSH service to connection attempts from
the entire IPv4 Internet.

Instead:

Administrator IP
       │
       │ SSH :22
       ▼
     WEB-SG
       │
       ▼
WEB-SERVER-01

Only the trusted administrator source address is permitted.

🖥️ EC2 Web Server

The public workload runs on:

WEB-SERVER-01
Amazon Linux
Nginx

Nginx was installed and managed from the Linux command line.

Example:

sudo dnf install nginx -y
sudo systemctl enable nginx
sudo systemctl start nginx
sudo systemctl status nginx

The server was tested locally using:

curl http://localhost

This combines basic AWS administration, Linux administration,
networking, and service troubleshooting in one workload.

Full command reference:

➡️ commands/linux-commands.md

🔐 IAM & Credential Security

One of the primary security goals was avoiding static AWS credentials on
the EC2 instance.

The insecure approach would be:

EC2
 │
 ├── AWS_ACCESS_KEY_ID
 └── AWS_SECRET_ACCESS_KEY

Instead, the instance uses an IAM role:

WEB-SERVER-01
       │
       ▼
SecureShield-EC2-SecurityReadOnly
       │
       ▼
AWS STS Temporary Credentials
       │
       ▼
AWS APIs

The active identity can be verified with:

aws sts get-caller-identity

This allows the EC2 workload to obtain temporary credentials without
manually storing long-lived AWS access keys on the server.

🔑 Least-Privilege IAM Challenge

During the lab, an AWS CLI investigation command was executed from the
EC2 instance:

aws cloudtrail lookup-events \
  --region eu-west-1 \
  --max-results 10

AWS returned:

AccessDeniedException

The easy but insecure solution would have been to attach:

AdministratorAccess

Instead, the missing API permission was identified:

cloudtrail:LookupEvents

A narrowly scoped inline policy was added:

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCloudTrailInvestigation",
      "Effect": "Allow",
      "Action": [
        "cloudtrail:LookupEvents"
      ],
      "Resource": "*"
    }
  ]
}

The command was then tested again.

Security lesson

API Request
    │
    ▼
ACCESS DENIED
    │
    ▼
Identify Required Permission
    │
    ▼
Grant Minimum Permission
    │
    ▼
Retry
    │
    ▼
SUCCESS

This demonstrates least privilege rather than solving authorization
problems with excessive permissions.

Policy file:

➡️
iam/cloudtrail-lookup-policy.json

👁️ AWS Audit Logging

AWS CloudTrail provides the audit trail for AWS management activity.

The project uses:

SecureShield-Audit-Trail

The investigation focuses on answering:

WHO performed the action?

WHAT action occurred?

WHEN did it happen?

WHERE did it originate?

WHICH AWS resource was affected?

Useful CloudTrail fields include:

eventName
eventTime
userIdentity
sourceIPAddress
eventSource
requestParameters

CloudTrail logs are delivered to a dedicated S3 security-log bucket.

🗄️ Audit Log Protection

The CloudTrail S3 bucket is designed as security-log storage rather than
public application storage.

Controls used include:

Block Public Access     ENABLED
Server-side encryption  ENABLED
Versioning              ENABLED
Public access            DISABLED

The actual bucket name and sensitive identifiers should be redacted from
public screenshots and documentation.

🚨 Incident Scenario --- AWS-IR-001

Accidental Public Service Exposure

A controlled incident was introduced to simulate a common cloud
configuration mistake.

An administrator accidentally creates:

Protocol: TCP
Port:     8080
Source:   0.0.0.0/0

The environment changes from:

SECURE BASELINE

HTTP :80  → Internet
SSH  :22  → Trusted Admin Only

to:

MISCONFIGURED

HTTP :80   → Internet
TCP  :8080 → Internet   ⚠
SSH  :22   → Trusted Admin Only

This creates unnecessary public network exposure.

The test was intentionally limited to a lab Security Group and did
not involve exploiting a real external system.

🔎 Incident Investigation

The Security Group modification was investigated through CloudTrail.

Example:

aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=EventName,AttributeValue=AuthorizeSecurityGroupIngress \
  --region eu-west-1 \
  --max-results 10

The investigation looked for:

Event Name
Event Time
Actor / IAM Identity
Source IP
AWS Service
Affected Resource
Request Parameters

This creates the following investigation workflow:

Security Group Change
        │
        ▼
   CloudTrail Event
        │
        ▼
   AWS CLI Search
        │
        ▼
 Identify Actor + Action
        │
        ▼
 Assess Exposure
        │
        ▼
     Remediate

🛠️ Incident Remediation

After collecting the required evidence, the temporary public TCP/8080
rule was removed.

The Security Group returned to:

HTTP :80
Source: 0.0.0.0/0

SSH :22
Source: ADMIN-IP/32

The incident was marked:

AWS-IR-001
Status: RESOLVED

Full incident report:

➡️ incident-response/AWS-IR-001.md

🔄 Incident Response Lifecycle Demonstrated

┌───────────────────────┐
│    Secure Baseline    │
└───────────┬───────────┘
            │
            ▼
┌───────────────────────┐
│   Misconfiguration    │
│ Public TCP/8080 Rule  │
└───────────┬───────────┘
            │
            ▼
┌───────────────────────┐
│    Audit Evidence     │
│      CloudTrail       │
└───────────┬───────────┘
            │
            ▼
┌───────────────────────┐
│     Investigation     │
│ AWS CLI + Event Data  │
└───────────┬───────────┘
            │
            ▼
┌───────────────────────┐
│      Remediation      │
│ Remove Public :8080   │
└───────────┬───────────┘
            │
            ▼
┌───────────────────────┐
│      Verification     │
│ Secure Baseline Back  │
└───────────────────────┘

🧪 Security Controls Demonstrated

Security Area             Implementation

Network isolation         Dedicated VPC
Network segmentation      Dedicated public subnet
Firewall control          EC2 Security Group
Administrative security   SSH restricted to /32
Credential management     EC2 IAM role
Authentication            Temporary AWS credentials
Authorization             Least-privilege IAM
Auditability              AWS CloudTrail
Log retention             S3
Log protection            Encryption + Block Public Access
Investigation             CloudTrail + AWS CLI
Incident response         Controlled SG exposure/remediation

Detailed controls:

➡️ security/security-controls.md

🧰 Technologies Used

Amazon Web Services (AWS)
├── Amazon VPC
├── Amazon EC2
├── IAM
├── AWS STS
├── AWS CloudTrail
├── Amazon S3
├── Security Groups
└── AWS CLI

Operating System
└── Amazon Linux

Web Server
└── Nginx

📂 Repository Structure

aws-secure-web-incident-response/
│
├── README.md
│
├── architecture/
│   └── architecture.png
│
├── iam/
│   └── cloudtrail-lookup-policy.json
│
├── incident-response/
│   └── AWS-IR-001.md
│
├── commands/
│   ├── aws-cli.md
│   └── linux-commands.md
│
└── security/
    └── security-controls.md

💻 AWS CLI Investigation Commands

Some of the commands demonstrated in the lab include:

aws sts get-caller-identity
aws ec2 describe-instances
aws ec2 describe-security-groups
aws cloudtrail lookup-events
aws iam list-attached-role-policies
aws s3api get-public-access-block
aws s3api get-bucket-encryption
aws s3api get-bucket-versioning

Full reference:

➡️ commands/aws-cli.md

📚 Key Lessons Learned

1. Public does not mean unrestricted

An Internet-facing EC2 instance can still have tightly controlled
administrative access.

2. IAM roles are preferable to static workload credentials

EC2 workloads can obtain temporary AWS credentials without storing
access keys on disk.

3. AccessDenied is useful security feedback

An authorization failure can reveal where additional permissions are
required. The correct response is to add only the necessary permission
rather than granting broad administrative access.

4. Audit logs matter

Without CloudTrail, determining who changed a cloud resource and when
becomes significantly harder.

5. Cloud incidents often begin with configuration changes

Not every security incident requires malware. A firewall or IAM
misconfiguration can itself create meaningful risk.

6. Detection is only part of incident response

A complete workflow includes:

Detection → Investigation → Containment/Remediation → Verification

🧠 Interview Talking Points

This project can be explained in an interview as:

I built a small AWS web environment inside a dedicated VPC and
deployed an Amazon Linux EC2 instance running Nginx. I restricted SSH
to a trusted administrator address while keeping HTTP available
publicly. Instead of storing AWS access keys on the instance, I
attached an IAM role so the workload used temporary credentials. I
configured CloudTrail for AWS API auditing and protected its logs in
S3. I then simulated an accidental public Security Group exposure on
TCP/8080, investigated the change through CloudTrail using the AWS
CLI, identified the relevant activity, removed the insecure rule, and
verified the environment returned to its secure baseline. During the
investigation I also encountered an IAM AccessDenied error and
resolved it by adding only the required CloudTrail lookup permission
rather than using AdministratorAccess.

🚧 Current Scope

This repository documents only controls that were actually implemented
or tested.

GuardDuty is not claimed as implemented.

GuardDuty was evaluated during the lab, but the AWS account returned a
subscription requirement. It was therefore excluded rather than enabling
an additional service solely for the portfolio.

This distinction is intentional: portfolio documentation should
accurately represent what was built.

🚀 Future Improvements

Version 2 --- Multi-Tier Architecture

Internet
   │
   ▼
Public Web Tier
   │
   ▼
Private Application Tier

Version 3 --- Automated Detection

CloudTrail
   │
   ▼
CloudWatch
   │
   ▼
Detection / Alarm
   │
   ▼
SNS Notification

Version 4 --- Infrastructure as Code

Rebuild the environment using:

Terraform
AWS Provider
Reusable Variables
Security-focused Modules

🔒 Repository Security

Before publishing evidence, redact:

AWS Account IDs
EC2 Instance IDs where appropriate
Personal Public IP Addresses
Sensitive CloudTrail Fields
Unique S3 Bucket Names where appropriate

Never upload:

*.pem
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_SESSION_TOKEN
Passwords
Private Keys
.env files containing secrets

👤 Author

Ahilya Sarnaik

MSc Cyber Security
Cloud Security • SOC • Network Security • Incident Response

⭐ Project Status

AWS Infrastructure       ✅
EC2 Web Server           ✅
Security Group Hardening ✅
IAM Instance Role        ✅
Least-Privilege Policy   ✅
CloudTrail Auditing      ✅
S3 Audit Storage         ✅
Incident Simulation      ✅
Incident Investigation   ✅
Incident Remediation     ✅
GuardDuty                Not Implemented

Project 1: Complete --- ready for portfolio documentation and further
enhancement.

📄 License

This project is licensed under the MIT License.

You are free to use, modify, and distribute the project in accordance
with the terms of the license.

See LICENSE for the full license text.
