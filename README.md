# WordPress Deployment with Terraform on AWS

## Project Overview

This project automates the deployment of a fully functional WordPress website on AWS using Terraform and cloud-init. The infrastructure is provisioned as code, making it reproducible, scalable, and easy to maintain.

## Architecture

The deployment consists of the following AWS components:

- **VPC (Virtual Private Cloud)**: Isolated network environment
- **Public & Private Subnets**: Segmented network architecture
- **Internet Gateway**: Allows public subnet resources to access the internet
- **NAT Gateway**: Enables private subnet resources to access the internet securely
- **Route Tables**: Manages traffic routing between subnets
- **Security Groups**: Firewall rules controlling inbound/outbound traffic
- **EC2 Instance**: Hosts the WordPress application
- **Cloud-init**: Automates the installation and configuration of WordPress, Apache, and MySQL

## File Structure

```
.
├── main.tf              # Core infrastructure resources (VPC, EC2, networking)
├── variables.tf         # Input variables for customization
├── outputs.tf           # Output values (public IP, instance ID, etc.)
├── terraform.tfvars     # Variable values (instance type, region, etc.)
└── cloud-init.yaml      # Automation script for WordPress setup
```

## What Each File Does

### `main.tf`
Defines the main infrastructure components:
- VPC with CIDR block
- Public and private subnets
- Internet Gateway and NAT Gateway
- Route tables and associations
- Security groups (HTTP, HTTPS, SSH)
- EC2 instance with cloud-init configuration

### `variables.tf`
Declares configurable parameters such as:
- AWS region
- Instance type
- VPC CIDR blocks
- SSH key name
- AMI ID

### `terraform.tfvars`
Contains actual values for the variables (not committed to Git for security)

### `outputs.tf`
Exports useful information after deployment:
- Public IP address of WordPress site
- Instance ID
- VPC ID

### `cloud-init.yaml`
Automates the server setup:
- Updates system packages
- Installs Apache web server
- Installs and configures MySQL database
- Installs PHP and required extensions
- Downloads and configures WordPress
- Sets proper permissions
- Starts services automatically

## How It Works

1. **Infrastructure Provisioning**: Terraform reads the configuration files and creates the AWS resources in the correct order
2. **Network Setup**: VPC, subnets, and gateways are created to establish secure networking
3. **EC2 Launch**: An EC2 instance is launched with the cloud-init script embedded
4. **Automated Configuration**: Cloud-init runs on first boot, installing and configuring WordPress, Apache, and MySQL without manual intervention
5. **Service Ready**: WordPress is accessible via the public IP address within minutes

## Why This Approach is Useful

### Infrastructure as Code (IaC)
- **Reproducibility**: Deploy identical environments in minutes
- **Version Control**: Track infrastructure changes like application code
- **Documentation**: The code itself documents the infrastructure

### Automation Benefits
- **Speed**: Deployment takes 5-10 minutes vs hours of manual setup
- **Consistency**: Eliminates human error in configuration
- **Scalability**: Easy to deploy multiple instances or environments

### Cloud-Init Advantages
- **No Manual SSH**: Server configures itself automatically
- **Idempotent**: Can be run multiple times safely
- **Flexibility**: Easy to modify and update the setup process

## Challenges Faced & Solutions

### 1. Cloud-Init Script Debugging
**Challenge**: Cloud-init runs at boot time, making it difficult to debug when things fail silently.

**Solution**: 
- Added logging to `/var/log/cloud-init-output.log`
- Used `terraform output` to get the public IP and SSH in to check logs
- Tested the script locally before embedding in Terraform

### 2. Security Group Configuration
**Challenge**: Initially locked myself out by not properly configuring SSH access in security groups.

**Solution**: 
- Created separate security group rules for SSH (port 22), HTTP (port 80), and HTTPS (port 443)
- Used CIDR blocks carefully to balance security and accessibility
- Learned to always include my IP for SSH access during development

### 3. NAT Gateway Costs
**Challenge**: NAT Gateways are expensive (~$32/month) and I initially left one running accidentally.

