## AWS VPC High-Availability Reference

This repository captures a production-ready AWS network foundation that spans two Availability Zones (AZs) for resilience and fault tolerance. It combines properly segmented networking, managed load balancing, and elastic compute capacity so that downstream application teams can deploy services securely and confidently.

### Core Architecture
- **Virtual Private Cloud (VPC)** with RFC1918 CIDR, divided into dedicated public and private subnets across two AZs (four subnets total) to isolate ingress resources from application workloads.
- **Internet Gateway (IGW)** attached to the VPC to provide outbound connectivity from public subnets and to receive inbound traffic through the load balancer.
- **NAT Gateway** deployed in each public subnet to give private subnets egress to the internet for software updates while keeping them unreachable from the public internet.
- **Route Tables** that direct:
  - Public subnet traffic to the IGW.
  - Private subnet traffic to the NAT Gateway for outbound-only internet access.

### Application Layer
- **Application Load Balancer (ALB)** provisioned in the public subnets to terminate TLS, perform health checks, and distribute traffic evenly to backend targets.
- **Auto Scaling Group (ASG)** spanning the private subnets, ensuring a configurable minimum number of instances and scaling based on demand or health.
- **Target Group** registered with the ASG so only healthy instances receive traffic from the ALB.

### High Availability and Security
- Multi-AZ deployment prevents single-AZ failures from impacting availability.
- Separate public/private subnets enforce least privilege and limit the attack surface.
- Managed NAT Gateway centralizes outbound traffic control and simplifies security group rules.
- Security Groups and Network ACLs (not shown in code) should be scoped so that:
  - ALB accepts inbound traffic on required ports (e.g., 80/443) and forwards only to the application port.
  - Private instances allow ingress solely from the ALB or bastion hosts.

### Deployment Guidance
1. Customize CIDR blocks, AZs, and resource names to match your AWS account’s limits.
2. Provision networking primitives first (VPC, subnets, routing, IGW, NAT).
3. Create ALB, target group, listener, and security groups.
4. Launch the Auto Scaling Group with an appropriate launch template/launch configuration.
5. Validate:
   - ALB health checks succeed against all instances.
   - Instances in private subnets can reach the internet via the NAT Gateway.
   - Cross-AZ failover works by temporarily disabling an instance/AZ.

### Operations Checklist
- Enable AWS CloudWatch alarms for ALB 5xx rates, ASG capacity, and NAT Gateway usage.
- Capture infrastructure as code (Terraform/CDK/CloudFormation) for repeatability.
- Document standard operating procedures for scaling events and AZ outages.
- Regularly patch AMIs through immutable deployments or rolling updates.

### Next Steps
- Add diagrams (AWS Perspective/Draw.io) illustrating subnet layout and traffic flow.
- Layer in additional shared services (bastion hosts, centralized logging, VPC endpoints).
- Integrate CI/CD automation (CodePipeline, GitHub Actions) to provision infrastructure consistently.

### Reference Material
- This build follows the walkthrough in Abhishek Veeramalla’s “Ultimate End to End DevOps Project” tutorial (`https://www.youtube.com/watch?v=FZPTL_kNvXc&t=13s`), which demonstrates deploying a production VPC across two AZs with ALB, Auto Scaling, private subnets, and dual-AZ NAT gateways. Additional context, docs, and community links referenced in the video:
  - Course hub: `https://www.udemy.com/course/ultimate...`
  - Author resources and community: `https://github.com/iam-veeramalla/aws...`, `https://t.me/abhishekveeramalla`, `https://www.buymeacoffee.com/abhishekprd`, `https://topmate.io/abhishek_veeramalla`
  - AWS documentation baseline: `https://docs.aws.amazon.com/vpc/latest/...`

