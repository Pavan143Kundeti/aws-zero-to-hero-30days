# Day 7: Production-Grade VPC Project

## Project Overview

**What We'll Build**: Real production architecture used by companies

**Why Important**: 
- AWS recommended approach
- Secure application deployment
- Uses all concepts from Day 0-6
- Interview-ready project

---

## Architecture Diagram

```
Internet
    ↓
Internet Gateway
    ↓
┌─────────────────────────────────────────────────┐
│                    VPC                          │
│                                                 │
│  ┌──────────────────┐  ┌──────────────────┐   │
│  │  Public Subnet   │  │  Public Subnet   │   │
│  │   (AZ-1a)        │  │   (AZ-1b)        │   │
│  │                  │  │                  │   │
│  │  Load Balancer   │  │  NAT Gateway     │   │
│  │  Bastion Host    │  │                  │   │
│  └────────┬─────────┘  └──────────────────┘   │
│           │                                     │
│  ┌────────┴─────────┐  ┌──────────────────┐   │
│  │ Private Subnet   │  │ Private Subnet   │   │
│  │   (AZ-1a)        │  │   (AZ-1b)        │   │
│  │                  │  │                  │   │
│  │  App Server 1    │  │  App Server 2    │   │
│  └──────────────────┘  └──────────────────┘   │
└─────────────────────────────────────────────────┘
```

---

## Key Concepts Before Starting

### 1. Auto Scaling Group

**What**: Automatically manages number of EC2 instances

**Example**:
- Start with 2 servers
- Traffic increases → Scale to 4 servers
- Traffic decreases → Scale back to 2 servers

**Why**: Handle traffic spikes automatically

### 2. Load Balancer

**What**: Distributes traffic across multiple servers

**Example**:
- 100 requests coming
- Server 1 gets 50 requests
- Server 2 gets 50 requests

**Why**: Balance load, no single server overloaded

### 3. Bastion Host (Jump Server)

**What**: EC2 in public subnet to access private instances

**Why Needed**:
- Private instances have no public IP
- Can't SSH directly
- Bastion acts as gateway

**Benefits**:
- Proper logging
- Audit who accessed what
- Centralized access control

### 4. NAT Gateway

**What**: Allows private instances to access internet

**How**: Masks private IP with public IP

**Example**:
- Private instance IP: 172.16.3.4
- Wants to download from internet
- NAT Gateway changes IP to its public IP
- Internet sees NAT Gateway IP only
- Private IP stays hidden

---

## Step 1: Create VPC

### Process

1. Go to VPC service
2. Click "Create VPC"
3. Select **"VPC and more"** (not "VPC only")

**Why "VPC and more"?**
- AWS creates everything automatically
- Subnets, route tables, internet gateway
- Saves time and configuration

### Configuration

**Name**: aws-prod-example

**IP Range**: 10.0.0.0/16 (65,536 IPs)

**Availability Zones**: 2

**Public Subnets**: 2 (one per AZ)

**Private Subnets**: 2 (one per AZ)

**NAT Gateways**: 1 per AZ

**VPC Endpoints**: None (remove S3 endpoint)

### What AWS Creates

1. VPC with IP range
2. 2 Public subnets (us-east-1a, us-east-1b)
3. 2 Private subnets (us-east-1a, us-east-1b)
4. Internet Gateway
5. Route Tables (public and private)
6. NAT Gateways with Elastic IPs

**Note**: If elastic IP limit reached, release unused IPs first

---

## Step 2: Create Launch Template

**Why**: Template for Auto Scaling Group

### Configuration

1. Go to EC2 → Launch Templates
2. Click "Create launch template"

**Name**: aws-prod-example

**Description**: Proof of concept for app deploy in AWS private subnet

**AMI**: Ubuntu (recent)

**Instance Type**: t2.micro (free tier)

**Key Pair**: Select your existing key

**Network Settings**:
- Create new security group
- Name: aws-prod-example
- Allow SSH (port 22) from anywhere
- Allow Custom TCP (port 8000) from anywhere

**Why Port 8000?** Python app will run on this port

**Storage**: Default (no changes)

3. Click "Create launch template"

---

## Step 3: Create Auto Scaling Group

### Configuration

1. Go to EC2 → Auto Scaling Groups
2. Click "Create Auto Scaling group"

**Name**: aws-prod-example

