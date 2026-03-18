# Day 6: AWS Route 53 - DNS as a Service

## What You'll Learn Today
1. What is DNS (Domain Name System)
2. What is Route 53 and why use it
3. How DNS works in real world
4. Route 53 components
5. How Route 53 integrates with VPC

---

## What is DNS?

### DNS = Domain Name System

**Simple Definition**: Service that converts domain names to IP addresses

### Real-World Usage

**What We Use Daily**:
- amazon.com
- flipkart.com
- instagram.com
- google.com

**What We DON'T Use**:
- 3.6.10.171
- 192.168.1.100
- 54.239.28.85

**Why?** Domain names are easier to remember than IP addresses!

---

## Understanding the Problem

### Scenario: Accessing Applications

**Previous Classes Setup**:
```
VPC
  ↓
Internet Gateway
  ↓
Public Subnet (Load Balancer)
  ↓
Private Subnet (Application)
```

**Problem 1: Hard to Remember**

Without DNS:
- Load Balancer IP: `3.6.10.171`
- User must remember: `http://3.6.10.171`
- Multiple apps = Multiple IPs to remember
- Practically impossible!

With DNS:
- Domain: `amazon.com`
- Easy to remember
- User-friendly

**Problem 2: IP Addresses Change**

**Scenarios Where IP Changes**:
- Restart load balancer
- No static IP assigned
- Home/college projects (dynamic IPs)
- Network changes
- Server migrations

**Solution**: Use domain names (they stay same even if IP changes)

---

## What is Route 53?

### Route 53 = DNS as a Service

**AWS Services Pattern**:
- EC2 = Compute as a Service
- EKS = Kubernetes as a Service
- Route 53 = DNS as a Service

**What Route 53 Provides**:
- Domain name registration
- DNS record management
- Health checks
- Traffic routing
- Integration with AWS services

---

## How DNS Works

### Basic DNS Flow

**Step 1: User Makes Request**
```
User types: amazon.com
```

**Step 2: DNS Lookup**
```
DNS Server checks records
Finds: amazon.com → 3.6.10.171
```

**Step 3: Returns IP**
```
Browser receives: 3.6.10.171
Connects to that IP
```

**Step 4: Application Responds**
```
Load Balancer at 3.6.10.171 receives request
Forwards to application
User gets response
```

### DNS Records

**What DNS Stores**:
```
Domain Name          →    IP Address
amazon.com          →    3.6.10.171
flipkart.com        →    52.84.23.100
myapp.com           →    54.239.28.85
```

**DNS = Database of domain-to-IP mappings**

---

## Route 53 in VPC Architecture

### Complete Traffic Flow with Route 53

```
User (Internet)
    ↓
Route 53 (DNS Resolution)
    ↓
Internet Gateway
    ↓
Public Subnet
    ↓
Load Balancer
    ↓
Private Subnet
    ↓
Application
```

### Detailed Flow

**Step 1: User Request**
```
User types: amazon.com in browser
```

**Step 2: Route 53 Intercepts**
```
Route 53 receives DNS query
Checks DNS records
Finds: amazon.com → Load Balancer IP
```

**Step 3: DNS Resolution**
```
Route 53 returns: 3.6.10.171 (Load Balancer IP)
Browser now knows where to connect
```

**Step 4: Request to Load Balancer**
```
Browser connects to 3.6.10.171
Request goes through Internet Gateway
Reaches Load Balancer in Public Subnet
```

**Step 5: Application Response**
```
Load Balancer forwards to Application
Application processes request
Response sent back to user
```

---

## Route 53 Components

### 1. Domain Registration

**What**: Buy domain names through AWS

**How It Works**:

**Traditional Way (Before AWS)**:
1. Go to GoDaddy
2. Search for domain (pavankumar.com)
3. If taken, try pavankumar.in
4. If taken, try pavankumar.cricket
5. Purchase domain
6. Go to hosting provider (Hostinger)
7. Set up hosting
8. Configure DNS records
9. Maintain everything

**AWS Route 53 Way**:
1. Go to Route 53
2. Search for domain
3. Purchase directly from AWS
4. AWS handles everything
5. Integrated with AWS services

**Options**:
- **Option 1**: Buy domain from AWS Route 53
- **Option 2**: Buy domain elsewhere (GoDaddy) and integrate with Route 53

**Cost**: Upfront payment (not pay-as-you-go)

