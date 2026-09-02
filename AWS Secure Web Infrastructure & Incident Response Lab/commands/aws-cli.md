Yes. Rather than trying to build a huge “medium enterprise” environment immediately, make Project 1 focused, complete, demonstrable, and interview-ready.

Project 1 — AWS Secure Web Server & Cloud Incident Response Lab

Difficulty: Beginner → Intermediate
Target roles: Cloud Support Engineer, Junior Cloud Security Engineer, SOC Analyst, Security Analyst, Systems/Infrastructure Support
Expected build time: ~3–5 hours
Goal: Deploy an internet-facing Linux web server securely, implement IAM least privilege and AWS audit logging, deliberately create a safe security misconfiguration, investigate it through CloudTrail, and remediate it.

This is much better for your first project because you can actually explain every component in an interview.

Final architecture
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
What you're proving

The project answers six useful interview questions:

Question	Your implementation
Can you deploy AWS infrastructure?	VPC, subnet, IGW, route table, EC2
Can you secure a Linux server?	Security groups + restricted SSH
Do you understand IAM?	EC2 role + least privilege
Do you understand credentials?	Temporary role credentials instead of hardcoded keys
Can you audit AWS activity?	CloudTrail
Can you investigate incidents?	Controlled SG change + CloudTrail investigation
Part 1 — Network

You already built most of this.

VPC
Name: SecureShield-VPC
CIDR: 10.0.0.0/16
Region: eu-west-1
Public subnet
Name: Public-Web-Subnet
CIDR: 10.0.1.0/24
Internet Gateway
SecureShield-IGW
Route table
Public-RT

10.0.0.0/16 → local
0.0.0.0/0   → SecureShield-IGW

Associated with:

Public-Web-Subnet

That demonstrates basic AWS networking.

Part 2 — Secure EC2 Web Server

Your server:

WEB-SERVER-01

OS:
Amazon Linux

Web Server:
Nginx

Public IP:
Yes

Security group:

WEB-SG

Inbound:

HTTP
TCP 80
0.0.0.0/0

SSH
TCP 22
YOUR-IP/32

The important security decision is:

Internet ───── HTTP :80 ─────► Web Server

Internet ───── SSH :22 ──X

Your IP ────── SSH :22 ─────► Web Server

Don't expose SSH publicly.

Part 3 — Linux Administration

You installed Nginx:

sudo dnf install nginx -y

sudo systemctl enable nginx

sudo systemctl start nginx

sudo systemctl status nginx

And created a website.

Test:

curl http://localhost

Then:

http://EC2-PUBLIC-IP

This demonstrates basic Linux administration as well as AWS.

Part 4 — IAM Security

This is one of the strongest parts of your current project.

You created an EC2 IAM role:

SecureShield-EC2-SecurityReadOnly

Then attached it to:

WEB-SERVER-01

Instead of doing:

aws configure

and storing:

AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY

on EC2, you used:

EC2
 │
 ▼
IAM Role
 │
 ▼
STS Temporary Credentials
 │
 ▼
AWS API

You verified it using:

aws sts get-caller-identity

and received an:

assumed-role/SecureShield-EC2-SecurityReadOnly/...

That's a very useful screenshot for GitHub.

Interview explanation

If asked:

Why did you use an IAM role instead of access keys?

Answer:

I used an EC2 IAM role so the workload could obtain temporary credentials through AWS rather than storing long-lived access keys on the server. This reduces credential exposure and makes permissions easier to control and rotate.

Part 5 — Least-Privilege Experiment

This is probably the best part of what you've done so far.

You attempted:

aws cloudtrail lookup-events

AWS responded:

AccessDeniedException

because the role didn't have:

cloudtrail:LookupEvents

You did not fix this by granting:

AdministratorAccess

Instead you created a narrowly scoped policy:

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "cloudtrail:LookupEvents"
      ],
      "Resource": "*"
    }
  ]
}

That's a genuine example of applying least privilege.

Your story is:

AWS API request
       │
       ▼
ACCESS DENIED
       │
       ▼
Investigate IAM permissions
       │
       ▼
Required permission identified
       │
       ▼
cloudtrail:LookupEvents
       │
       ▼
Minimal permission added
       │
       ▼
Request succeeds

That's much more interesting to discuss than simply saying, “I created an IAM user.”

Part 6 — CloudTrail

Your trail:

SecureShield-Audit-Trail

Configuration:

Multi-region                    YES
Management Events               YES
Read Events                     YES
Write Events                    YES
Log File Validation             YES
SSE-KMS                         NO
CloudWatch integration          optional

CloudTrail provides the audit evidence for AWS administrative/API actions.

You learned to investigate:

WHO?
WHAT?
WHEN?
FROM WHERE?
WHICH RESOURCE?

