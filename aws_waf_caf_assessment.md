# Design and Evaluate an AWS Solution Using the Well-Architected and Cloud Adoption Frameworks

## Task 1 – Existing Architecture & Risks

Current on-premises setup (my quick sketch in original-on-prem-architecture.png):

- 1 physical database server in the internal network
- Single data center, flat network

Main risks I see immediately:

- Single point of failure everywhere
- No auto-scaling, traffic spikes will kill the site
- Security groups basically don’t exist (everything open)
- Backups are manual and slow to restore
- No monitoring or centralized logging
- Costs are fixed even when traffic is low

## Task 2 – Well-Architected Framework Assessment

| Pillar                 | Observation (Current State)                                                     | Improvement Recommendation                                             | Supporting AWS Service / Feature                                                         |
| ---------------------- | ------------------------------------------------------------------------------- | ---------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| Operational Excellence | Everything is manual (deployments via RDP, no CI/CD, no infrastructure as code) | Automate deployments and treat infra as code                           | AWS CodePipeline + CodeDeploy, CloudFormation / CDK                                      |
| Security               | Security groups allow 0.0.0.0/0 on 80/443 and RDP/SSH from internet             | Principle of least privilege, encrypt everything                       | AWS IAM, Security Groups, NACLs, AWS Shield, KMS, WAF                                    |
| Reliability            | Single AZ, single DB instance, no failover                                      | Multi-AZ, automatic failover, backups & PITR                           | RDS Multi-AZ, Elastic Load Balancing, Auto Scaling Groups, S3 backup                     |
| Performance Efficiency | Fixed server sizing, no auto-scaling, DB on same box sometimes                  | Right-size dynamically, use cache, separate read replicas              | EC2 Auto Scaling, ElastiCache (Redis), RDS Read Replicas                                 |
| Cost Optimization      | Paying for 24/7 hardware even at 2 am, over-provisioned most of the time        | Shut down/stop instances when unused, Reserved Instances/Savings Plans | AWS Cost Explorer and budgets, Savings Plans, stop idle instances, Spot for non-critical |

## Task 3 – AWS Cloud Adoption Framework (CAF) Readiness

### Business Perspective

Management loves the idea of “cloud = cheaper and faster”, but they don’t have clear business goals yet. They just said “move everything to AWS to save money”. No one calculated actual TCO or defined success metrics (e.g., 99.9 % uptime or 30 % cost reduction). We need a proper business case with ROI numbers and tie cloud migration to actual outcomes (faster feature releases, ability to handle Black Friday traffic). Right now it feels like migration for migration’s sake.

### People Perspective

Most of the on-prem team has never touched AWS console. Two devs know Docker, one sysadmin did the old AWS free tier years ago. Big fear of “losing control”. We definitely need AWS training (I already did Cloud Practitioner). Recommend creating a Cloud Center of Excellence with 3-4 people who become the cloud champions and train everyone else.

### Governance Perspective

No cloud governance at all right now. We don’t have tagging strategy, cost allocation, or IAM policies. If we just give everyone admin, it will be chaos. Need to set up AWS Organizations + SCPs, enforce MFA, create landing zone with Account Factory, and mandatory tagging for cost allocation (environment, owner, project).

### Platform Perspective

We picked a simple lift-and-shift first (Task 1–2), but long term we want to modernize: containerize the app and maybe go serverless for some parts. Need to decide reference architecture early. I suggest starting with ECS Fargate or EKS because the devs already like Docker. Also need standardized AMIs, Golden Images, or Container repos in ECR.

### Security Perspective

Security team is paranoid about “public cloud”. They think everything on internet is hacked instantly. Need to show them shared responsibility model and that AWS is often more secure than our on-prem. Actions: enable GuardDuty, Config rules, Security Hub, centralize logs in CloudTrail + CloudWatch. Also implement detective controls and regular penetration tests.

### Operations Perspective

Currently we get called at 3 am when a server dies. In AWS we can do way better. Set up CloudWatch alarms, automated remediation with Lambda, predictive scaling, and proper runbooks. Move to “you build it, you run it” culture so devs own their services in production.

## Task 4 – Improved AWS Architecture

(see architecture-diagram.png)

Key changes I made:

- VPC with public and private subnets across 2 AZs
- Internet-facing Application Load Balancer → Auto Scaling Group of EC2 instances (or Fargate tasks later)
- Web tier in private subnets, only talks outbound to internet for updates
- RDS MySQL in Multi-AZ with automated backups and read replica
- S3 bucket for static assets
- All traffic HTTPS only (ACM certificates)
- IAM roles instead of keys, least-privilege policies

Cost-wise: will use Savings Plans for steady state, Spot for dev/test, auto-stop dev environments at night.

## Reflection

This lab forced me to actually think like an architect instead of just clicking buttons. The Well-Architected Framework table was super useful, every time I wrote a weakness I immediately knew which service fixes it. CAF part made me realize migration fails because of people and processes, not technology. Drawing the new diagram took the longest because I kept adding “one more best practice” and refering to researching other diagrams. Now I understand why AWS pushes Multi-AZ and Auto Scaling so hard, one AZ going down on-prem kills everything. I’m definitely finishing the Solutions Architect Associate cert after this. Also, doing the exercise in lucidchart and writing markdown felt very “real job”.