**Domain Extensions**:
- .com (most popular)
- .in (India)
- .co.uk (UK)
- .cricket, .xyz, .tech (alternatives)

### 2. Hosted Zones

**What**: Container for DNS records

**Purpose**: Store all DNS mappings for your domain

**Types**:

**Public Hosted Zone**:
- Accessible from internet
- For public websites
- Example: amazon.com

**Private Hosted Zone**:
- Only within VPC
- For internal applications
- Example: internal.mycompany.com

**What's Inside Hosted Zone**:
```
DNS Records:
- myapp.com        → 3.6.10.171
- api.myapp.com    → 52.84.23.100
- admin.myapp.com  → 54.239.28.85
```

**How It Works**:

**If Domain Bought from Route 53**:
1. AWS automatically creates hosted zone
2. DNS records auto-configured
3. Ready to use

**If Domain Bought Elsewhere**:
1. Create hosted zone in Route 53
2. Get nameservers from Route 53
3. Update nameservers at domain registrar (GoDaddy)
4. Create DNS records in hosted zone

### 3. Health Checks

**What**: Monitor health of your resources

**Purpose**: Ensure traffic only goes to healthy servers

**How It Works**:

**Scenario**: Application on 2 servers in different AZs
```
Server 1 (us-east-1a): 3.6.10.171
Server 2 (us-east-1b): 52.84.23.100
```

**Health Check Process**:
1. Route 53 pings Server 1 every 30 seconds
2. Route 53 pings Server 2 every 30 seconds
3. Checks if servers respond

**If Server 1 Healthy**:
- Route 53 sends traffic to Server 1

**If Server 1 Unhealthy**:
- Route 53 stops sending traffic to Server 1
- All traffic goes to Server 2

**If Both Healthy**:
- Route 53 distributes traffic (load balancing)

**Check Intervals**:
- Standard: Every 30 seconds
- Fast: Every 10 seconds (costs more)

**What Health Checks Monitor**:
- HTTP/HTTPS endpoints
- TCP connections
- Response time
- Status codes

---

## DNS Records Types

### Common Record Types

**A Record (Address)**:
- Maps domain to IPv4 address
- Example: `myapp.com → 3.6.10.171`

**AAAA Record**:
- Maps domain to IPv6 address
- Example: `myapp.com → 2001:0db8:85a3::8a2e:0370:7334`

**CNAME Record (Canonical Name)**:
- Maps domain to another domain
- Example: `www.myapp.com → myapp.com`

**MX Record (Mail Exchange)**:
- For email servers
- Example: `myapp.com → mail.myapp.com`

**TXT Record**:
- Text information
- Used for verification, SPF records

**NS Record (Name Server)**:
- Specifies authoritative DNS servers
- Automatically created by Route 53

---

## Route 53 Routing Policies

### 1. Simple Routing

**What**: One domain → One IP

**Use Case**: Single server application

**Example**:
```
myapp.com → 3.6.10.171
```

### 2. Weighted Routing

**What**: Distribute traffic by percentage

**Use Case**: A/B testing, gradual rollouts

**Example**:
```
myapp.com → 80% to Server 1
myapp.com → 20% to Server 2
```

### 3. Latency-Based Routing

**What**: Route to server with lowest latency

**Use Case**: Global applications

**Example**:
```
User in India → Mumbai server
User in US → Virginia server
```

### 4. Failover Routing

**What**: Primary and backup servers

**Use Case**: High availability

**Example**:
```
Primary: Server 1 (if healthy)
Secondary: Server 2 (if primary fails)
```

### 5. Geolocation Routing

**What**: Route based on user location

**Use Case**: Region-specific content

**Example**:
```
Users in India → India server
Users in Europe → Europe server
```

**Note**: Routing policies covered in detail in future projects

---

## Why Use Route 53?

### Benefits

**1. Integrated with AWS**
- Works seamlessly with EC2, Load Balancers, S3
- No external configuration needed
- Single console for everything

**2. Highly Available**
- 100% uptime SLA
- Distributed globally
- No single point of failure

**3. Scalable**
- Handles millions of queries
- Auto-scales with demand
- No capacity planning needed

**4. Cost-Effective**
- Pay only for what you use
- No minimum fees (except domain registration)
- Cheaper than managing own DNS

**5. Fast**
- Global network of DNS servers
- Low latency responses
- Anycast routing

**6. Secure**
- DNSSEC support
- IAM integration
- Private hosted zones for internal apps

