# Highly Available & Zero-Trust Cloud Infrastructure on AWS
...
cd ~/aws-resilient-enterprise-infrastructure

cat << 'EOF' > README.md
# Highly Available & Zero-Trust Cloud Infrastructure on AWS

A production-grade, multi-tier AWS architecture designed across multiple Availability Zones, featuring automated scaling, load balancing, strict subnet isolation, and zero-trust portless instance management.

---

## 🏛️ Architecture Overview

Security & Architectural Highlights :

 - Subnet Isolation: Six dedicated subnets across 2 Availability Zones separating Public Ingress, Private Compute, and Isolated Data tiers.

 - Security Group Chaining:

   *  ALB-SG permits public traffic only on Ports 80 & 443.

   *  App-SG permits HTTP traffic strictly originating from the ALB-SG ID.

   *  DB-SG permits MySQL (3306) and SSH (22) traffic strictly originating from the App-SG ID.

 - Zero-Trust Access Control: Inbound SSH (Port 22) is completely closed across all firewalls. Remote instance management is handled securely via AWS Systems Manager (SSM) Session Manager with IAM role-based authentication.
 
 - High Availability & Fault Tolerance: Configured an Application Load Balancer (ALB) distributing health-checked traffic to an Auto Scaling Group (ASG) spanning multiple availability zones.

 - Outbound Gateway Control: Compute nodes update dependencies via an Elastic IP-backed NAT Gateway in the public subnet; external inbound connections to private nodes are blocked at the routing layer.


Verification & Resilience Tests :

 - Multi-AZ Traffic Balancing: Verified round-robin routing across private EC2 instances by accessing the ALB DNS endpoint.

 - Dynamic Self-Healing: Simulated host failure by manually terminating an active EC2 instance; the ASG detected the unhealthy target and provisioned a replacement node with zero downtime.

 - Internal VPC TCP Handshake: Executed nc -zv 10.0.3.203 22 from the app tier terminal via SSM to confirm inter-subnet routing while external access remained completely blocked


                               [ Internet Users ]
                                       │
                              [ Internet Gateway ]
                                       │
┌──────────────────────────────────────▼──────────────────────────────────────┐
│ Custom VPC (10.0.0.0/16) - Multi-Availability Zone                          │
│                                                                             │
│  ┌───────────────────────────────┐        ┌───────────────────────────────┐ │
│  │ Public Subnet AZ-1a (10.0.1.0)│        │ Public Subnet AZ-1b (10.0.2.0)│ │
│  │  [ Application Load Balancer ]│────────┼[ Application Load Balancer ]  │ │
│  │  [ NAT Gateway (AZ-1a)       ]│        │                               │ │
│  └──────────────┬────────────────┘        └───────────────┬───────────────┘ │
│                 │                                         │                 │
│  ┌──────────────▼────────────────┐        ┌───────────────▼───────────────┐ │
│  │ Private App AZ-1a (10.0.11.0) │        │ Private App AZ-1b (10.0.12.0) │ │
│  │  [ Auto Scaling Group EC2    ]│◀───────┼[ Auto Scaling Group EC2    ]  │ │
│  │  (No Public IP Assigned)      │        │  (No Public IP Assigned)      │ │
│  └──────────────┬────────────────┘        └───────────────┬───────────────┘ │
│                 │ (Outbound updates via NAT)              │                 │
│  ┌──────────────▼────────────────┐        ┌───────────────▼───────────────┐ │
│  │ Private DB AZ-1a (10.0.21.0)  │        │ Private DB AZ-1b (10.0.22.0)  │ │
│  │  [ Isolated MySQL Database ]  │        │  [ Standby Replica / Subnet ] │ │
│  │  (Zero Direct Internet Access)│        │  (Zero Direct Internet Access)│ │
│  └───────────────────────────────┘        └───────────────────────────────┘ │
│                                                                             │
│  [ Zero-Trust Management ]: AWS SSM Session Manager (Port 22 Closed)        │
└─────────────────────────────────────────────────────────────────────────────┘