using fields such as:

userIdentity
eventName
eventTime
sourceIPAddress
eventSource
requestParameters
Part 7 — S3 Audit Storage

If you created the bucket during the CloudTrail setup, include it as part of Project 1.

For example:

secureshield-security-logs-xxxx

Security:

Block Public Access     ON
Encryption              ON
Versioning              ON
ACLs                     Disabled

Purpose:

AWS Activity
     ↓
CloudTrail
     ↓
S3 Security Logs

Don't publish the actual bucket name if it contains identifying information. Use a sanitized version in documentation.

Part 8 — Incident Response Simulation

This makes Project 1 stand out.

You temporarily created:

Custom TCP
Port: 8080
Source: 0.0.0.0/0

representing an administrator accidentally exposing a service publicly.

Incident
AWS-IR-001

Public Network Exposure
Detection

CloudTrail.

You queried:

aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=EventName,AttributeValue=AuthorizeSecurityGroupIngress \
  --region eu-west-1 \
  --max-results 10

and/or:

aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=EventName,AttributeValue=ModifySecurityGroupRules \
  --region eu-west-1 \
  --max-results 10
Investigation

You examined:

EventName
EventTime
Username / Role
SourceIPAddress
Security Group
RequestParameters
Remediation

Removed:

TCP 8080
0.0.0.0/0

and restored the secure baseline.

That's an actual security lifecycle:

Secure Baseline
      │
      ▼
Misconfiguration
      │
      ▼
Audit Evidence
      │
      ▼
Investigation
      │
      ▼
Attribution
      │
      ▼
Remediation
      │
      ▼
Secure Baseline
Part 9 — GuardDuty

Do not claim GuardDuty as implemented in Project 1.

You tested:

aws guardduty list-detectors --region eu-west-1

and AWS returned:

SubscriptionRequiredException

Therefore your documentation should simply say:

GuardDuty integration was evaluated but excluded from the lab because the service was not active for the account and the project was designed around cost-controlled AWS usage.

That's perfectly acceptable.

Project 1 GitHub structure

When we document it, I recommend:

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
├── screenshots/
│   ├── 01-vpc.png
│   ├── 02-subnet.png
│   ├── 03-web-server.png
│   ├── 04-security-group.png
│   ├── 05-website.png
│   ├── 06-iam-role.png
│   ├── 07-access-denied.png
│   ├── 08-least-privilege-policy.png
│   ├── 09-cloudtrail-event.png
│   └── 10-remediation.png
│
└── security/
    └── security-controls.md

Never upload:

*.pem
AWS access keys
secret keys
account IDs
credentials
private IP information if unnecessary
personal public IP
Resume version

Once we finish documenting Project 1, I'd put it on your CV as:

AWS Secure Web Infrastructure & Incident Response Lab

AWS | EC2 | VPC | IAM | CloudTrail | S3 | Linux | Nginx | AWS CLI

Designed and secured an AWS web environment using VPC networking, EC2, restrictive security groups and Amazon Linux/Nginx. Implemented EC2 IAM roles with temporary credentials and least-privilege CloudTrail access rather than long-lived access keys. Centralized AWS management-event auditing through CloudTrail and S3 and conducted a controlled security-group misconfiguration exercise, investigating API activity, identity and source telemetry before remediating unauthorized public network exposure.

That's a project I would be comfortable putting on a junior candidate's CV once all of those components are verified.

Then build Project 2

Don't make Project 1 enormous.

Your progression should be:

Project 1 — AWS Secure Web Server + Incident Response ← we're here

Project 2 — AWS Multi-Tier Enterprise Network Security

Internet
   ↓
Public Web Tier
   ↓
Private Application Tier
   ↓
Private Database/Data Tier

Project 3 — AWS Cloud SOC & Automated Detection

CloudTrail
   ↓
CloudWatch
   ↓
Detection
   ↓
SNS
   ↓
Alert
   ↓
Incident Response

Project 4 — Terraform AWS Secure Infrastructure

Rebuild Projects 1/2 through Infrastructure as Code.

That progression gives your GitHub a much better story: fundamentals → enterprise architecture → detection engineering → automation, instead of one giant project you can't fully explain.

how to get cloud trail policy

For your project, you want a small custom CloudTrail policy rather than broad CloudTrail administrator access.

AWS confirms that IAM policies can grant specific CloudTrail actions, and LookupEvents is the API operation used to search recent CloudTrail event history.

Create it in the AWS Console

Go to:

IAM → Roles → SecureShield-EC2-SecurityReadOnly

Then:

Permissions → Add permissions → Create inline policy → JSON

Paste:

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

Click Next, and name it:

CloudTrail-Lookup-Only

Then Create policy.