**7. Easy to Use**
- Simple console interface
- API access
- Infrastructure as Code support

---

## Route 53 vs Traditional DNS

### Traditional Approach

**Steps**:
1. Buy domain from GoDaddy ($10-15/year)
2. Buy hosting from Hostinger ($50-100/year)
3. Configure DNS manually
4. Set up nameservers
5. Manage DNS records
6. Monitor uptime yourself
7. Handle scaling issues
8. Deal with multiple vendors

**Problems**:
- Complex setup
- Multiple services to manage
- Downtime risks
- Scaling challenges
- No integration with cloud

### Route 53 Approach

**Steps**:
1. Buy domain from Route 53 (or integrate existing)
2. Create hosted zone (automatic if bought from AWS)
3. Create DNS records
4. Done!

**Benefits**:
- Simple setup
- Single service
- High availability built-in
- Auto-scaling
- Integrated with AWS

---

## Real-World Example

### Scenario: Launching MyApp.com

**Step 1: Domain Registration**
```
Go to Route 53 → Domain Registration
Search: myapp.com
Purchase: $12/year
```

**Step 2: Hosted Zone Created**
```
AWS automatically creates hosted zone
Nameservers assigned:
- ns-123.awsdns-12.com
- ns-456.awsdns-34.net
- ns-789.awsdns-56.org
- ns-012.awsdns-78.co.uk
```

**Step 3: Create DNS Records**
```
A Record: myapp.com → Load Balancer IP
CNAME: www.myapp.com → myapp.com
```

**Step 4: Users Access**
```
User types: myapp.com
Route 53 resolves to Load Balancer
Load Balancer forwards to Application
User sees website
```

---

## Integration with VPC Components

### Complete Architecture

```
User
  ↓
Route 53 (DNS: myapp.com → LB IP)
  ↓
Internet Gateway
  ↓
Public Subnet
  ├── Load Balancer (receives traffic)
  └── NAT Gateway
  ↓
Private Subnet
  ├── Application Server 1
  ├── Application Server 2
  └── Database
```

### How Components Work Together

**Route 53**:
- Resolves domain to Load Balancer IP
- Performs health checks
- Routes traffic intelligently

**Internet Gateway**:
- Allows traffic in/out of VPC
- Public IP management

**Load Balancer**:
- Receives traffic from Route 53
- Distributes to application servers
- SSL/TLS termination

**Application Servers**:
- Process requests
- Return responses
- Stay in private subnet (secure)

---

## Cost Breakdown

### Route 53 Pricing

**Hosted Zones**:
- $0.50 per hosted zone per month
- First 25 hosted zones

**DNS Queries**:
- First 1 billion queries/month: $0.40 per million
- Very cheap for most applications

**Domain Registration**:
- .com: ~$12/year
- .in: ~$9/year
- .co.uk: ~$9/year
- Varies by extension

**Health Checks**:
- AWS endpoints: $0.50/month
- Non-AWS endpoints: $0.75/month

**Example Monthly Cost**:
```
1 Hosted Zone: $0.50
10 million queries: $0.04
2 Health Checks: $1.00
Total: ~$1.54/month
```

**Domain registration is separate (annual cost)**

---

## Common Use Cases

### 1. Simple Website

**Setup**:
- Domain: mywebsite.com
- Single EC2 instance
- A record pointing to EC2 IP

### 2. Load Balanced Application

**Setup**:
- Domain: myapp.com
- Application Load Balancer
- Multiple EC2 instances
- A record pointing to ALB

### 3. Multi-Region Application

**Setup**:
- Domain: globalapp.com
- Servers in US, Europe, Asia
- Latency-based routing
- Users routed to nearest server

### 4. Blue-Green Deployment

**Setup**:
- Blue environment (current)
- Green environment (new version)
- Weighted routing (gradually shift traffic)

### 5. Disaster Recovery

**Setup**:
- Primary region: us-east-1
- Backup region: us-west-2
- Failover routing
- Auto-switch if primary fails

---

## Best Practices

### 1. Use Alias Records for AWS Resources

**Why**: Free queries, automatic updates

**Example**:
```
Instead of: A record → Load Balancer IP
Use: Alias record → Load Balancer DNS name
```

### 2. Enable Health Checks

**Why**: Automatic failover, high availability

**Setup**:
- Monitor all critical endpoints
- Set appropriate intervals
- Configure alarms

### 3. Use Private Hosted Zones

**Why**: Internal applications stay internal

