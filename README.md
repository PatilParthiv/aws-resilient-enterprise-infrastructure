# Highly Available & Zero-Trust Cloud Infrastructure on AWS

A production-grade, multi-tier AWS architecture designed across multiple Availability Zones, featuring automated scaling, load balancing, strict subnet isolation, and zero-trust portless instance management.
<img width="861" height="656" alt="Screenshot 2026-08-29 101700" src="https://github.com/user-attachments/assets/c7a88ebc-0dab-441f-8d1d-b27e3daf7a69" />



Architecture Overview :

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

<img width="664" height="710" alt="image" src="https://github.com/user-attachments/assets/8710c3f7-f5e6-456a-874b-aefc2188cd41" />

