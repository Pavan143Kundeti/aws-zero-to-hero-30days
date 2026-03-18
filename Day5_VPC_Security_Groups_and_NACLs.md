# Day 5: VPC Security - Security Groups and NACLs

## Quick Recap - Day 4
- Understood VPC concept
- Learned VPC components
- Used gated community analogy
- Studied traffic flow from user to application

**Today**: Deep dive into VPC security with Security Groups and NACLs (Theory + Practical)

---

## Why This Topic is Critical

### VPC = Private Cloud in Public Cloud

**Key Point**: VPC introduces private cloud security concepts into AWS public cloud.

**Why Important**:
- VPC is foundational for AWS
- Every DevOps engineer must understand VPC
- Security is the top priority for organizations
- Without VPC, AWS wouldn't be as secure

---

## AWS Security Philosophy

### Shared Responsibility Model

**AWS Says**: "Security in AWS is a SHARED RESPONSIBILITY"

**What This Means**:

**AWS's Part**:
- Provides VPC infrastructure
- Offers security tools (Security Groups, NACLs)
- Maintains physical security
- Provides API Gateway, Load Balancers

**Your Part (DevOps Engineer)**:
- Configure VPC properly
- Set up Security Groups correctly
- Configure NACLs appropriately
- Use load balancers wisely
- Monitor and maintain security

**Remember**: AWS gives you tools, YOU must use them correctly!

---

## Traffic Flow Review

### Complete Path from User to Application

```
User (Internet)
    ↓
Internet Gateway (Entry to VPC)
    ↓
Public Subnet (Accessible from internet)
    ↓
Load Balancer (Distributes traffic)
    ↓
Route Table (Guides traffic)
    ↓
NACL (First layer of defense - Subnet level)
    ↓
Security Group (Last layer of defense - Instance level)
    ↓
EC2 Instance
    ↓
Application
```

**Last Point of Security**: Security Groups and NACLs

---

## Security Groups

### What are Security Groups?

**Definition**: Virtual firewall at EC2 instance level

**Level**: Instance-specific (each EC2 has its own)

**Purpose**: Control traffic to and from EC2 instances

### Real Example from Day 3

**What Happened**:
1. Created EC2 instance
2. Deployed Jenkins application
3. Tried to access Jenkins
4. **Blocked by default!**
5. Opened port 8080 in Security Group
6. Jenkins became accessible

**Why Blocked**: AWS default = deny all inbound traffic

### Inbound vs Outbound Traffic

**Inbound Traffic**:
- Traffic COMING TO your application
- User accessing your website
- External requests to your server

**Example**: 
- User accessing amazon.com
- You accessing Jenkins on EC2

**Outbound Traffic**:
- Traffic GOING FROM your application
- Your app accessing external services
- Downloading packages from internet

**Example**:
- Amazon.com calling Razorpay API
- Amazon.com calling Amazon Pay
- EC2 downloading updates from internet

### AWS Default Security Group Behavior

**Inbound Rules (Default)**:
- **DENY EVERYTHING** by default
- You must explicitly allow traffic
- No ports open initially

**Outbound Rules (Default)**:
- **ALLOW EVERYTHING** (except port 25)
- Your app can access any external service
- You can restrict if needed

### Port 25 Story

**Why Port 25 Blocked?**
- Port 25 = Email/SMTP service
- AWS blocks to prevent spam
- Protects AWS IP reputation
- Prevents malicious email activity

**Lesson**: AWS protects its infrastructure and yours!

---

## Network ACLs (NACLs)

### What are NACLs?

**Full Name**: Network Access Control Lists

**Definition**: Firewall at subnet level

**Level**: Applies to entire subnet (all instances in it)

**Purpose**: Additional security layer before Security Groups

### Why NACLs When We Have Security Groups?

**Problem Scenario**:

Development Team 1:
- Gets EC2 instance
- Wants quick access
- Opens ALL PORTS in Security Group
- Allows traffic from ANYWHERE
- **Security risk!**

**DevOps Engineer's Solution**:
- Use NACLs at subnet level
- Block dangerous ports for entire subnet
- Even if developer opens all ports, NACL blocks
- Organizational security enforced

### Security Groups vs NACLs

| Feature | Security Groups | NACLs |
|---------|----------------|-------|
| **Level** | Instance (EC2) | Subnet |
| **Applies To** | Single instance | All instances in subnet |
| **Rules** | Allow only | Allow AND Deny |
| **State** | Stateful | Stateless |
| **Default** | Deny inbound, Allow outbound | Allow all |
| **Use Case** | Instance-specific rules | Subnet-wide policies |

### Stateful vs Stateless

**Security Groups (Stateful)**:
- If you allow inbound, response automatically allowed
- Tracks connection state
- Return traffic automatically permitted