Resource: "*" is appropriate here because LookupEvents doesn't use a specific trail ARN for this permission pattern. AWS's own read-only CloudTrail example also includes cloudtrail:LookupEvents with "Resource": "*".

Test it from your EC2 terminal

Run:

aws cloudtrail lookup-events \
  --region eu-west-1 \
  --max-results 5

You should get events rather than:

AccessDeniedException

Then test your security investigation:

aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=EventName,AttributeValue=AuthorizeSecurityGroupIngress \
  --region eu-west-1 \
  --max-results 10
For GitHub

Save this policy as:

iam/cloudtrail-lookup-policy.json

This is actually a good part of your project story: your EC2 role initially received AccessDenied, and instead of adding AdministratorAccess, you granted only the required cloudtrail:LookupEvents permission—demonstrating least privilege.

AWS also offers the broader managed AWSCloudTrail_ReadOnlyAccess policy, but for this lab I prefer your one-action custom policy because it better demonstrates that you understand IAM permissions rather than simply attaching a broad managed policy.

. ├── README.md ├── architecture/ │   └── architecture.png ├── iam/ │   └── cloudtrail-lookup-policy.json ├── incident-response/ │   └── AWS-IR-001.md ├── commands/ │   ├── aws-cli.md │   └── linux-commands.md └── security/     └── security-controls.md  
give me these files

Done. I created the complete Project 1 GitHub package based only on the AWS work you've actually implemented, without falsely claiming GuardDuty or other unfinished services.

Download

Download complete AWS project ZIP

Individual files:

README.md
architecture.png
cloudtrail-lookup-policy.json
AWS-IR-001.md
aws-cli.md
linux-commands.md
security-controls.md

Your repository structure is ready:

aws-secure-web-incident-response/
│
├── README.md
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

I also deliberately marked GuardDuty as evaluated but not implemented, because your account returned SubscriptionRequiredException. That keeps the portfolio technically accurate.

The next useful step is to create the GitHub repository and push this structure, then add sanitized screenshots of your actual AWS environment as evidence. 

README.md
Document
architecture.png
Image
cloudtrail-lookup-policy.json
Code
AWS-IR-001.md
Document
aws-cli.md
Document
linux-commands.md
Document
security-controls.md
Document
Show less

aws-cli.md
AWS CLI Commands

These commands were used to validate identity, inspect infrastructure,
and investigate AWS activity.

Replace placeholders with your own resource identifiers. Do not commit
account IDs, personal IPs, credentials, or private keys.

Identity

Confirm the IAM identity/role used by the EC2 instance:

aws sts get-caller-identity

Expected pattern:

arn:aws:sts::<REDACTED>:assumed-role/SecureShield-EC2-SecurityReadOnly/<INSTANCE>
EC2 Instances
aws ec2 describe-instances \
  --region eu-west-1 \
  --query 'Reservations[].Instances[].{Name:Tags[?Key==`Name`]|[0].Value,State:State.Name,PrivateIP:PrivateIpAddress}' \
  --output table
Security Groups
aws ec2 describe-security-groups \
  --region eu-west-1 \
  --query 'SecurityGroups[].{Name:GroupName,Id:GroupId,VpcId:VpcId}' \
  --output table

Inspect one Security Group:

aws ec2 describe-security-groups \
  --group-ids <SECURITY-GROUP-ID> \
  --region eu-west-1
CloudTrail Investigation

Recent events:

aws cloudtrail lookup-events \
  --region eu-west-1 \
  --max-results 10

Security Group ingress changes:

aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=EventName,AttributeValue=AuthorizeSecurityGroupIngress \
  --region eu-west-1 \
  --max-results 10

Alternative modification event:

aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=EventName,AttributeValue=ModifySecurityGroupRules \
  --region eu-west-1 \
  --max-results 10
IAM

List policies attached to the EC2 role:

aws iam list-attached-role-policies \
  --role-name SecureShield-EC2-SecurityReadOnly

List inline policy names:

aws iam list-role-policies \
  --role-name SecureShield-EC2-SecurityReadOnly
S3 Security

List buckets:

aws s3api list-buckets \
  --query 'Buckets[].Name' \
  --output table

Check public-access-block configuration:

aws s3api get-public-access-block \
  --bucket <SECURITY-LOG-BUCKET>

Check versioning:

aws s3api get-bucket-versioning \
  --bucket <SECURITY-LOG-BUCKET>

Check default encryption:

aws s3api get-bucket-encryption \
  --bucket <SECURITY-LOG-BUCKET>
CloudTrail Trails
aws cloudtrail describe-trails \
  --region eu-west-1

Check trail status:

aws cloudtrail get-trail-status \
  --name SecureShield-Audit-Trail \
  --region eu-west-1
