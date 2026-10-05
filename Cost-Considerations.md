# Cost Considerations and Optimization Notes

This document outlines the primary cost drivers in the architecture and the
strategies used to manage and optimize costs.

---

## 1. Primary Cost Drivers

### Networking
- NAT Gateways (per Availability Zone)
- Transit Gateway attachments and data processing
- Gateway Load Balancer and Network Firewall endpoints
- Inter-AZ and cross-VPC traffic inspection

### Compute
- EC2 instances in Auto Scaling Groups
- Load Balancers (ALB)
- Amazon EFS throughput and storage usage

### Edge & Security
- CloudFront request and data transfer costs
- AWS WAF rule evaluations
- AWS Shield Advanced (if enabled)

### Data Services
- Amazon RDS Multi-AZ instance and storage costs
- Amazon ElastiCache node usage
- Backup storage via AWS Backup

---

## 2. Cost Optimization Strategies

- Auto Scaling Groups configured with minimum and maximum bounds
- VPC endpoints used to reduce NAT Gateway data processing costs
- Log retention and lifecycle policies applied to S3 and CloudWatch logs
- Instance types selected based on right-sizing and expected load
- Budgets and alerts configured using AWS Budgets and SNS

---

## 3. Intentional Design Choices

Certain components increase cost but were included intentionally:

- **Security VPC + Network Firewall**  
  Chosen to meet strict security and inspection requirements.

- **Multi-AZ RDS**  
  Prioritized availability and durability over single-AZ cost savings.

- **CloudFront + WAF**  
  Improves performance and security at the edge, reducing backend load.

---

## 4. Portfolio Deployment Notes

To control cost in a portfolio or lab environment:

- Expensive components (Transit Gateway, Network Firewall, GWLB) may be
  defined in Terraform but disabled by default using feature flags
- Smaller instance sizes are used for demonstration
- Some components may be validated via Terraform plan rather than
  continuous deployment

These decisions balance realism with responsible cost management.

---

## 5. Future Cost Improvements

Potential future optimizations include:

- Savings Plans or Reserved Instances for steady-state workloads
- Graviton-based instances for improved price-performance
- S3 + CloudFront for static assets to reduce EFS usage
- Scheduled scaling for predictable traffic patterns

---

## Author
Marwane Roman