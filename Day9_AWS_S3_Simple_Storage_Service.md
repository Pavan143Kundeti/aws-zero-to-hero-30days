# Day 9: AWS S3 - Simple Storage Service

## What You'll Learn Today
1. What is S3 and why it's popular
2. S3 characteristics and features
3. Storage classes and pricing
4. **Demo 1**: Bucket policies and access control
5. **Demo 2**: Static website hosting

---

## Why S3 is Popular

### The Storage Problem

**Personal Level**:
- Mobile phone runs out of storage
- Laptop needs more space
- Buy external hard disk or pen drive

**Organization Level**:
- Heavy databases (TB of data)
- Continuous database backups
- Application logs (30+ days)
- Dashboards, reports, CSV files
- Where to store all this data?

**AWS Solution**: Amazon S3 (Simple Storage Service)

---

## The Magic Number: 99.999999999%

### 11 Nines Reliability

**What it means**:
- Upload 1 billion objects over 100 years
- AWS guarantees only 1 object might be lost
- 99.999999999% reliability (11 nines)

**How AWS Achieves This**:
- Multiple replicas across availability zones
- Multiple copies in different data centers
- Automatic replacement if one copy fails
- Your data is virtually never lost

---

## What is S3?

**S3 = Simple Storage Service**

**Key Characteristics**:
1. **Highly Available & Durable** (11 nines reliability)
2. **Highly Scalable** (no storage limits)
3. **Secure** (encryption, access controls)
4. **Cost Effective** (very cheap storage)
5. **High Performance** (fast uploads/downloads)

---

## S3 Basics

### Global Accessibility

**Important**: S3 buckets are **globally accessible** but **regionally stored**

**Example**:
- Create bucket in Mumbai region (low latency)
- Access from anywhere in the world via HTTPS
- URL format: `https://bucket-name.s3.amazonaws.com`

### What DevOps Engineers Store

**Common Use Cases**:
- Application log files
- Database backups (3-4 TB dumps)
- Configuration files
- Reports and dashboards
- Any file type (no restrictions)

---

## Hands-On: Creating S3 Bucket

### Step 1: Create Bucket

1. Go to S3 service
2. Click "Create bucket"
3. **Bucket name**: Must be globally unique
   - Example: `app1-payments-prod-example.com`
   - Use naming standards for organization

4. **Region**: Choose closest to you (for low latency)
5. **Block Public Access**: Keep enabled (security)
6. **Versioning**: Default disabled
7. **Encryption**: Enabled by default
8. Click "Create bucket"

### Step 2: Upload Objects

1. Click on bucket name
2. Click "Upload"
3. Select files from computer
4. Click "Upload"

**Note**: Anything uploaded to S3 is called an "object"

---

## S3 Storage Classes

### Why Different Classes?

**Trade-off**: Cost vs Access Speed

**Like Hard Disks**:
- SSD: Fast access, expensive
- HDD: Slower access, cheaper

### Storage Class Options

| Class | Cost | Access Time | Use Case |
|-------|------|-------------|----------|
| **S3 Standard** | $0.023/GB | Immediate | Frequently accessed |
| **S3 Standard-IA** | $0.0125/GB | Immediate | Infrequently accessed |
| **S3 Glacier** | $0.004/GB | Minutes-Hours | Archive/Backup |
| **S3 Glacier Deep Archive** | $0.00099/GB | 12-48 hours | Long-term archive |

**Example**: 1TB for 1 year in Glacier Deep Archive = ~$12

### Choosing Storage Class

**Questions to Ask**:
- How often will you access this data?
- How quickly do you need it when requested?
- What's your budget?

**Recommendation**: Start with S3 Standard, move to cheaper classes for older data

---

## S3 Versioning

### What is Versioning?

**Like Git for Files**:
- Keep multiple versions of same file
- Retrieve old versions when needed
- Track changes over time

### Enable Versioning

1. Go to bucket → Properties
2. Find "Bucket Versioning"
3. Click "Edit" → "Enable"
4. Save changes

### How It Works

