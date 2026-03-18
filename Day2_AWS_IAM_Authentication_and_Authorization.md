# Day 2: AWS IAM - Authentication and Authorization

## Quick Recap - Day 1
- Cloud computing concept
- Private vs Public Cloud
- Why public cloud is important
- Why AWS is a leading provider

---

## Today's Topics
1. What is AWS IAM?
2. Why do we need IAM?
3. IAM Components (Users, Groups, Policies, Roles)
4. Hands-on Practice

---

## Real-Life Example: Bank Security

### Bank Structure
- **Service Desk Area**: Customer queries
- **Employee Desk Area**: Staff workspace
- **Sensitive Area**: Documents, money, checks

### Bank Security Process
1. **Authentication**: Only account holders can enter
2. **Authorization**: Different access levels for different people
   - Customers → Service desk only
   - Employees → Employee area + service desk
   - Managers → All areas including sensitive zone

**Key Point**: Without authentication and authorization, anyone could rob the bank!

---

## AWS Without IAM (The Problem)

### Scenario
- DevOps engineer creates AWS account for company
- AWS has multiple services:
  - EC2 (Compute)
  - RDS (Database)
  - S3 (Storage)
  - EKS (Kubernetes)

### Without IAM
- Everyone gets root access
- Anyone can delete anything
- No tracking of who did what
- Major security risk
- Accidental deletions possible

**Example**: Employee unknowingly deletes entire database → Company data lost forever!

---

## AWS IAM Solution

### What is IAM?
**IAM = Identity and Access Management**

A service that handles:
- **Authentication**: Who can enter AWS account
- **Authorization**: What they can do inside

### How It Works
1. DevOps engineer creates AWS account (root access)
2. Root access NOT shared with anyone
3. New employee (501) requests access
4. DevOps creates IAM user for employee 501
5. Grants specific permissions (e.g., read DB, access Kubernetes)
6. Employee 501 logs in with IAM credentials
7. Can only do what permissions allow

---

## IAM Components

### 1. Users (Authentication)
- **Purpose**: Allow people to enter AWS
- **How**: Create username and password
- Each person gets their own IAM user
- Never share root account

**Example**: Create user "test-user-501" for new employee

### 2. Policies (Authorization)
- **Purpose**: Define what users can do
- **Format**: JSON documents
- **Types**:
  - AWS Managed Policies (pre-built by AWS)
  - Custom Policies (you create your own)

**Example Permissions**:
- S3 Read-Only Access
- S3 Full Access
- EC2 Full Access

### 3. Groups (Efficiency)
- **Purpose**: Organize users by role
- **Benefit**: Attach policies once to group, not each user

**Example Groups**:
- Developers
- QA Engineers
- DB Admins
- Others

**Why Groups Matter**:
- Without groups: Add permission to 501, 502, 503, 504 individually (4 times)
- With groups: Add permission to "Developers" group once (all 4 get it)

### 4. Roles (Service-to-Service Access)
- **Purpose**: Grant temporary access
- **Use Cases**:
  - Application outside AWS needs to access AWS services
  - One AWS account talking to another AWS account
  - Services within AWS talking to each other

**Key Difference from Users**:
- Users = For people
- Roles = For applications/services
- Roles have temporary credentials

**Note**: Roles covered in detail in future lessons with CI/CD and Terraform

---

## Hands-On Practice Summary

### Step 1: Create IAM User (Authentication Only)
1. Login as root user
2. Go to IAM service
3. Create user: "test-user-501"
4. Enable console access
5. Use auto-generated password
6. Require password reset on first login
7. Don't attach any policies yet

**Result**: User can login but cannot do anything

### Step 2: Test Authentication Without Authorization
1. Login as test-user-501
2. Try to access S3 buckets
3. **Error**: "You don't have permissions"
4. Try to access EC2
5. **Error**: "You don't have permissions"

