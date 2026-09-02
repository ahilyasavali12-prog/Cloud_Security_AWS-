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
