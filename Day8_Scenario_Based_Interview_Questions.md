# Day 8: Scenario-Based Interview Questions

## Why This Topic?

**Modern Interviews**: Focus on scenarios, not definitions
- Instead of "What is VPC?" → "How would you design VPC for 2-tier app?"
- Instead of "What is EC2?" → "How would you handle scaling issues?"

**Topics Covered**: VPC, Subnets, Security Groups, EC2, IAM (Day 0-7)

---

## Question 1: Two-Tier Application Architecture

**Scenario**: Design VPC architecture for two-tier application that is highly available and scalable.

### Breaking Down the Question

**Part 1**: Two-tier application in VPC
**Part 2**: High availability + Scalability

### Answer

**High Availability Solution**:
- Use multiple Availability Zones (us-east-1a, us-east-1b)
- Deploy same application in both AZs
- If one AZ fails, other continues serving

**Scalability Solution**:
- Use Auto Scaling Groups
- Automatically add instances when traffic increases
- Remove instances when traffic decreases

**Architecture**:
- **Public Subnet**: Load Balancer (internet-facing)
- **Private Subnet**: Application servers (secure)
- **Multiple AZs**: For high availability
- **Auto Scaling**: For handling load variations

---

## Question 2: Subnet Internet Access Control

**Scenario**: VPC with multiple subnets. Restrict outbound internet for one subnet, allow for another.

### Answer

**Solution**: Modify Route Tables

**Steps**:
1. Identify subnet that needs restriction
2. Find route table attached to that subnet
3. Remove default route (0.0.0.0/0 → Internet Gateway)
4. Without Internet Gateway route, no internet access

**Key Point**: Route table controls internet access, not security groups

---

## Question 3: Private Subnet Internet Access

**Scenario**: Private subnet instances need internet for software updates. How to allow this?

### Answer

**Solution**: NAT Gateway

**How It Works**:
1. Place NAT Gateway in public subnet
2. Configure private subnet route table to send traffic to NAT Gateway
3. NAT Gateway forwards requests to internet
4. Uses Network Address Translation (masks private IPs)

**Security Benefit**: 
- Private instance IP: 172.16.3.4
- Internet sees: NAT Gateway public IP
- Private IP stays hidden

---

## Question 4: EC2 Instance Communication

**Scenario**: EC2 instances in VPC need to communicate using private IPs.

### Answer

**Requirements**:
1. **Same VPC**: Instances must be in same VPC
2. **Network Connectivity**: Either same subnet OR proper routing between subnets

**Options**:
- **Same Subnet**: Direct communication (same IP range)
- **Different Subnets**: Route tables must allow communication
- **Different VPCs**: Use VPC Peering (covered in future)

---

## Question 5: Strict Network Access Control

**Scenario**: Implement strict network access control for VPC.

### Answer

**Solution**: Use NACLs (Network Access Control Lists)

**Why NACLs**:
- **Subnet-level security** (applies to all instances in subnet)
- **Allow and Deny rules** (Security Groups only allow)
- **Extra layer of defense**
- **Fine-grained control** (IP ranges, ports, protocols)

**Best Practice**: Use both NACLs and Security Groups (defense in depth)

---

## Question 6: Isolated Environment in VPC

**Scenario**: Create isolated environment within VPC for sensitive workloads.

### Answer

**Solution**: Private Subnet with No Internet Access

**Steps**:
1. Create private subnet within VPC
2. Don't attach Internet Gateway route
3. No NAT Gateway access
4. No Bastion Host access (if needed)
5. Use dedicated security groups

**Result**: Completely isolated from internet and other subnets

---

## Question 7: AWS Services Access Within VPC

**Scenario**: Application needs secure access to AWS services (S3, DynamoDB) within VPC.

### Answer

**Solution**: VPC Endpoints

**How It Works**:
- Direct connection to AWS services
- Traffic stays within AWS network
- No internet gateway needed
- More secure than internet routing

**Note**: VPC Endpoints covered in detail with S3 lessons

---

## Question 8: Security Groups vs NACLs

