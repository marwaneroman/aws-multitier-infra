# AWS Secure Multi-Tier Architecture – Design Notes

## 1. Requirements
This architecture was designed to meet the following requirements:

- Public-facing web application with global access
- High availability across multiple Availability Zones
- Strict network isolation with application and data tiers in private subnets
- Centralized traffic inspection and edge protection
- Secure secrets management and encrypted data at rest
- Scalable application tiers with minimal single points of failure

---

## 2. High-Level Architecture Overview
User traffic is resolved via Amazon Route 53 and delivered through Amazon CloudFront.
AWS Shield and AWS WAF provide edge-level DDoS and application-layer protection.

Traffic is forwarded to an Application Load Balancer and routed through a dedicated
Security VPC, where AWS Network Firewall performs centralized ingress and egress
inspection using a Gateway Load Balancer and Transit Gateway.

The Workload VPC hosts the application stack across two Availability Zones. Stateless
web and application tiers run in private subnets using Auto Scaling Groups. Shared
storage is provided via Amazon EFS, while Amazon ElastiCache is used to reduce database
load. The data tier uses Amazon RDS in a Multi-AZ configuration.

---

## 3. Key Design Decisions

- **Security VPC separation**  
  A dedicated Security VPC isolates inspection components and allows centralized
  control of traffic for current and future workload VPCs.

- **Defense in depth**  
  Security controls are layered across edge (CloudFront, WAF, Shield), network
  (Network Firewall), and application (security groups, private subnets).

- **Multi-AZ high availability**  
  Web, application, and database tiers span multiple Availability Zones to tolerate
  AZ-level failures.

- **Stateless compute with Auto Scaling**  
  Web and application tiers are stateless, enabling horizontal scaling and easier
  recovery.

- **Managed services where possible**  
  RDS Multi-AZ, ElastiCache, Secrets Manager, and AWS Backup reduce operational burden
  and improve reliability.

---

## 4. Traffic Flow Summary

1. Client resolves domain via Route 53
2. Request passes through CloudFront, Shield, and WAF
3. Traffic reaches the Application Load Balancer
4. Traffic is inspected via the Security VPC (GWLB + Network Firewall)
5. Requests are routed to web and application Auto Scaling Groups
6. Application accesses EFS, ElastiCache, and RDS as needed
7. Responses return through the same controlled path

---

## 5. Operations & Observability

- CloudWatch metrics and alarms for ALB errors, Auto Scaling events, and RDS failovers
- CloudTrail enabled for API auditing
- Centralized logging to Amazon S3 with lifecycle policies
- AWS Backup used for automated backup and retention
- Budget alerts configured using AWS Budgets and SNS

---

## 6. Trade-offs and Limitations

- Increased cost due to NAT Gateways, Transit Gateway, and firewall endpoints
- Higher operational complexity compared to single-VPC architectures
- Slight additional latency due to centralized traffic inspection

These trade-offs were accepted to prioritize security, isolation, and scalability.

---

## 7. Simplified Alternative

For smaller teams or lower-risk workloads, this architecture could be simplified by:

- Removing the Security VPC and Transit Gateway
- Using AWS WAF and security groups for traffic control
- Hosting all tiers within a single VPC

This reduces cost and operational overhead while maintaining reasonable security.

## Author
Marwane Roman