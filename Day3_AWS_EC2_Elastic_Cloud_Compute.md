# Day 3: AWS EC2 - Elastic Cloud Compute

## What You'll Learn Today
1. What is EC2 and why use it
2. Different types of EC2 instances
3. Regions and Availability Zones
4. Hands-on: Create EC2 instance
5. Deploy your first application (Jenkins)

---

## What is EC2?

### EC2 = Elastic Cloud Compute

**Breaking down the name:**

1. **Compute**: Virtual server with CPU + RAM + Disk
2. **Cloud**: Hosted on AWS public cloud platform
3. **Elastic**: Can scale up or down as needed

**Simple Definition**: EC2 is a service where you request AWS to give you a virtual machine that can be scaled easily.

### Virtual Server vs Physical Server

**Physical Server**:
- Your laptop (only you use it)
- Company buys from IBM/HP
- One person/application per server
- Expensive and wasteful

**Virtual Server**:
- Install hypervisor on physical server
- Create multiple virtual machines
- Share one physical server among many users
- Cost-effective and efficient

### How AWS EC2 Works

1. AWS has physical servers worldwide
2. You request an EC2 instance in specific region
3. AWS uses virtualization to create virtual machine
4. You get isolated virtual server
5. Multiple users share same physical hardware (securely)

---

## Why Use EC2?

### Problem with Managing Your Own Servers

If you create 1000 virtual machines yourself:
- Daily upgrades needed
- Security patches required
- Monitor if servers are up/down
- Fix issues 24/7
- Dedicated team needed
- Full-time job just for maintenance

### AWS EC2 advantages

**1. No Maintenance Burden**
- AWS handles all upgrades
- AWS manages security patches
- AWS monitors server health
- You focus on your application

**2. Cost Effective**
- Pay-as-you-go model
- Shut down when not needed (nights, holidays)
- No payment when server is off
- AWS buys at scale (cheaper for you)

**3. Elastic Nature**
- Increase/decrease resources anytime
- Add more RAM, CPU, storage
- Scale based on demand

---

## Types of EC2 Instances

### 1. General Purpose
- **Use**: Balanced compute, memory, networking
- **Best for**: Web servers, small databases, development
- **Example**: T2, T3 families
- **What we'll use**: Throughout this course

### 2. Compute Optimized
- **Use**: High compute power
- **Best for**: Gaming servers, machine learning, scientific modeling
- **Example**: C5, C6 families
- **Higher**: CPU ratio compared to memory

### 3. Memory Optimized
- **Use**: High memory operations
- **Best for**: Big data analytics, real-time processing, in-memory databases
- **Example**: R5, X1 families
- **Higher**: Memory ratio compared to CPU

### 4. Storage Optimized
- **Use**: High disk I/O operations
- **Best for**: Data warehouses, log processing, distributed file systems
- **Example**: I3, D2 families

### 5. Accelerated Computing
- **Use**: GPU-based workloads
- **Best for**: Graphics rendering, cryptocurrency mining, AI/ML training
- **Example**: P3, G4 families

### Choosing Instance Type

**Interview Question**: "Which EC2 instance type would you choose?"

**Answer**: "It depends on the application:
- Machine learning → Compute Optimized
- Big data analytics → Memory Optimized
- General web app → General Purpose
- Heavy storage needs → Storage Optimized"

---

## Regions and Availability Zones

### Regions

**What**: Geographic locations where AWS has data centers

**Examples**:
- North Virginia (US East)
- Mumbai (Asia Pacific)
- Frankfurt (Europe)
- Sydney (Australia)
- Ohio, California, Oregon, Singapore, etc.

### Why Regions Matter

**1. Security/Compliance**
- European banks want data in Europe only
- Government regulations
- Data sovereignty requirements

**2. Latency**
- Customer in US → Create instance in US
- Customer in India → Create instance in Mumbai
- Closer = Faster response time

**3. Cost**
- Different regions have different pricing
- Choose based on budget and requirements

### Availability Zones (AZ)

**What**: Multiple data centers within a region

**Example**: Mumbai region has multiple AZs
- ap-south-1a
- ap-south-1b
- ap-south-1c

### Why Availability Zones Matter

**Problem**: What if one data center goes down?
- Power failure
- Natural disaster
- Network issues

