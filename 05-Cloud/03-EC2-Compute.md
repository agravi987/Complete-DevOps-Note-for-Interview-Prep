# AWS EC2 – Elastic Compute Cloud

## 1. Overview
**EC2 (Elastic Compute Cloud)** is AWS's core virtual server service. It allows you to rent virtual machines of any size — from a tiny `t2.micro` (1 vCPU, 1 GB RAM) to a massive `p4d.24xlarge` (96 vCPU, 1152 GB RAM with 8 GPUs for AI workloads). EC2 is the backbone of almost every cloud deployment.

## 2. Why This Matters
- **On-Demand Servers:** Launch a production-grade Ubuntu server in 60 seconds. No procurement process, no hardware delivery delays.
- **Elastic Scaling:** Traffic doubles on a product launch? Add 10 servers in minutes. Traffic drops? Terminate them and stop paying.
- **Full Control:** Unlike PaaS services (like Heroku), EC2 gives you root SSH access to configure the server exactly as needed — custom kernel settings, firewalls, custom software installations.

## 3. Core Concepts

- **AMI (Amazon Machine Image):** The template (snapshot) used to launch an EC2 instance. It contains the OS (Ubuntu 22.04, Amazon Linux 2, Windows Server), pre-installed software, and root volume configuration.
- **Instance Type:** Defines the virtual hardware (vCPU, RAM, Network speed). Categories:
  - `t` series → General purpose, burstable (dev/test)
  - `m` series → Balanced compute/memory (web servers)
  - `c` series → Compute optimized (CI/CD runners, encoding)
  - `r` series → Memory optimized (databases, caches)
- **Key Pair:** An SSH public/private key set. AWS stores the public key; you keep the private `.pem` file. Required to SSH into Linux instances.
- **Elastic IP (EIP):** A static public IP address that stays the same even if you stop and restart the EC2 instance.
- **User Data:** A bootstrap script that runs automatically the first time an instance boots. Used to install software, pull configs, execute setup commands.
- **Instance Store vs EBS:**
  - **Instance Store:** Physically attached temporary SSD. Fastest I/O, but all data is lost when the instance stops.
  - **EBS (Elastic Block Store):** Network-attached persistent disk. Survives instance stop/start. Like a hard drive you can unplug and reattach.

## 4. Architecture / Workflow

```
Developer  →  AWS Console / CLI / Terraform
                     │
                     ▼
              Choose AMI + Instance Type
                     │
                     ▼
              Place in VPC Subnet + Security Group
                     │
                     ▼
              Attach EBS Volume + Key Pair
                     │
                     ▼
              User Data script runs on first boot
                     │
                     ▼
              EC2 Instance → Running → SSH Access
```

## 5. Commands & Practical Usage

Launch an EC2 instance via CLI:
```bash
aws ec2 run-instances \
  --image-id ami-0f5ee92e2d63afc18 \  # Ubuntu 22.04 in ap-south-1
  --instance-type t2.micro \
  --key-name my-key-pair \
  --security-group-ids sg-0abc12345 \
  --subnet-id subnet-0abc12345 \
  --count 1 \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=WebServer}]'
```

List all running instances with their public IPs:
```bash
aws ec2 describe-instances \
  --query 'Reservations[*].Instances[*].[InstanceId,State.Name,PublicIpAddress,Tags[?Key==`Name`].Value|[0]]' \
  --output table
```

SSH into a Linux EC2 instance:
```bash
# Fix key permissions first (required!)
chmod 400 my-key.pem
ssh -i my-key.pem ubuntu@<PUBLIC-IP>
```

Stop and terminate an instance:
```bash
aws ec2 stop-instances --instance-ids i-0abc12345
aws ec2 terminate-instances --instance-ids i-0abc12345
```

## 6. Configuration / Code Examples

**User Data script** — Installs Docker automatically on first boot:
```bash
#!/bin/bash
# This runs as root on the very first boot
apt-get update -y
apt-get install -y docker.io
systemctl enable docker
systemctl start docker
usermod -aG docker ubuntu
echo "Bootstrap complete — Docker is ready!" >> /var/log/user-data.log
```

Attach this when launching via console: EC2 → Advanced Details → User Data field.

## 7. Hands-on Step-by-Step (VERY IMPORTANT)

**Step 1: Create a Key Pair**
1. Go to EC2 → Key Pairs → Create Key Pair.
2. Name: `my-devops-key`, Format: `.pem` (for Linux SSH). Download and save it safely.