**Lesson**: Authentication alone is not enough!

### Step 3: Add Policies (Authorization)
1. Login as root user
2. Go to IAM → Users → test-user-501
3. Click "Add permissions"
4. Attach policy: "AmazonS3FullAccess"
5. Save changes

**Result**: User can now list, create, and delete S3 buckets

### Step 4: Test Authorization
1. Login as test-user-501
2. Go to S3 service
3. Can now see all buckets
4. Can create new bucket
5. Can delete buckets

**Lesson**: Policies grant specific permissions!

### Step 5: Create Groups (Better Organization)
1. Login as root user
2. Go to IAM → User Groups
3. Create group: "Development"
4. Attach policy: "AmazonS3FullAccess"
5. Add users to group: test-user-501

**Future Benefit**:
- Need to add EC2 access for all developers?
- Just add policy to "Development" group
- All members automatically get access

---

## Policy Structure (JSON Format)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:*",
      "Resource": "*"
    }
  ]
}
```

### Components
- **Version**: Policy language version
- **Statement**: Array of permissions
- **Effect**: "Allow" or "Deny"
- **Action**: What operations (s3:*, ec2:*, etc.)
- **Resource**: Which resources (* = all)

**Note**: Custom policies covered in future lessons

---

## Best Practices

### 1. Never Use Root Account
- Create IAM user even for yourself
- Grant admin permissions to your IAM user
- Keep root account secure

### 2. Why Not Root?
- Can't track who did what
- Too much power
- Security risk
- No accountability

### 3. Use IAM Users Instead
- Each person has their own credentials
- Track all actions
- Limit permissions as needed
- Easy to revoke access

### 4. Organize with Groups
- Create groups by role/department
- Attach policies to groups
- Add users to appropriate groups
- Saves time and reduces errors

### 5. Follow Least Privilege
- Give minimum permissions needed
- Add more permissions when required
- Don't give full access by default

---

## Important Interview Points

**Q: Do you use root account?**
**A**: "No, we never use root account. As DevOps engineers, we have IAM users with admin permissions. We create IAM users for all team members and use groups to manage permissions efficiently."

**Q: How do you manage AWS access?**
**A**: "We use IAM service to create users, organize them into groups based on roles (developers, QA, etc.), and attach appropriate policies. This ensures proper authentication and authorization."

---

## Your Assignment

### Task 1: Create IAM User
1. Login with root account
2. Go to IAM service
3. Create user: "test-user" (or any name)
4. Enable console access
5. Use auto-generated password
6. Don't attach policies yet

### Task 2: Test Without Permissions
1. Login as new IAM user
2. Try accessing S3
3. Verify you get permission errors

### Task 3: Add S3 Permissions
1. Login as root
2. Attach "AmazonS3FullAccess" to your IAM user
3. Login as IAM user again
4. Verify you can now access S3
5. Create a test bucket

### Task 4: Practice with Groups
1. Create a group (e.g., "Developers")
2. Attach S3 policy to group
3. Add your IAM user to group
4. Test access

---

## Key Takeaways

1. **IAM = Security for AWS accounts**
2. **Users = Authentication (who can enter)**
3. **Policies = Authorization (what they can do)**
4. **Groups = Efficiency (manage multiple users easily)**
5. **Roles = For services/applications (covered later)**
6. **Never share root account credentials**
7. **Always use IAM users for daily work**

---

## Common Errors and Solutions

### Error: "You don't have permissions"
**Solution**: Attach appropriate policy to user or group

### Error: "Invalid credentials"
**Solution**: Check username, password, and account ID

### Error: "Password must be reset"
**Solution**: Normal for first login with auto-generated password

---

## Next Steps
- Practice creating users and groups
- Experiment with different policies
- Prepare for Day 3 topics
- Keep your IAM user credentials safe

## Resources
- AWS IAM Documentation
- AWS Managed Policies List
- IAM Best Practices Guide