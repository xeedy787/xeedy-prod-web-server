AWS Production React Deployment

This project demonstrates the deployment of a production-ready React application on AWS using Amazon EC2, Apache, an Application Load Balancer, Auto Scaling Groups, Launch Templates, a Custom baked AMI and CloudWatch scaling policies. The goal was to simulate a highly available production environment while learning core AWS infrastructure concepts.

[aws arc sketch.txt](https://github.com/user-attachments/files/30782252/aws.arc.sketch.txt)
                               [ Public Internet ]
                                       │
                                       ▼
                       [ Application Load Balancer (ALB) ]
                         (Inbound: Port 80 from 0.0.0.0/0)
                                       │
                 ┌─────────────────────┴─────────────────────┐
                 │                                           │
                 ▼                                           ▼
      [ Public Subnet / AZ-1 ]                    [ Public Subnet / AZ-2 ]
  ┌───────────────────────────────┐           ┌───────────────────────────────┐
  │   Auto Scaling Group (ASG)    │           │   Auto Scaling Group (ASG)    │
  │  ┌─────────────────────────┐  │           │  ┌─────────────────────────┐  │
  │  │   EC2 (Custom AMI v1)   │  │           │  │   EC2 (Custom AMI v1)   │  │
  │  │  ┌───────────────────┐  │  │           │  │  ┌───────────────────┐  │  │
  │  │  │ Apache2 Web Server│  │  │           │  │  │ Apache2 Web Server│  │  │
  │  │  │ (React Build Files│  │  │           │  │  │ (React Build Files│  │  │
  │  │  └───────────────────┘  │  │           │  │  └───────────────────┘  │  │
  │  └─────────────────────────┘  │           │  └────────    ─────────────┘  │
  └───────────────────────────────┘           └───────────────────────────────┘
                 │                                           │
                 └─────────────────────┬─────────────────────┘
                                       │
                       Cloudwatch Dynamic Scaling Policy 
                         (Target tracking: CPU > 50%)


                         # TECHS STACK & TOOLS

| Service                   | Purpose                                |
| ------------------------- | -------------------------------------- |
| EC2                       | Host Ubuntu web servers                |
| Apache                    | Serve the React production build       |
| Application Load Balancer   Distribute incoming traffic            |
| Target Groups             | Route requests to healthy instances    |
| Auto Scaling Group        | Automatically manage EC2 capacity      |
| Launch Template           | Standardize EC2 configuration          |
| Custom AMI                | Rapidly launch preconfigured instances |
| CloudWatch                | Monitor CPU and trigger scaling        |
| IAM                       | Instance permissions                   |
| Security Groups           | Control inbound and outbound traffic   |
| EBS                       | Persistent storage for EC2             |



           # PROJECT WORKFLOW
User

↓

Application Load Balancer

↓

Target Group

↓

EC2

↓

Apache

↓

React Build


                 # DEPLOYMENT STROLL

Created a VPC and networking resources.
Launched an Ubuntu EC2 instance from a launch template.
Installed Apache2, node, Docker, using User Data script from the launch template .
Copied the production build of my React applicatio to /var/www/html.
Configured Apache.
Created a custom AMI.
Configured the Launch Template to version 2 to make changes now from the custom AMI.
Created an Auto Scaling Group.
Attached an Application Load Balancer and Target group.
Configured CloudWatch scaling policies.









                         
                                    