**Step 2: Launch an EC2 Instance**
1. EC2 → Instances → Launch Instances.
2. Name: `MyFirstServer`.
3. AMI: `Ubuntu Server 22.04 LTS`.
4. Instance type: `t2.micro` (Free Tier eligible).
5. Key pair: `my-devops-key`.
6. Network: Select your VPC and public subnet.
7. Security Group: Allow SSH (22) from your IP.
8. Click **Launch Instance**.

**Step 3: SSH into Your Server**
```bash
chmod 400 ~/Downloads/my-devops-key.pem
ssh -i ~/Downloads/my-devops-key.pem ubuntu@<YOUR-EC2-PUBLIC-IP>
```

**Step 4: Install Nginx manually to verify it works**
```bash
sudo apt update && sudo apt install nginx -y
sudo systemctl start nginx
```

**Step 5: Open Port 80 in Security Group**
Go to your Security Group → Inbound Rules → Add Rule: HTTP, Port 80, Source: `0.0.0.0/0`.

**Step 6: Test in browser**
Open `http://<YOUR-EC2-PUBLIC-IP>`. You will see the Nginx welcome page.

## 8. Common Errors & Troubleshooting

- **Error: `Permission denied (publickey)` when trying to SSH**
  - **Issue 1:** Wrong username. Ubuntu uses `ubuntu`, Amazon Linux uses `ec2-user`, CentOS uses `centos`.
  - **Issue 2:** Wrong `.pem` file or wrong key pair selected at launch.
  - **Fix:** Verify the username for your AMI. Double-check that the key pair in the SSH command matches what was used at launch.

- **Error: `ssh: connect to host X port 22: Connection timed out`**
  - **Issue:** The Security Group does not allow inbound SSH on port 22, or the instance is in a private subnet with no public IP.
  - **Fix:** Edit the Security Group to allow TCP port 22 from your IP address.

- **Error: Instance keeps stopping unexpectedly**
  - **Issue:** The instance type is a `t` series and it ran out of CPU credits, or you selected a SPOT instance that was reclaimed by AWS.
  - **Fix:** Upgrade to a `m` or `c` series for consistent workloads, or switch from Spot to On-Demand pricing.

## 9. Best Practices

1. **Never open port 22 to `0.0.0.0/0`** in production. Restrict SSH to your office IP range only or use AWS Systems Manager Session Manager, which requires zero open ports.
2. **Use IAM Roles for EC2, not Access Keys.** Instead of copying your AWS Access Key onto a server, attach an IAM Role with the required permissions. If the server is compromised, you can revoke the Role — you cannot revoke a hardcoded key.
3. **Enable Termination Protection** for production instances so no one accidentally destroys a critical database server.
4. **Use Auto Scaling Groups** for any instance running in production. Never manage EC2 instances manually at scale.

## 10. Interview Questions & Answers

**Q1: What is the difference between stopping and terminating an EC2 instance?**
**A1:** Stopping an instance shuts it down but preserves the EBS root volume and its data — you can restart it later. Terminating permanently deletes the instance and (by default) deletes the attached EBS volume. Data is gone.

**Q2: What is an AMI and why would you create a custom one?**
**A2:** An AMI is a pre-configured OS template. You create a custom AMI when you want to bake in software (like Apache, your app, monitoring agents) so that new instances launched from it are immediately production-ready without needing a bootstrap script.

**Q3: Why should you use IAM Roles instead of Access Keys on EC2 instances?**
**A3:** Access Keys hardcoded on a server are a massive security risk — if the server is breached, the keys are stolen. IAM Roles are attached to the instance at the AWS infrastructure level. Credentials are rotated automatically by AWS and never touch the disk.

**Q4: What happens to data on an Instance Store volume when the EC2 instance is stopped?**
**A4:** All data is permanently lost. Instance Store is ephemeral. It only survives reboots, not stop/start cycles or instance failures. Always use EBS for any data that needs to persist.

**Q5: What pricing model should you choose for a production database server that runs 24/7?**
**A5:** Reserved Instances (1-year or 3-year commitment) for predictable 24/7 workloads — they offer up to 72% discount over On-Demand. Use On-Demand only for variable or unpredictable workloads. Never use Spot Instances for databases.

## 11. Quick Revision Summary
- **AMI:** OS template for launching instances.
- **Instance Type:** Virtual hardware (t2=dev, m=prod, c=compute, r=memory).
- **Key Pair:** SSH authentication. `.pem` = private key. Keep it secret.
- **EBS:** Persistent network-attached disk. Survives stop/start.
- **User Data:** Bootstrap script on first boot. Automates setup.

## 12. Official Documentation Links
- [Amazon EC2 User Guide](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html)
- [EC2 Instance Types](https://aws.amazon.com/ec2/instance-types/)
