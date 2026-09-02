Linux Commands

Commands used on WEB-SERVER-01.

Identity and Host Information

whoami
hostname
cat /etc/os-release

Network Test

curl -I https://aws.amazon.com

Install Nginx

sudo dnf install nginx -y

Enable Nginx at boot:

sudo systemctl enable nginx

Start Nginx:

sudo systemctl start nginx

Check status:

sudo systemctl status nginx

Test Web Server

curl http://localhost

Create Lab Web Page

sudo tee /usr/share/nginx/html/index.html > /dev/null <<'EOF'
<!DOCTYPE html>
<html>
<head>
  <title>SecureShield Enterprise</title>
</head>
<body>
  <h1>SecureShield Enterprise</h1>
  <h2>AWS Enterprise Security Lab</h2>
  <p>Public Web Tier: Operational</p>
  <p>Region: eu-west-1</p>
  <p>Security: Segmented VPC + Restricted Security Groups</p>
</body>
</html>
EOF

Useful Troubleshooting

Check listening ports:

sudo ss -tulpn

Check Nginx:

sudo systemctl status nginx

View recent Nginx logs:

sudo journalctl -u nginx --no-pager -n 50

Test local HTTP:

curl -I http://localhost

AWS Role Validation

When AWS CLI is installed on the instance:

aws sts get-caller-identity

The server should use an IAM role with temporary credentials. Do not run
aws configure with long-lived access keys on the EC2 workload.