**Launch Template**: aws-prod-example

**VPC**: aws-prod-example

**Subnets**: Select BOTH private subnets
- private-subnet-1a
- private-subnet-1b

**Load Balancer**: None (create separately)

**Group Size**:
- Desired: 2
- Minimum: 2
- Maximum: 4

**Scaling Policies**: None (for now)

3. Click "Create Auto Scaling group"

### Verification

1. Go to EC2 → Instances
2. Should see 2 instances running
3. Check availability zones:
   - Instance 1: us-east-1a
   - Instance 2: us-east-1b
4. Note: No public IP addresses (private subnet)

---

## Step 4: Create Bastion Host

**Why**: To access private instances

### Configuration

1. Go to EC2 → Launch Instance

**Name**: bastion-host

**AMI**: Ubuntu

**Instance Type**: t2.micro

**Key Pair**: Same as before

**Network Settings**:
- VPC: aws-prod-example
- Subnet: PUBLIC subnet (important!)
- Auto-assign Public IP: Enable
- Security Group: Allow SSH (port 22)

2. Click "Launch instance"

---

## Step 5: Copy Key to Bastion

**Why**: Bastion needs key to access private instances

### Commands

```bash
# On your local machine
# Copy .pem file to Bastion
scp -i aws-login.pem aws-login.pem ubuntu@<BASTION_PUBLIC_IP>:~/

# SSH to Bastion
ssh -i aws-login.pem ubuntu@<BASTION_PUBLIC_IP>

# Verify key copied
ls
# Should see: aws-login.pem
```

---

## Step 6: Access Private Instance

### From Bastion Host

```bash
# Get private IP of instance from AWS console
# Example: 10.0.140.109

# SSH to private instance
ssh -i aws-login.pem ubuntu@10.0.140.109

# You're now in private instance!
```

---

## Step 7: Deploy Application

### On Private Instance

```bash
# Update packages
sudo apt update

# Create HTML file
nano index.html
```

**HTML Content**:
```html
<!DOCTYPE html>
<html>
<body>
<h1>My First AWS Project</h1>
<p>To demonstrate apps in private subnet</p>
</body>
</html>
```

**Start Python Server**:
```bash
python3 -m http.server 8000
```

**Note**: Deploy on ONLY ONE instance (for demo purposes)

---

## Step 8: Create Target Group

**What**: Defines which instances Load Balancer sends traffic to

### Configuration

1. Go to EC2 → Target Groups
2. Click "Create target group"

**Target Type**: Instances

**Name**: aws-prod-example

**Protocol**: HTTP

**Port**: 8000 (where app runs)

**VPC**: aws-prod-example

**Health Check**: HTTP on /

3. Click "Next"
4. Select BOTH instances
5. Click "Include as pending below"
6. Click "Create target group"

---

## Step 9: Create Load Balancer

### Configuration

1. Go to EC2 → Load Balancers
2. Click "Create Load Balancer"
3. Select "Application Load Balancer"

**Name**: aws-prod-example

**Scheme**: Internet-facing

**IP Type**: IPv4

**VPC**: aws-prod-example

**Subnets**: Select BOTH public subnets
- public-subnet-1a
- public-subnet-1b

**Security Group**: 
- Remove default
- Select aws-prod-example security group

**Listeners**: HTTP on port 80

**Target Group**: aws-prod-example

4. Click "Create load balancer"

---

## Step 10: Fix Security Group

**Problem**: Load Balancer not accessible

**Why**: Security group doesn't allow port 80

### Fix

1. Go to Load Balancer → Security tab
2. Click on Security Group
3. Edit Inbound Rules
4. Add Rule:
   - Type: HTTP
   - Port: 80
   - Source: 0.0.0.0/0 (anywhere)
5. Save rules

---

## Step 11: Test Application

### Access Load Balancer

1. Go to Load Balancer
2. Copy DNS name
3. Paste in browser: `http://<LOAD_BALANCER_DNS>`

**Result**: Should see "My First AWS Project"!

---

## Understanding Health Checks

**What Happened**: Only one instance working, but no errors

**Why**: Target Group health checks

**How It Works**:
- Target Group monitors instance health
- Instance 1: Healthy (app running)
- Instance 2: Unhealthy (no app)
- Load Balancer sends traffic ONLY to healthy instance