**Upload same file multiple times**:
- Version 1: Original file
- Version 2: Modified file
- Version 3: Another modification

**Access old versions**:
- Click on object
- See "Versions" tab
- Download any version needed

---

## Demo 1: Bucket Policies & Access Control

### Scenario

**Problem**: You're DevOps engineer with sensitive data in S3. Other team members have S3 access but shouldn't access your specific bucket.

### Step 1: Create IAM User

1. Go to IAM → Users
2. Create user: "demo-s3-bucket-user"
3. Enable console access
4. Set custom password
5. **Don't attach any policies yet**

### Step 2: Test No Access

1. Open incognito window
2. Login as new IAM user
3. Try to access S3
4. **Result**: "You don't have permissions to list buckets"

### Step 3: Grant S3 Access

1. Go back to root account
2. IAM → Users → demo-s3-bucket-user
3. Add permissions → Attach policies
4. Search "S3" → Select "AmazonS3FullAccess"
5. Add permissions

### Step 4: Test Full Access

1. Refresh IAM user session
2. Go to S3 → Can see all buckets
3. Click on your bucket
4. Can download files

**Problem**: They can access your sensitive bucket!

### Step 5: Create Bucket Policy

1. Go to your bucket → Permissions
2. Click "Bucket policy" → Edit
3. Use policy generator or copy from GitHub

**Policy Explanation**:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::your-bucket-name",
        "arn:aws:s3:::your-bucket-name/*"
      ],
      "Condition": {
        "StringNotEquals": {
          "aws:PrincipalArn": "arn:aws:iam::ACCOUNT-ID:root"
        }
      }
    }
  ]
}
```

**What it does**:
- **Deny** all actions on this bucket
- **For everyone** (Principal: "*")
- **Except** the bucket owner (Condition)

### Step 6: Test Restricted Access

1. IAM user tries to access bucket
2. **Result**: "Insufficient permissions to list objects"
3. **Success**: Bucket owner can still access, others cannot

---

## Demo 2: Static Website Hosting

### Why Host on S3?

**Benefits**:
- Very cheap hosting
- Globally accessible
- No server management
- Perfect for static sites

### Step 1: Prepare Website Files

**Create index.html**:
```html
<!DOCTYPE html>
<html>
<body>
<h1>My First Heading</h1>
<p>My first paragraph</p>
</body>
</html>
```

### Step 2: Upload to S3

1. Create new bucket (or use existing)
2. Upload index.html file
3. Verify file is uploaded

### Step 3: Enable Static Website Hosting

1. Go to bucket → Properties
2. Scroll to "Static website hosting"
3. Click "Edit"
4. Select "Enable"
5. **Index document**: index.html
6. Save changes

### Step 4: Make Bucket Public

**Problem**: Website shows "Access Denied"

**Solution**: Remove public access blocks

1. Go to Permissions tab
2. "Block public access" → Edit
3. **Uncheck** "Block all public access"
4. Save changes → Confirm

### Step 5: Add Public Read Policy

1. Go to Bucket policy → Edit
2. Add policy for public read access:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::your-bucket-name/*"
    }
  ]
}
```

3. Save policy

### Step 6: Access Website

1. Go to Properties → Static website hosting
2. Copy the website URL
3. Open in browser
4. **Success**: Your website is live!

---

## S3 Features Overview

### Bucket Properties

**Versioning**: Keep multiple file versions
**Tags**: Organize resources by project
**Encryption**: Data encrypted by default
**Access Logging**: Track who accesses what
**Event Notifications**: Trigger actions on file changes
**Object Lock**: Prevent file modifications
**Static Website Hosting**: Host websites directly

### Security Features

**Bucket Policies**: Control access at bucket level
**Access Control Lists (ACLs)**: Fine-grained permissions
**Encryption**: At rest and in transit
**Access Logging**: Audit trail
**Integration**: Works with CloudWatch, Lambda

---

## Performance Features

### Multi-part Upload

**Problem**: Uploading large files (4TB) takes days, might fail

