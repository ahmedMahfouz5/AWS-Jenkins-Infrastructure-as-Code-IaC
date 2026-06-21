# AWS Jenkins Infrastructure as Code (IaC) Project

## Project Overview

**AWS Jenkins Infrastructure as Code** is a comprehensive Infrastructure as Code (IaC) solution that automates the provisioning and configuration of a complete CI/CD infrastructure on Amazon Web Services (AWS). This project leverages HashiCorp Terraform to define, deploy, and manage AWS resources in a reproducible, version-controlled manner.

The project demonstrates DevOps best practices by providing a fully automated, modular, and scalable infrastructure that deploys a Jenkins server pre-configured with Terraform, enabling immediate CI/CD pipeline development.

---

## Architecture & Components

### 1. **Networking Module** (`networking/`)
- **VPC (Virtual Private Cloud)**: Isolated network environment with CIDR block `10.0.0.0/16`
- **Public Subnets**: Two availability zones (`us-east-1a`, `us-east-1b`) for high availability
  - Subnet 1: `10.0.1.0/24`
  - Subnet 2: `10.0.2.0/24`
- **Internet Gateway**: Enables external communication for public resources
- **Route Tables**: Public route table with routes to the Internet Gateway for direct internet access
- **Outputs**: VPC ID, public subnet IDs, and CIDR blocks

### 2. **Security Groups Module** (`security-groups/`)
- **EC2 General Security Group**: 
  - Inbound: SSH (22), HTTP (80), HTTPS (443)
  - Outbound: Unrestricted traffic
  - Purpose: Allow secure remote access and web communication
- **Jenkins-Specific Security Group**:
  - Inbound: Port 8080 (Jenkins web interface)
  - Purpose: Enable Jenkins dashboard access from anywhere

### 3. **Jenkins Module** (`jenkins/`)
- **EC2 Instance Configuration**:
  - Instance Type: `t2.medium` (suitable for Jenkins workloads)
  - AMI: Amazon Linux 2023 (customizable via variables)
  - Public IP: Enabled for direct internet access
- **Key Management**: SSH key pair for secure access
- **User Data Script**: Automated installation of:
  - Java Development Kit (OpenJDK 17)
  - Jenkins (latest stable version)
  - Terraform (latest stable version)
- **Security**: IMDSv2 enforced for enhanced EC2 metadata security

### 4. **Remote State Management** (`remote_backend_s3.tf`)
- **S3 Backend**: Terraform state stored in S3 bucket (`jenkins-infra-9-7-2025`)
- **Benefits**: 
  - Centralized state management for team collaboration
  - State locking to prevent concurrent modifications
  - Version history and backup capabilities

---

## Key Features

✅ **Fully Automated Deployment**: One-command infrastructure provisioning  
✅ **Modular Design**: Reusable, independent components  
✅ **High Availability**: Multi-AZ subnet configuration  
✅ **Security Best Practices**: IMDSv2, security groups with principle of least privilege  
✅ **Jenkins Pre-configured**: Terraform and Java pre-installed for immediate use  
✅ **Remote State Management**: S3 backend for team collaboration  
✅ **Infrastructure as Code**: Version-controlled, auditable infrastructure  
✅ **Scalable**: Easily extensible for additional resources  

---

## Project Structure

```
.
├── main.tf                          # Root module orchestrating all components
├── provider.tf                      # AWS provider configuration
├── variables.tf                     # Input variable definitions
├── terraform.tfvars                 # Variable values
├── remote_backend_s3.tf             # S3 backend configuration
├── networking/
│   └── main.tf                      # VPC, subnets, IGW, route tables
├── security-groups/
│   └── main.tf                      # Security group definitions
├── jenkins/
│   └── main.tf                      # EC2 instance and key pair configuration
├── jenkins-template/
│   └── template.sh                  # User data script for automation
└── README.md                        # This file
```

---

## Prerequisites

Before deploying this infrastructure, ensure you have:

1. **AWS Account**: Active AWS account with appropriate permissions
2. **AWS Credentials**: Configured in `~/.aws/credentials` with profile `terraform`
3. **Terraform**: Version 6.0 or higher of AWS provider
4. **AWS CLI**: (Optional) For manual verification
5. **SSH Key**: Public SSH key for EC2 access

### AWS IAM Permissions Required:
- EC2: `DescribeInstances`, `RunInstances`, `TerminateInstances`
- VPC: `CreateVpc`, `CreateSubnet`, `CreateInternetGateway`, `CreateSecurityGroup`
- S3: `GetBucketVersioning`, `PutObject`, `GetObject` (for state management)
- IAM: `CreateKeyPair`, `DeleteKeyPair`

---

## Configuration

### 1. Update Variables (`terraform.tfvars`)

```hcl
# Network Configuration
vpc_cidr             = "10.0.0.0/16"          # Modify for different CIDR
vpc_name             = "us-east-1-vpc"
cidr_public_subnet   = ["10.0.1.0/24", "10.0.2.0/24"]
us_availability_zone = ["us-east-1a", "us-east-1b"]

# EC2 Configuration
ec2_ami_id = "ami-020cba7c55df1f615"  # Amazon Linux 2023 in us-east-1
public_key = "ssh-ed25519 AAAA..."    # Your public SSH key
```