**Solution**: Deploy in multiple AZs
- If one AZ fails, others still work
- High availability for your application
- No downtime for customers

---

## AWS Free Tier Limits

### EC2 Free Tier
- **750 hours per month** of t2.micro instance
- **1 CPU + 1GB RAM**
- **Valid for 12 months** from account creation

### Understanding 750 Hours
- 24 hours × 31 days = 744 hours
- **One instance running 24/7 = FREE**
- **Two instances = You'll be charged**

### Managing Free Tier
- Run 1 instance all month = Free
- Run 3 instances for 250 hours each = Free
- Always stop instances when not using
- Monitor usage in billing dashboard

### Important Notes
- Only t2.micro is free tier eligible
- 8GB storage included
- Exceeding limits = Charges apply
- Set up billing alerts!

---

## Hands-On: Create EC2 Instance

### Step 1: Go to EC2 Service
1. Login to AWS Console
2. Search "EC2"
3. Click "Virtual servers in the cloud"
4. Click "Instances" → "Launch Instance"

### Step 2: Name Your Instance
- Enter name: "my-first-instance"

### Step 3: Choose Operating System
- Select: **Ubuntu**
- Choose: **Free tier eligible** version
- Other options: Amazon Linux, Red Hat, Windows

### Step 4: Choose Instance Type
- Select: **t2.micro** (Free tier eligible)
- 1 CPU, 1GB RAM
- Don't select larger instances (you'll be charged)

