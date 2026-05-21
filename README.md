# Enterprise Multi-Region Disaster Recovery Network on AWS with Transit Gateway Peering and Private S3 Endpoints

## Project Overview

This project demonstrates the design and implementation of a secure multi-region disaster recovery architecture on AWS using private networking, Transit Gateway (TGW) inter-region peering, and S3 Gateway Endpoints.

The environment simulates a real-world enterprise cloud networking architecture where workloads communicate privately across AWS regions without relying on public internet exposure.

The project was built across:

- us-east-1 → Primary Production Region
- us-west-2 → Disaster Recovery (DR) Region

The architecture includes:

- Multi-region VPCs
- Public and private subnet segmentation
- Bastion host access pattern
- Private EC2 workloads
- Transit Gateway inter-region peering
- Cross-region private routing
- Regional S3 Gateway Endpoints
- IAM role-based authentication
- Secure S3 uploads without NAT gateways

---

# Real-World Use Case

Organizations often require highly available disaster recovery environments across multiple AWS regions while maintaining secure private connectivity between workloads and AWS services.

This project demonstrates how enterprises can:

- Maintain private connectivity between regions
- Isolate workloads in private subnets
- Access S3 securely without internet/NAT dependency
- Use AWS backbone networking for inter-region traffic
- Implement scalable cloud routing architecture
- Prepare for future disaster recovery enhancements

---

# Architecture Diagram

Upload architecture image later inside:

/architecture/enterprise-multi-region-dr-network.png

---

# Architecture Flow

## Cross-Region Private Networking Flow

```text
Client Admin
    │
    ▼
East Bastion Host (Public Subnet)
    │
    ▼
East Private EC2
    │
    ▼
East Private Route Table
    │
    ▼
TGW-East
    │
    ▼
Inter-Region TGW Peering
    │
    ▼
TGW-West
    │
    ▼
West Private Route Table
    │
    ▼
West Private EC2
```

---

# Private S3 Endpoint Flow

```text
East Private EC2
    │
    ▼
S3 Gateway Endpoint (East)
    │
    ▼
East S3 Bucket

West Private EC2
    │
    ▼
S3 Gateway Endpoint (West)
    │
    ▼
West DR S3 Bucket
```

---

# AWS Services Used

- Amazon VPC
- Transit Gateway (TGW)
- TGW Inter-Region Peering
- EC2
- IAM Roles
- Amazon S3
- S3 Gateway Endpoints
- Route Tables
- Internet Gateway
- Security Groups

---

# Environment Design

## Primary Region — us-east-1

### Components

- East-App-VPC
- Public Subnet
- Private Subnet
- East Bastion Host
- East Private EC2
- TGW-East
- East S3 Gateway Endpoint
- Primary S3 Bucket

---

## Disaster Recovery Region — us-west-2

### Components

- West-DR-VPC
- Public Subnet
- Private Subnet
- West Bastion Host
- West Private EC2
- TGW-West
- West S3 Gateway Endpoint
- DR S3 Bucket

---

# Step-by-Step Implementation

## Step 1 — Create Multi-Region VPCs

Created separate VPCs in two AWS regions using non-overlapping CIDR ranges.

### Primary Region

```text
10.10.0.0/16
```

### DR Region

```text
10.20.0.0/16
```

### Why

Using non-overlapping CIDRs is required for successful inter-region routing through Transit Gateway peering.

---

## Step 2 — Create Public and Private Subnets

Each VPC was segmented into:

- Public subnet
- Private subnet

### Why

This simulates real enterprise network segmentation where workloads remain isolated inside private subnets while bastion hosts provide administrative access.

---

## Step 3 — Configure Internet Gateways

Internet Gateways were attached to both VPCs.

### Why

Public subnets required internet connectivity for bastion host administration while private workloads remained isolated.

---

## Step 4 — Configure Route Tables

Separate public and private route tables were configured.

### Public Route Tables

```text
0.0.0.0/0 → Internet Gateway
```

### Private Route Tables

Initially contained only local VPC routing until Transit Gateway routes were added.

### Why

This ensures workloads in private subnets remain inaccessible from the internet.

---

## Step 5 — Deploy Bastion Hosts

Bastion hosts were launched in public subnets within both regions.

### Why

Private EC2 instances had no public IP addresses, so bastion hosts were required for secure SSH access.

---

## Step 6 — Deploy Private EC2 Instances

Private EC2 workloads were launched inside private subnets.

### Security Design

- No public IP addresses
- Accessible only through bastion hosts
- IAM role-based AWS access

### Why

This follows enterprise security best practices by minimizing direct internet exposure.

---

## Step 7 — Configure Security Groups

Security groups were configured to allow:

- SSH from bastion hosts
- ICMP traffic between regions
- Internal workload communication

### Why

This enabled secure administrative access and cross-region connectivity testing.

---

## Step 8 — Deploy Transit Gateways

Transit Gateways were created in:

- us-east-1
- us-west-2

### Why

Transit Gateway acts as a centralized cloud router for scalable enterprise networking architectures.

---