**View Health**:
1. Go to Target Groups
2. Select aws-prod-example
3. Click "Targets" tab
4. See health status of each instance

---

## Your Assignment

### Task: Enable Load Balancing

**Current State**: Only Instance 1 has app

**Goal**: Both instances serve traffic

**Steps**:

1. SSH to Bastion Host
2. SSH to Instance 2 (private IP)
3. Create index.html:
```html
<!DOCTYPE html>
<html>
<body>
<h1>My Second AWS Project</h1>
<p>This is instance 2!</p>
</body>
</html>
```
4. Start Python server: `python3 -m http.server 8000`
5. Refresh Load Balancer URL multiple times
6. Should see alternating messages:
   - "My First AWS Project"
   - "My Second AWS Project"

**This proves load balancing is working!**

---

## Traffic Flow Summary

### User Request Path

```
User Browser
    ↓
Load Balancer DNS (http://aws-lb-xxx.amazonaws.com)
    ↓
Internet Gateway
    ↓
Public Subnet
    ↓
Load Balancer (port 80)
    ↓
Target Group (health check)
    ↓
Private Subnet
    ↓
EC2 Instance (port 8000)
    ↓
Python Application
    ↓
Response back to user
```

### Bastion Access Path

```
Your Laptop
    ↓
SSH to Bastion (public IP)
    ↓
Bastion Host (public subnet)
    ↓
SSH to Private Instance (private IP)
    ↓
Private EC2 Instance
    ↓
Deploy/Manage Application
```

---

## Key Takeaways

1. **VPC with 2 AZs** = High availability
2. **Public Subnets** = Load Balancer, NAT Gateway, Bastion
3. **Private Subnets** = Application servers (secure)
4. **Auto Scaling Group** = Manages EC2 instances
5. **Load Balancer** = Distributes traffic
6. **Target Group** = Defines backend instances
7. **Bastion Host** = Access private instances
8. **Health Checks** = Only send traffic to healthy instances

---

## Common Issues & Solutions

### Issue 1: Can't Access Load Balancer

**Solution**: Add HTTP (port 80) to security group

### Issue 2: Elastic IP Limit Reached

**Solution**: Release unused Elastic IPs

### Issue 3: Can't SSH to Private Instance

**Solution**: 
- Verify key copied to Bastion
- Check private IP address
- Ensure security group allows SSH

### Issue 4: Application Not Loading

**Solution**:
- Check app running: `python3 -m http.server 8000`
- Verify port 8000 in security group
- Check target group health status

---

## Production Best Practices

1. **Use 2+ Availability Zones** (we did this)
2. **Private subnets for apps** (we did this)
3. **Bastion for access** (we did this)
4. **Load Balancer in public subnet** (we did this)
5. **Auto Scaling for resilience** (we did this)
6. **Health checks enabled** (automatic)
7. **NAT Gateway for updates** (we did this)

---

## Cost Considerations

**What Costs Money**:
- NAT Gateway (~$0.045/hour + data transfer)
- Load Balancer (~$0.025/hour)
- EC2 instances (free tier: 750 hours/month)
- Elastic IPs (free if attached)

**Tip**: Stop/delete resources after practice!

---

## Interview Questions

**Q: Explain this architecture**
**A**: VPC with public/private subnets across 2 AZs. Load Balancer in public subnet distributes traffic to app servers in private subnets. Auto Scaling manages instances. Bastion host for admin access. NAT Gateway for outbound internet.

**Q: Why 2 availability zones?**
**A**: High availability. If one AZ fails, other continues serving traffic.

**Q: Why private subnet for apps?**
**A**: Security. No direct internet access. Only accessible through Load Balancer.

**Q: What is Bastion host?**
**A**: Jump server in public subnet to access private instances securely.

---

## Cleanup Steps

**To avoid charges**:

1. Delete Load Balancer
2. Delete Target Group
3. Delete Auto Scaling Group
4. Terminate Bastion Host
5. Delete NAT Gateways
6. Release Elastic IPs
7. Delete VPC (deletes everything else)

---

## What's Next

**Day 8**: More AWS services and advanced concepts

**Skills Gained**:
- Production VPC setup
- Load Balancer configuration
- Auto Scaling Groups
- Bastion host usage
- Security best practices

---

**Congratulations! You've built a production-grade AWS architecture!** 🎉

*This is the same setup used by real companies in production!*