**NACLs (Stateless)**:
- Must explicitly allow both inbound AND outbound
- Doesn't track connection state
- Each direction needs separate rule

---

## NACL as Automation Tool

### Scenario: 10,000 EC2 Instances

**Without NACLs**:
- Configure Security Group for each instance
- 10,000 configurations needed
- Time-consuming and error-prone
- Hard to maintain consistency

**With NACLs**:
- Configure NACL once at subnet level
- Applies to all 10,000 instances automatically
- Easy to maintain
- Consistent security policy

**Use Case**: 
- Development Team 3 has 50 instances
- All in same subnet
- Apply NACL once
- All 50 instances protected

---

## NACL Rule Priority

### How Rules are Evaluated

**Rule Numbers**: Determine order of evaluation

**Example Rules**:
```
Rule 100: Allow all traffic
Rule 200: Deny port 8000
Rule * (star): Deny all (default)
```

**Evaluation Order**:
1. Check Rule 100 first (lowest number)
2. If matches, apply and STOP
3. If not, check Rule 200
4. If not, check Rule * (catch-all)

### Important: First Match Wins!

**Scenario 1**:
```
Rule 100: Allow all traffic
Rule 200: Deny port 8000
```
**Result**: Port 8000 is ALLOWED (Rule 100 matches first)

**Scenario 2**:
```
Rule 100: Deny port 8000
Rule 200: Allow all traffic
```
**Result**: Port 8000 is DENIED (Rule 100 matches first)

**Key Lesson**: Order matters! Lower number = Higher priority

---

## Hands-On Practical

### What We'll Build

**Architecture**:
- Custom VPC (not default)
- Public subnet
- EC2 instance in public subnet
- Python HTTP server on port 8000
- Test Security Groups and NACLs

### Step 1: Create Custom VPC

**Why Custom VPC?**
- Default VPC = AWS managed
- Organizations use custom VPCs
- More control and customization

**Process**:

1. Go to VPC service
2. Click "Create VPC"
3. Select "VPC and more" (AWS creates everything)

**What "VPC and more" Creates**:
- VPC itself
- Public subnets (both AZs)
- Private subnets (both AZs)
- Internet Gateway
- Route Tables
- NAT Gateway (optional)
- S3 VPC Endpoints

4. **Name**: demo-vpc
5. **IP Range**: 10.0.0.0/16 (65,536 IPs)
6. **Availability Zones**: 2 (for high availability)
7. **Public Subnets**: 2
8. **Private Subnets**: 2
9. Click "Create VPC"

**What AWS Creates**:
- VPC with DNS configuration
- Subnets (public + private)
- Internet Gateway
- Route Tables
- Route associations
- Default NACLs
- Default Security Groups

### Understanding IP Ranges

**Example 1**: `10.0.0.0/16`
- Available IPs: 65,536
- Large VPC

**Example 2**: `10.0.0.0/24`
- Available IPs: 256
- Small subnet

**Tip**: Use AWS IP calculator if confused!

### Step 2: View VPC Resource Map

1. Go to VPC → Your VPC
2. Click "Resource Map"
3. See visual diagram of your VPC

**What You'll See**:
- VPC structure
- Subnets (public/private)
- Internet Gateway
- Route Tables
- Connections between components

### Step 3: Create EC2 Instance in VPC

**Important Differences from Day 3**:

1. **Name**: demo-instance
2. **OS**: Ubuntu
3. **Instance Type**: t2.micro (free tier)
4. **Key Pair**: Use existing (aws-login)

5. **Network Settings** (CRITICAL):
   - **VPC**: Select "demo-vpc" (NOT default VPC)
   - **Subnet**: Select "demo-vpc-subnet-public1-us-east-1a"
   - **Auto-assign Public IP**: Enable
   - **Security Group**: Create new

**Why Public Subnet?**
- Industry practice: Use private subnet
- Today's demo: Using public for simplicity
- No load balancer setup needed

6. Click "Launch Instance"

### Step 4: Install Python Application

**Connect to Instance**:

```bash
# Change key permissions
chmod 600 aws-login.pem

# Connect via SSH
ssh -i aws-login.pem ubuntu@<PUBLIC_IP>
```

**Update Packages**:

```bash
# Update package list
sudo apt update
```

**Check Python**:

```bash
# Verify Python installed
python3 --version
```

**Run Simple HTTP Server**:

```bash
# Start server on port 8000
python3 -m http.server 8000
```

**What This Does**:
- Starts web server on port 8000
- Lists directory contents
- Simple test application

### Step 5: Test Access (Will Fail)

**Try to Access**:

```
http://<PUBLIC_IP>:8000
```

**Result**: Cannot connect (timeout)

**Why?**
- Security Group blocks by default
- Only port 22 (SSH) allowed
- Port 8000 not opened

### Step 6: Check NACL Configuration