### Step 5: Key Pair (Important!)
- Click "Create new key pair"
- Name: "aws-login"
- Type: RSA
- Format: .pem
- Click "Create key pair"
- **File downloads**: aws-login.pem
- **Keep this file safe!** (Don't share with anyone)

**What is Key Pair?**
- Public key: Stored on EC2 instance
- Private key: You keep (.pem file)
- Used to login to instance securely
- Password authentication disabled by default

### Step 6: Network Settings
- **Don't change anything** (for now)
- We'll learn networking in future lessons
- Keep default settings

### Step 7: Storage
- Default: 8GB
- Free tier: Up to 30GB
- Keep default for now

### Step 8: Launch Instance
- Click "Launch Instance"
- Wait 1-2 minutes
- Instance state: Running

---

## Connecting to EC2 Instance

### Get Instance Details
1. Click on Instance ID
2. Copy **Public IP address**
3. Note: Private IP is for internal AWS network

### For Windows Users

**Option 1: PuTTY**
1. Download PuTTY
2. Convert .pem to .ppk using PuTTYgen
3. Use PuTTY to connect

**Option 2: MobaXterm**
1. Download MobaXterm (Free version)
2. Better than PuTTY (can save sessions)
3. Direct .pem file support

### For Mac/Linux Users

**Use Terminal**:

```bash
# Go to downloads folder
cd ~/Downloads

# Change permissions of key file
chmod 600 aws-login.pem

# Connect to instance
ssh -i aws-login.pem ubuntu@<PUBLIC_IP>

# Type 'yes' when prompted
```

**Why chmod 600?**
- .pem file has sensitive private key
- Must have restricted permissions
- 600 = Only you can read/write

### Default Usernames by OS
- Ubuntu → ubuntu
- Amazon Linux → ec2-user
- Red Hat → ec2-user
- Debian → admin

---

## Deploy Your First Application

### Step 1: Update Packages

```bash
# Switch to root user (optional)
sudo su -

# Update package list
apt update
```

### Step 2: Install Java

```bash
# Install Java 11
apt install openjdk-11-jdk -y

# Verify installation
java --version
```

### Step 3: Install Jenkins

```bash
# Add Jenkins repository key
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | apt-key add -

# Add Jenkins repository
sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'

# Update package list
apt update

# Install Jenkins
apt install jenkins -y
```

### Step 4: Start Jenkins

```bash
# Check Jenkins status
systemctl status jenkins

# If not running, start it
systemctl start jenkins

# Enable on boot
systemctl enable jenkins
```

### Step 5: Open Port 8080

**Problem**: Jenkins runs on port 8080, but it's blocked by default

**Solution**: Modify Security Group

1. Go to EC2 Console
2. Click on your instance
3. Click "Security" tab
4. Click on Security Group link
5. Click "Edit inbound rules"
6. Click "Add rule"
7. **Type**: Custom TCP
8. **Port**: 8080
9. **Source**: Anywhere IPv4 (0.0.0.0/0)
10. Click "Save rules"

### Step 6: Access Jenkins

1. Open browser
2. Go to: `http://<PUBLIC_IP>:8080`
3. You'll see Jenkins unlock screen

### Step 7: Get Initial Password

```bash
# Get Jenkins initial password
cat /var/lib/jenkins/secrets/initialAdminPassword
```

4. Copy password
5. Paste in browser
6. Click "Continue"

**Congratulations!** Your first application is deployed on AWS! 🎉

---

## Understanding Security Groups

### What Happened?
- Jenkins installed and running
- But not accessible from browser
- Why? **Security Groups**

### Security Groups = Firewall

**Inbound Rules**: Traffic coming TO your instance
**Outbound Rules**: Traffic going FROM your instance

### Default Behavior
- All inbound traffic: **BLOCKED**
- All outbound traffic: **ALLOWED**

### Opening Port 8080
- Added inbound rule for port 8080
- Now external traffic can reach Jenkins
- Security group acts as virtual firewall

**Note**: We'll learn security groups in detail in future lessons

---

## Important Commands Summary

```bash
# Change key permissions
chmod 600 aws-login.pem

# Connect to EC2
ssh -i aws-login.pem ubuntu@<PUBLIC_IP>

# Switch to root
sudo su -

# Update packages
apt update

# Install Java
apt install openjdk-11-jdk -y

# Check Java version
java --version

# Check Jenkins status
systemctl status jenkins

# Start Jenkins
systemctl start jenkins

# Get Jenkins password
cat /var/lib/jenkins/secrets/initialAdminPassword
```

---

## Best Practices

### 1. Always Stop Instances When Not Using
- Saves free tier hours
- Reduces costs
- Easy to start again when needed

### 2. Keep Key Pairs Safe
- Never share .pem files
- Don't commit to Git
- Store in secure location
- Use chmod 600 permissions

### 3. Use Appropriate Instance Types
- Don't use large instances for learning
- Stick to t2.micro for practice
- Upgrade only when needed

### 4. Monitor Free Tier Usage
- Check billing dashboard regularly
- Set up billing alerts
- Stop unused instances

### 5. Security Groups
- Only open required ports
- Don't open all ports to 0.0.0.0/0
- Review rules regularly

---

## Common Issues and Solutions

### Issue 1: Can't Connect via SSH
**Error**: "Permission denied" or "Connection refused"

**Solutions**:
- Check key file permissions: `chmod 600 aws-login.pem`
- Verify correct username (ubuntu for Ubuntu)
- Check public IP is correct
- Ensure instance is running

### Issue 2: Can't Access Application in Browser
**Solutions**:
- Check security group inbound rules
- Verify application is running: `systemctl status jenkins`
- Confirm correct port number
- Check public IP is correct

### Issue 3: Instance Not Starting
**Solutions**:
- Check free tier limits
- Verify region has capacity
- Try different availability zone

---

## Key Takeaways

1. **EC2 = Virtual servers on AWS cloud**
2. **Elastic = Can scale up/down easily**
3. **Regions = Geographic locations of data centers**
4. **Availability Zones = Multiple data centers per region**
5. **Free tier = 750 hours of t2.micro per month**
6. **Key pairs = Secure way to login (keep private!)**
7. **Security groups = Firewall for your instance**
8. **Always stop instances when not using**

---

## Your Assignment

### Task 1: Create EC2 Instance
1. Launch t2.micro Ubuntu instance
2. Create and download key pair
3. Wait for instance to be running

### Task 2: Connect to Instance
1. Use SSH to connect
2. Update packages
3. Explore the system

### Task 3: Deploy Application
1. Install Java
2. Install Jenkins
3. Configure security group
4. Access Jenkins in browser

### Task 4: Clean Up
1. Stop your instance (don't terminate yet)
2. Verify it's stopped
3. Check billing dashboard

---

## Next Steps
- Learn about EBS volumes (storage)
- Understand VPC and networking
- Explore load balancers
- Study auto-scaling

## Important Reminder
**Stop your EC2 instance after practice to save free tier hours!**

---

*Congratulations on deploying your first application on AWS!* 🚀