**Solution**: 
- Used `terraform destroy` to clean up resources after testing
- Learned to check AWS console for orphaned resources
- Considered using a NAT instance for dev environments to save costs

### 4. MySQL Root Password Security
**Challenge**: Hardcoding the MySQL root password in cloud-init is insecure.

**Solution**: 
- Used Terraform variables to pass sensitive data
- Marked variables as `sensitive = true`
- Learned about AWS Secrets Manager for production deployments

### 5. Terraform State Management
**Challenge**: Lost my state file once and couldn't manage existing infrastructure.

**Solution**: 
- Learned the importance of state file backups
- Researched remote state storage with S3 and DynamoDB for locking
- Used `.gitignore` to never commit state files

### 6. WordPress Database Connection Issues
**Challenge**: WordPress couldn't connect to MySQL immediately after launch.

**Solution**: 
- Added `sleep` commands in cloud-init to wait for MySQL to fully start
- Configured MySQL to bind to localhost properly
- Ensured WordPress database user had correct permissions

## Skills Acquired

### Technical Skills
- **Terraform**: Writing and managing infrastructure as code
- **AWS Networking**: Understanding VPCs, subnets, route tables, and gateways
- **Security Groups**: Configuring firewall rules and network security
- **Cloud-Init**: Automating server configuration and application deployment
- **Linux Administration**: Managing services, permissions, and package installation
- **LAMP Stack**: Setting up Linux, Apache, MySQL, and PHP environments

### DevOps Practices
- **Infrastructure as Code**: Treating infrastructure like application code
- **Automation**: Reducing manual work and human error
- **Version Control**: Managing infrastructure changes systematically
- **Problem Solving**: Debugging issues in automated deployments
- **Cost Awareness**: Understanding AWS pricing and resource management

### Soft Skills
- **Documentation**: Writing clear technical documentation
- **Troubleshooting**: Systematic approach to identifying and fixing issues
- **Patience**: Dealing with trial-and-error in infrastructure automation
- **Research**: Finding solutions in documentation and community forums

## Deployment Instructions

1. **Prerequisites**:
   - AWS account with appropriate permissions
   - Terraform installed (v1.0+)
   - AWS CLI configured with credentials

2. **Configure Variables**:
   ```bash
   cp terraform.tfvars.example terraform.tfvars
   # Edit terraform.tfvars with your values
   ```

3. **Initialize Terraform**:
   ```bash
   terraform init
   ```

4. **Review Plan**:
   ```bash
   terraform plan
   ```

5. **Deploy**:
   ```bash
   terraform apply
   ```

6. **Get WordPress URL**:
   ```bash
   terraform output wordpress_url
   ```

7. **Clean Up**:
   ```bash
   terraform destroy
   ```

## Future Improvements

- Implement remote state storage with S3 and DynamoDB
- Add Application Load Balancer for high availability
- Use RDS for managed MySQL instead of local installation
- Implement auto-scaling groups for traffic spikes
- Add CloudWatch monitoring and alarms
- Use AWS Secrets Manager for sensitive data
- Implement multi-AZ deployment for redundancy
- Add SSL/TLS certificates with ACM and Route53

## Lessons Learned

1. **Always use version control** for Terraform configurations
2. **Test cloud-init scripts** locally before deploying
3. **Use variables** for everything that might change
4. **Mark sensitive data** appropriately in Terraform
5. **Clean up resources** immediately after testing to avoid costs
6. **Document as you go** - don't wait until the end
7. **Security groups are critical** - get them right from the start
8. **Infrastructure as Code is powerful** but requires discipline and practice

## Conclusion

This project demonstrated the power of Infrastructure as Code and automation in modern cloud deployments. By combining Terraform's infrastructure provisioning with cloud-init's configuration management, I was able to deploy a complete WordPress environment in minutes rather than hours. The skills acquired here are directly applicable to real-world DevOps and cloud engineering roles, and the challenges faced provided valuable learning experiences in troubleshooting, security, and cost management.

---

**Author**: Roody Adams  
**Date**: November 2025  
**Technologies**: Terraform, AWS, Cloud-Init, WordPress, Apache, MySQL