**Solution**: Break into smaller chunks
- Upload 100MB pieces
- If one fails, retry only that piece
- Much more reliable for large files

### Transfer Acceleration

**Use CloudFront edge locations for faster uploads**

---

## Cost Optimization

### Lifecycle Policies

**Scenario**: Log files
- Day 0-30: S3 Standard (frequent access)
- Day 31-90: S3 Standard-IA (less frequent)
- Day 91+: Glacier (archive)

**Automatic transitions save money**

### Intelligent Tiering

**AWS automatically moves objects between storage classes based on access patterns**

---

## Important Limits

### Object Size Limits

**Single Object**: Maximum 5TB
**Workaround**: Split larger files

**Bucket Limits**: No limit on number of objects or total storage

### Naming Rules

**Bucket Names**:
- Globally unique
- 3-63 characters
- Lowercase letters, numbers, hyphens
- No spaces or special characters

---

## Best Practices

### Security

1. **Never make buckets public** unless absolutely necessary
2. **Use bucket policies** for fine-grained control
3. **Enable versioning** for important data
4. **Enable access logging** for audit trails
5. **Use encryption** (enabled by default)

### Cost Management

1. **Choose appropriate storage class**
2. **Set up lifecycle policies**
3. **Delete old versions** when not needed
4. **Monitor usage** with CloudWatch

### Organization

1. **Use consistent naming conventions**
2. **Tag all resources** for easy management
3. **Separate buckets** by environment (dev/staging/prod)
4. **Document bucket purposes**

---

## Common Use Cases

### DevOps Applications

**Log Storage**: Application and system logs
**Backup Storage**: Database and file backups
**Configuration Management**: Store config files
**Artifact Storage**: Build artifacts and releases
**Data Lake**: Raw data for analytics
**Content Distribution**: Static assets for applications

### Web Applications

**Static Website Hosting**: HTML, CSS, JS files
**Media Storage**: Images, videos, documents
**User Uploads**: Profile pictures, documents
**Content Backup**: Regular site backups

---

## Troubleshooting Common Issues

### Access Denied Errors

**Check**:
1. Bucket policy allows the action
2. IAM user has required permissions
3. Public access settings (if needed)
4. Object-level permissions

### Website Not Loading

**Check**:
1. Static website hosting enabled
2. Index document specified correctly
3. Public access enabled
4. Bucket policy allows GetObject

### Upload Failures

**Solutions**:
1. Check file size (max 5TB)
2. Use multipart upload for large files
3. Verify IAM permissions
4. Check network connectivity

---

## Key Takeaways

1. **S3 is globally accessible but regionally stored**
2. **11 nines reliability** - your data is safe
3. **Choose storage class** based on access patterns
4. **Use bucket policies** for security
5. **Enable versioning** for important files
6. **Perfect for static website hosting**
7. **Very cost-effective** for storage needs
8. **No limits** on storage amount
9. **Encryption enabled** by default
10. **Essential service** for any AWS architecture

---

## Your Assignment

### Practice Tasks

1. **Create S3 bucket** with proper naming convention
2. **Upload different file types** and explore interface
3. **Enable versioning** and upload same file multiple times
4. **Create IAM user** and test bucket policies
5. **Host a simple website** using static website hosting
6. **Explore different storage classes** and pricing
7. **Set up lifecycle policy** to move old files to cheaper storage

### Real-World Project

**Build a simple photo gallery**:
1. Create S3 bucket for images
2. Upload sample photos
3. Enable static website hosting
4. Create HTML page that displays images
5. Make it publicly accessible
6. Share the URL with friends

---

## Resources

- **GitHub Repository**: Complete code examples and policies
- **AWS S3 Documentation**: Comprehensive official guide
- **AWS Pricing Calculator**: Estimate costs for your use case
- **S3 Storage Classes Guide**: Detailed comparison

---

**Remember**: S3 is often the first AWS service people learn because it's simple yet powerful. Master these basics and you'll use S3 in almost every AWS project! 📦

*Next: We'll explore more AWS services and see how they integrate with S3!*