**Scenario**: Explain difference between Security Groups and NACLs.

### Answer

| Feature | Security Groups | NACLs |
|---------|----------------|-------|
| **Level** | Instance | Subnet |
| **Rules** | Allow only | Allow + Deny |
| **State** | Stateful | Stateless |
| **Scope** | Single instance | All instances in subnet |

**Stateful vs Stateless**:

**Security Groups (Stateful)**:
- Open port 80 outbound to google.com
- Response automatically allowed back
- No need to open inbound for response

**NACLs (Stateless)**:
- Open port 80 outbound to google.com
- Must also open inbound for response
- Each direction needs separate rule

---

## Question 9: IAM Components

**Scenario**: Explain IAM Users, Groups, Roles, and Policies.

### Answer

**IAM Users**:
- For people (developers, admins)
- Authentication (who can enter AWS)
- Each person gets own user

**IAM Policies**:
- Define permissions (what can be done)
- Authorization (EC2 access, S3 read, etc.)
- Attached to users, groups, or roles

**IAM Groups**:
- Collection of users with similar needs
- Attach policies to group, not individual users
- Example: "Developers" group with dev permissions

**IAM Roles**:
- For services, not people
- EC2 instance needs S3 access → Attach role
- Applications accessing AWS services
- Temporary credentials

**Example**:
- 500 developers need new permission
- Without groups: Update 500 individual policies
- With groups: Update 1 group policy

---

## Question 10: Bastion Host Setup

**Scenario**: Private subnet instances need administrative access but no direct internet access.

### Answer

**Solution**: Bastion Host (Jump Server)

**Architecture**:
```
Your Laptop → Bastion Host (Public Subnet) → Private Instances
```

**Setup Steps**:
1. Create EC2 in public subnet (Bastion Host)
2. Assign public IP to Bastion
3. Copy SSH keys to Bastion
4. SSH to Bastion, then SSH to private instances

**Security Benefits**:
- Single point of access
- Centralized logging
- Audit trail of who accessed what
- Additional security rules possible

**Protocols**:
- Linux: SSH (port 22)
- Windows: RDP (port 3389)

---

## Interview Tips

### How to Answer Scenario Questions

1. **Break down the question** into parts
2. **Identify key requirements** (security, availability, scalability)
3. **Explain your approach** step by step
4. **Mention alternatives** if applicable
5. **Highlight security considerations**

### Common Patterns

**High Availability** → Multiple AZs
**Scalability** → Auto Scaling Groups
**Security** → Private subnets + Security Groups + NACLs
**Internet Access** → NAT Gateway for private subnets
**Admin Access** → Bastion Host
**Service Communication** → VPC Endpoints

### Key Phrases to Use

- "For high availability, I would use multiple AZs"
- "To ensure security, I would place apps in private subnets"
- "For scalability, I would implement Auto Scaling Groups"
- "To control access, I would use both Security Groups and NACLs"
- "For admin access, I would set up a Bastion Host"

---

## Practice Questions

Try answering these yourself:

1. How would you migrate an on-premises application to AWS securely?
2. Design architecture for a web application expecting traffic spikes during holidays.
3. How would you troubleshoot connectivity issues between EC2 instances?
4. Explain how you would set up disaster recovery for a critical application.
5. How would you ensure compliance and auditing for AWS resource access?

---

## Key Takeaways

1. **Scenario questions test practical knowledge**, not just theory
2. **Always consider security, availability, and scalability**
3. **Break complex questions into smaller parts**
4. **Use real-world examples from Day 7 project**
5. **Practice explaining architectures clearly**
6. **Know when to use each AWS service**
7. **Understand trade-offs between different approaches**

---

## Resources

- **GitHub Repository**: All questions and answers available
- **Day 1-7 Videos**: Foundation knowledge for these scenarios
- **AWS Documentation**: For deeper understanding
- **Practice**: Try building these scenarios hands-on

---

**Remember**: These scenarios are based on real-world problems. The more you practice building these architectures, the better you'll be at interviews! 🚀

*Master these 10 scenarios and you'll be ready for most AWS VPC interviews!*