**Note**: Different AWS regions require different AMI IDs. Update `ec2_ami_id` accordingly.

### 2. AWS Provider Configuration (`provider.tf`)

```hcl
# Ensure credentials file exists at ~/.aws/credentials
# Profile: terraform
# Region: us-east-1 (customizable)
```

---

## Deployment Instructions

### Step 1: Clone the Repository
```bash
git clone <repository-url>
cd project-1
```

### Step 2: Initialize Terraform
```bash
terraform init
```
This command:
- Downloads provider plugins
- Initializes the S3 backend
- Creates the `.terraform` directory

### Step 3: Validate Configuration
```bash
terraform validate
```
Checks syntax and configuration validity.

### Step 4: Preview Changes (Plan)
```bash
terraform plan -out=tfplan
```
Review the resources to be created. Output includes:
- VPC and subnets
- Security groups
- EC2 instance
- Key pair

### Step 5: Apply Configuration
```bash
terraform apply tfplan
```
Provisioning typically takes 5-10 minutes. Upon completion:
- Jenkins public IP is displayed
- Access Jenkins at `http://<EC2_Public_IP>:8080`

### Step 6: Access Jenkins

1. **Get Initial Admin Password**:
```bash
ssh -i your-key.pem ec2-user@<EC2_Public_IP>
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

2. **Navigate to Jenkins Dashboard**:
   - Open browser: `http://<EC2_Public_IP>:8080`
   - Enter the admin password
   - Complete Jenkins setup wizard
   - Install recommended plugins

---

## Useful Commands

```bash
# Show infrastructure state
terraform show

# Destroy all resources
terraform destroy

# Output specific values
terraform output jenkins_ec2_instance_ip
terraform output dev_proj_1_ec2_instance_public_ip

# State management
terraform state list          # List all resources
terraform state show aws_instance.jenkins_ec2_instance_ip

# Refresh state
terraform refresh
```

---

## Security Considerations

### ⚠️ Current Security Posture (Not Production-Ready)

1. **Overly Permissive Security Groups**: 
   - SSH allowed from `0.0.0.0/0` (anywhere)
   - Jenkins UI exposed to `0.0.0.0/0`
   - **Recommendation**: Restrict to your IP or VPN range

2. **Public EC2 Instance**:
   - Instance directly accessible from internet
   - **Recommendation**: Use bastion host or private subnets with VPN

3. **S3 Backend Bucket**:
   - Ensure versioning enabled
   - Enable bucket encryption
   - Block public access

### 🔐 Production Hardening Recommendations

```hcl
# Restrict SSH to specific CIDR
ingress {
  from_port   = 22
  to_port     = 22
  protocol    = "tcp"
  cidr_blocks = ["YOUR_IP/32"]  # Replace with your IP
}

# Use private subnets with NAT gateway
# Implement VPN for secure access
# Enable VPC Flow Logs for monitoring
# Use AWS Secrets Manager for credentials
# Implement backup policies for Terraform state
```

---

## Cost Estimation

Based on AWS us-east-1 pricing (approximate monthly):

| Resource | Cost |
|----------|------|
| t2.medium EC2 | $30-40 |
| Data transfer | $0.09/GB out |
| S3 storage (state) | <$1 |
| **Total** | **~$30-50/month** |

Enable AWS Budget alerts to monitor costs.

---

## Troubleshooting

### Issue: Terraform state lock error
```
Error: resource temporarily unavailable
```
**Solution**: State might be locked by another user. Check S3 bucket for lock file.

### Issue: EC2 instance fails to launch
```
Error: InvalidAMIID
```
**Solution**: AMI ID may not exist in your region. Update `ec2_ami_id` in terraform.tfvars.

### Issue: Jenkins not accessible
**Solution**: 
1. Verify security group allows port 8080
2. Wait 5 minutes for Jenkins service to start
3. Check EC2 instance logs: `sudo systemctl status jenkins`

### Issue: SSH connection refused
**Solution**:
1. Verify security group allows port 22
2. Confirm correct EC2 key pair name: `aws_ec2_terraform`
3. Use correct AMI user (`ec2-user` for Amazon Linux)

---

## Contributing & Improvements

Potential enhancements:
- Add RDS database layer
- Implement Docker registry on Jenkins
- Add CloudFront CDN
- Implement monitoring with CloudWatch
- Add auto-scaling groups
- Migrate to Terraform Cloud for team collaboration

---

## License

This project is open source and available under the MIT License.

---

## Author

**Ahmed Mahfouz** - DevOps Engineer  
Created: 2025

---

## Support

For issues or questions, please open an issue in the repository or contact the project maintainer.

---

## Changelog

### v1.0 - Initial Release
- ✅ Modular Terraform structure
- ✅ AWS VPC with public subnets
- ✅ Security groups for web and Jenkins access
- ✅ Automated Jenkins deployment
- ✅ S3 remote state management
- ✅ Pre-configured with Terraform on EC2