1. Go to VPC → Network ACLs
2. Find NACL for demo-vpc
3. Check Inbound Rules

**Default NACL Rules**:
```
Rule 100: Allow ALL traffic
Rule *: Deny ALL traffic
```

**Meaning**: NACL allows everything by default

**Traffic Flow So Far**:
```
Internet → Internet Gateway → NACL (✓ Allowed) → Route Table → Security Group (✗ BLOCKED)
```

### Step 7: Open Port in Security Group

1. Go to EC2 → Security Groups
2. Find security group for demo-instance
3. Click "Edit inbound rules"
4. Click "Add rule"

**New Rule**:
- **Type**: Custom TCP
- **Port**: 8000
- **Source**: 0.0.0.0/0 (Anywhere IPv4)

5. Click "Save rules"

### Step 8: Test Access (Will Work)

**Try Again**:

```
http://<PUBLIC_IP>:8000
```

**Result**: Success! Directory listing appears

**Terminal Output**:
```
GET / HTTP/1.1" 200
```

**Meaning**: Request successful (200 = OK)

---

## Testing NACL Power

### Scenario: DevOps Blocks Port 8000

**Situation**:
- Developer opened port 8000 in Security Group
- Organization policy: Port 8000 must be blocked
- DevOps engineer enforces at NACL level

### Step 1: Block Port in NACL

1. Go to VPC → Network ACLs
2. Select demo-vpc NACL
3. Edit Inbound Rules
4. Remove "Allow all" rule

**Add New Rule**:
- **Rule Number**: 100
- **Type**: Custom TCP
- **Port**: 8000
- **Source**: 0.0.0.0/0
- **Action**: DENY

5. Save changes

### Step 2: Test Access (Will Fail)

**Try to Access**:

```
http://<PUBLIC_IP>:8000
```

**Result**: Cannot connect (blocked)

**Why?**
- NACL blocks at subnet level
- Security Group allows, but NACL denies
- NACL = First layer of defense
- NACL wins!

**Key Lesson**: NACL overrides Security Group permissions

---

## Understanding Rule Priority

### Experiment 1: Order Matters

**Configuration**:
```
Rule 100: Allow all traffic
Rule 200: Deny port 8000
```

**Result**: Port 8000 is ACCESSIBLE

**Why?**
- Rule 100 checked first
- Matches "allow all"
- Rule 200 never evaluated
- Traffic allowed

### Experiment 2: Reverse Order

**Configuration**:
```
Rule 100: Deny port 8000
Rule 110: Allow all traffic
```

**Result**: Port 8000 is BLOCKED

**Why?**
- Rule 100 checked first
- Matches "deny port 8000"
- Rule 110 never evaluated for port 8000
- Traffic denied

**Critical Understanding**: First matching rule wins!

---

## Advanced NACL Use Cases

### Block by IP Address

**Scenario**: Suspicious traffic from specific country

**Solution**:
```
Rule 100: Deny traffic from 3.4.5.6/32 (specific IP)
Rule 110: Allow all other traffic
```

**Result**: That IP blocked, others allowed

### Block IP Range

**Scenario**: Block entire IP range

**Solution**:
```
Rule 100: Deny traffic from 3.4.0.0/16 (65,536 IPs)
Rule 110: Allow all other traffic
```

**Result**: Entire range blocked

### Multiple Port Blocks

**Scenario**: Block multiple dangerous ports

**Solution**:
```
Rule 100: Deny port 8000
Rule 110: Deny port 9000
Rule 120: Deny port 10000
Rule 200: Allow all other traffic
```

---

## Real-World Scenarios

### Scenario 1: Developer Mistake

**Problem**:
- Developer opens all ports (0-65535)
- Allows traffic from anywhere (0.0.0.0/0)
- Major security risk

**DevOps Solution**:
- Configure NACL to block dangerous ports
- Even if Security Group allows, NACL blocks
- Organization protected

### Scenario 2: Compliance Requirements

**Problem**:
- Regulation requires blocking certain countries
- 1000 EC2 instances in subnet

**DevOps Solution**:
- Configure NACL to block IP ranges
- Applies to all 1000 instances automatically
- Compliance achieved efficiently

### Scenario 3: DDoS Protection

**Problem**:
- Attack from specific IP range
- Need immediate blocking

**DevOps Solution**:
- Add NACL rule to deny attacker IPs
- Takes effect immediately
- All instances in subnet protected

---

## Best Practices

### Security Groups

1. **Principle of Least Privilege**
   - Open only required ports
   - Restrict source IPs when possible
   - Don't use 0.0.0.0/0 unless necessary

2. **Use Descriptive Names**
   - "web-server-sg" not "sg-123"
   - Document purpose in description

3. **Regular Audits**
   - Review rules quarterly
   - Remove unused rules
   - Check for overly permissive rules

### NACLs