## Step 9 — Attach VPCs to Transit Gateways

Private subnets were attached to local Transit Gateways.

### Why

This enabled private workloads to communicate through centralized routing infrastructure.

---

## Step 10 — Configure Inter-Region TGW Peering

Transit Gateway peering was established between:

```text
TGW-East ↔ TGW-West
```

### Why

This enabled private communication between workloads across AWS regions using the AWS global backbone network.

---

## Step 11 — Configure Private Route Tables

Routes were added to private route tables.

### East Private Route Table

```text
10.20.0.0/16 → TGW-East
```

### West Private Route Table

```text
10.10.0.0/16 → TGW-West
```

### Why

Private subnets required routing awareness for remote-region traffic.

---

## Step 12 — Configure TGW Static Routes

Static routes were added to Transit Gateway route tables.

### TGW-East

```text
10.20.0.0/16 → TGW Peering Attachment
```

### TGW-West

```text
10.10.0.0/16 → TGW Peering Attachment
```

### Why

Transit Gateways required explicit routing instructions for forwarding inter-region traffic through the peering attachment.

---

## Step 13 — Create S3 Buckets

Created:

- Primary S3 bucket in us-east-1
- DR S3 bucket in us-west-2

### Why

This simulated regional application storage and disaster recovery storage architecture.

---

## Step 14 — Configure IAM Role for EC2

An IAM role with S3 permissions was attached to private EC2 instances.

### Why

This allowed EC2 instances to securely access S3 without storing static AWS access keys.

---

## Step 15 — Create S3 Gateway Endpoints

S3 Gateway Endpoints were deployed in both regions.

### Why

Gateway Endpoints enabled private S3 communication without requiring:

- NAT Gateway
- Internet Gateway
- Public internet routing

---

# Validation and Testing

## Test 1 — Cross-Region Private Connectivity

### Command

```bash
ping 10.20.2.x
```

### Result

Successful ICMP responses confirmed:

- TGW peering functionality
- Correct route table configuration
- Successful private inter-region communication

---

## Test 2 — Validate Private S3 Access

### Command

```bash
aws s3 ls
```

### Result

Successfully listed S3 buckets from private EC2 instances using:

- IAM role authentication
- S3 Gateway Endpoints

without internet exposure.

---

## Test 3 — Upload Files to S3 from Private EC2

### East Region Upload

```bash
echo "Private S3 access from East EC2 via Gateway Endpoint" > test-east.txt

aws s3 cp test-east.txt s3://basit-east-primary-dr-bucket-2026/
```

### West Region Upload

```bash
echo "Private S3 access from West DR EC2 via Gateway Endpoint" > test-west.txt

aws s3 cp test-west.txt s3://basit-west-dr-bucket-2026/
```

### Result

Successfully uploaded files privately through S3 Gateway Endpoints without NAT or public internet dependency.

---

# Security Design Highlights

- Private EC2 instances had no public IP addresses
- Bastion-only administrative access
- IAM role authentication instead of access keys
- Private S3 communication through Gateway Endpoints
- No NAT Gateway dependency for S3 traffic
- AWS backbone traffic between regions
- Security group segmentation

---

# Key Concepts Learned

- Multi-region AWS networking
- Transit Gateway architecture
- TGW inter-region peering
- Enterprise route table design
- Private cloud networking
- S3 Gateway Endpoints
- Bastion host architecture
- IAM role authentication
- AWS backbone routing
- Disaster recovery architecture design

---

# Lessons Learned

- Transit Gateway route tables require explicit static routes
- S3 Gateway Endpoints are regional resources
- Gateway Endpoints do not traverse TGW peering
- Bastion hosts simplify private infrastructure administration
- IAM roles are preferred over static access keys
- Private routing requires configuration at both VPC and TGW levels
- AWS backbone networking enables secure inter-region communication

---

# Future Improvements

- Enable S3 Cross-Region Replication (CRR)
- Add Route 53 health checks and failover
- Implement Auto Scaling Groups
- Add centralized CloudWatch monitoring
- Integrate AWS Network Firewall
- Add Site-to-Site VPN or Direct Connect
- Implement centralized logging architecture

---

# Resume Skills Demonstrated

- Multi-Region AWS Networking
- Transit Gateway Architecture
- Inter-Region TGW Peering
- Private Cloud Networking
- S3 Gateway Endpoint Architecture
- IAM Role Authentication
- Bastion Host Design
- Enterprise Route Table Design
- Disaster Recovery Architecture
- AWS Infrastructure Troubleshooting
- Secure AWS Service Communication

---

# Cleanup Performed

All AWS resources were deleted after testing to avoid ongoing billing charges.

Resources removed:

- EC2 instances
- Transit Gateways
- TGW attachments
- TGW peering attachments
- S3 buckets
- S3 Gateway Endpoints
- IAM roles
- Route tables
- Security groups
- Internet Gateways
- Subnets
- VPCs

---

# Final Outcome

Successfully built a secure enterprise-style multi-region AWS disaster recovery network architecture using Transit Gateway peering and private S3 connectivity without NAT or public internet dependency.