**Use Case**:
- Internal APIs
- Admin panels
- Database endpoints

### 4. Implement TTL Wisely

**TTL (Time To Live)**: How long DNS record is cached

**Short TTL (60 seconds)**:
- Quick changes
- Higher query costs
- Use during migrations

**Long TTL (24 hours)**:
- Stable applications
- Lower costs
- Use for production

### 5. Monitor DNS Queries

**Why**: Detect issues, optimize costs

**Tools**:
- CloudWatch metrics
- Route 53 query logs
- Cost explorer

---

## Common Mistakes to Avoid

### Mistake 1: Not Using Alias Records

**Problem**: Paying for queries unnecessarily

**Solution**: Use Alias records for AWS resources

### Mistake 2: Forgetting to Update Nameservers

**Problem**: Domain bought elsewhere, nameservers not updated

**Solution**: Always update nameservers at registrar

### Mistake 3: No Health Checks

**Problem**: Traffic sent to unhealthy servers

**Solution**: Enable health checks for all critical resources

### Mistake 4: Wrong TTL Values

**Problem**: Can't make quick changes or paying too much

**Solution**: Balance between flexibility and cost

### Mistake 5: Not Testing DNS Changes

**Problem**: DNS misconfiguration causes downtime

**Solution**: Test in staging, use dig/nslookup commands

---

## Testing DNS Configuration

### Using dig Command

```bash
# Check A record
dig myapp.com

# Check specific nameserver
dig @ns-123.awsdns-12.com myapp.com

# Check CNAME
dig www.myapp.com
```

### Using nslookup Command

```bash
# Basic lookup
nslookup myapp.com

# Specify nameserver
nslookup myapp.com ns-123.awsdns-12.com
```

### Using Browser

```
# Direct IP access
http://3.6.10.171

# Domain access
http://myapp.com

# Both should show same content
```

---

## Key Takeaways

1. **DNS = Domain Name System** (converts names to IPs)
2. **Route 53 = DNS as a Service** (AWS managed)
3. **Three main components**: Domain Registration, Hosted Zones, Health Checks
4. **Route 53 sits before Internet Gateway** in traffic flow
5. **Hosted Zones store DNS records**
6. **Health Checks ensure high availability**
7. **Multiple routing policies** for different use cases
8. **Integrated with all AWS services**
9. **Cost-effective and highly available**
10. **Essential for production applications**

---

## Interview Questions

**Q: What is Route 53?**
**A**: AWS's DNS service that provides domain registration, DNS record management, health checks, and traffic routing with high availability and low latency.

**Q: Difference between A record and CNAME?**
**A**: A record maps domain to IP address. CNAME maps domain to another domain name.

**Q: What are hosted zones?**
**A**: Containers for DNS records. Public hosted zones for internet-facing apps, private for VPC-internal apps.

**Q: How does Route 53 ensure high availability?**
**A**: Global network of DNS servers, 100% uptime SLA, health checks, automatic failover, and multiple routing policies.

**Q: What is TTL in DNS?**
**A**: Time To Live - how long DNS record is cached. Lower TTL = more queries but faster changes. Higher TTL = fewer queries but slower changes.

---

## Next Steps

**Day 7**: Complete VPC Project
- Create public and private subnets
- Deploy application in private subnet
- Configure load balancer
- Set up Route 53 with custom domain
- Implement complete production architecture

**What You'll Build**:
- Full VPC with public/private subnets
- Load balancer in public subnet
- Application in private subnet
- Route 53 DNS configuration
- Security groups and NACLs
- Complete traffic flow

---

## Additional Resources

**AWS Documentation**:
- Route 53 Developer Guide
- DNS Best Practices
- Routing Policy Examples
- Health Check Configuration

**Practice**:
- Create hosted zone
- Add DNS records
- Test with dig/nslookup
- Monitor query logs

---

## Important Notes

1. **Domain registration is annual cost** (not monthly)
2. **Hosted zones cost $0.50/month** each
3. **DNS queries are very cheap** (millions for pennies)
4. **Health checks have small cost** ($0.50-0.75/month each)
5. **Always test DNS changes** before production
6. **Use Alias records** for AWS resources (free queries)
7. **Tomorrow's project** will tie everything together

---

**Remember**: Route 53 is the entry point for all your applications. Configure it correctly for reliable, fast, and secure access! 🌐

*Tomorrow: Complete hands-on VPC project with Route 53 integration!*