1. **Use for Subnet-Wide Policies**
   - Block known bad IPs
   - Enforce organizational policies
   - Compliance requirements

2. **Keep Rules Simple**
   - Don't overcomplicate
   - Document rule purposes
   - Use consistent numbering (100, 110, 120...)

3. **Remember Rule Order**
   - Lower numbers = Higher priority
   - Test thoroughly
   - Document evaluation logic

### Combined Strategy

**Defense in Depth**:
```
Layer 1: Internet Gateway (AWS managed)
Layer 2: NACL (Subnet-wide policies)
Layer 3: Security Group (Instance-specific)
Layer 4: Application-level security
```

---

## Common Mistakes

### Mistake 1: Opening All Ports

**Bad**:
```
Security Group: Allow 0-65535 from 0.0.0.0/0
```

**Good**:
```
Security Group: Allow port 80, 443 from 0.0.0.0/0
                Allow port 22 from MY_IP only
```

### Mistake 2: Forgetting NACL is Stateless

**Problem**: Allow inbound but forget outbound

**Solution**: Remember to allow both directions

### Mistake 3: Wrong Rule Order in NACL

**Bad**:
```
Rule 100: Allow all
Rule 200: Deny port 8000  (Never evaluated!)
```

**Good**:
```
Rule 100: Deny port 8000
Rule 200: Allow all
```

### Mistake 4: Not Using NACLs

**Problem**: Relying only on Security Groups

**Better**: Use both for defense in depth

---

## Troubleshooting Guide

### Cannot Access Application

**Check 1: Security Group**
- Is port open?
- Is source IP allowed?
- Inbound rule exists?

**Check 2: NACL**
- Is traffic allowed?
- Check rule order
- Both inbound and outbound?

**Check 3: Route Table**
- Routes configured correctly?
- Internet Gateway attached?

**Check 4: Application**
- Is application running?
- Listening on correct port?
- Check with: `netstat -tulpn`

---

## Your Assignment

### Task 1: Recreate Demo

1. Create custom VPC
2. Launch EC2 in public subnet
3. Run Python HTTP server
4. Test with Security Group only
5. Test with NACL blocking

### Task 2: Experiment with Rules

1. Try different rule orders in NACL
2. Block specific IP addresses
3. Test multiple port blocks
4. Document your findings

### Task 3: Security Audit

1. Check your default VPC Security Groups
2. Identify overly permissive rules
3. Plan improvements
4. Document recommendations

### Task 4: Understand Traffic Flow

1. Draw complete traffic path
2. Mark where Security Group applies
3. Mark where NACL applies
4. Explain to someone else

---

## Key Takeaways

1. **Security = Shared Responsibility** (AWS + You)
2. **Security Groups = Instance-level firewall** (Allow only)
3. **NACLs = Subnet-level firewall** (Allow + Deny)
4. **NACLs evaluated before Security Groups**
5. **NACL rule order matters** (Lower number = Higher priority)
6. **Use both for defense in depth**
7. **Security Groups are stateful, NACLs are stateless**
8. **NACLs great for automation** (Apply to many instances)
9. **Always test security configurations**
10. **Document your security rules**

---

## Interview Questions

**Q: Difference between Security Group and NACL?**
**A**: Security Groups work at instance level, are stateful, and only allow rules. NACLs work at subnet level, are stateless, and support both allow and deny rules.

**Q: Can NACL override Security Group?**
**A**: Yes! NACL is evaluated first. If NACL denies, traffic never reaches Security Group.

**Q: When to use NACL vs Security Group?**
**A**: Use Security Groups for instance-specific rules. Use NACLs for subnet-wide policies, blocking bad IPs, or enforcing organizational standards.

**Q: What is stateful vs stateless?**
**A**: Stateful (Security Groups) automatically allows return traffic. Stateless (NACLs) requires explicit rules for both directions.

---

## Cost Considerations

### Free Components
- Security Groups (completely free)
- NACLs (completely free)
- VPC itself (free)

### No Hidden Costs
- Creating rules: Free
- Modifying rules: Free
- Number of rules: Free (within limits)

**Tip**: Security Groups and NACLs are free - use them generously!

---

## Next Steps

**Day 6**: VPC Hands-On Advanced
- Private subnets with NAT Gateway
- Load Balancer configuration
- Multi-tier application deployment
- Complete production setup

**Day 7**: VPC Best Practices
- High availability setup
- Multi-AZ deployment
- VPC Peering
- VPN connections

---

## Important Reminders

1. **Always stop EC2 instances after practice**
2. **Delete custom VPCs if not using** (avoid confusion)
3. **Document your security rules**
4. **Test before applying to production**
5. **Regular security audits**

---

**Remember**: Security Groups and NACLs are your last line of defense. Configure them wisely! 🛡️

*Practice this hands-on multiple times to master the concepts!*