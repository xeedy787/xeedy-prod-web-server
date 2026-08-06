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



    #CHALLENGES AND SOME FIX
1. SSH / SCP Key Pathing & Permission Handling
Problem: ssh and scp failed with Identity file not accessible and Permission denied (publickey).
Cause: Passing unquoted relative key paths caused the terminal to search the active working directory rather than the SSH directory.
Solution: Moved keys directly to ~/.ssh/, applied restrictive permissions (chmod 400 ~/.ssh/react-key.pem), and used absolute paths during transfer:
    scp -i ~/.ssh/react-key.pem -r /path/to/dist/* ubuntu@<EC2-IP>:/tmp/
   
2. Configuring Browser Behaviour from Network Connectivity
Problem: Web requests to the ec2 instance public IP timed out continuously in browser testing despite open Security Groups and active SSH sessions.
Diagnostics: Conducted low-level terminal diagnostics using curl from local WSL:
   curl -Iv http://<EC2-IP>
The terminal immediately returned HTTP/1.1 200 OK, proving the VPC, Internet Gateway, Route Tables, and Apache web server were functional.
Solution: Isolated the issue to Brave Browser's aggressive "Shields" security settings and also Chrome, which were silently forcing HTTP traffic onto Port 443 (HTTPS), where no SSL certificate was yet configured. Disabling HTTPS upgrades resolved the timeout.

3. Resolving Load Balancer 502 Bad Gateway Errors
Problem: Public requests to the Load Balancer DNS returned a 502 Bad Gateway error.
Cause: Traffic was arriving at the ALB, but the ALB could not hand off requests to backend EC2 instances due to Security Group isolation.
Solution: Configured the EC2 Instance Security Group to explicitly accept inbound Port 80 traffic sourced directly from the ALB's Security Group ID and verified Target Group health check responses returned Healthy.

Majorly configuring the browser to load the web page with my react and apache build gave the most challenging bug fix of all as it became so glaring and confusing as to why my public ip wouldnt render on the web while everything else checked out. Am still yet to find a full culprit to point to as the major reason for this behavior.


             # WHAT I LEARNED SO FAR
1. Difference between Launch Templates and AMIs and how AMIs are essential for launching applications faster.
2. How Auto Scaling Groups replace unhealthy instances.
3. How CloudWatch automates scaling decisions.
4. Why Target Groups are essential for load balancing.
5. How Apache serves/deploys React builds.
6. Why infrastructure troubleshooting requires validating each layer instead of assuming the problem is in one place.             


   # FUTURE IMPROVEMENTS
1. Deploy the application inside Docker.
2. Automate infrastructure with Terraform.
3. Add HTTPS using a certificate.
4. Point a custom domain to the load balancer.

         # PROJECT SCREENSHOTS
   Security group showing inbound rules
<img width="1920" height="1034" alt="Screenshot (181)" src="https://github.com/user-attachments/assets/cc8dbeec-7b07-4dd8-b214-150c134554fd" />

   EC2 instance dashboard
<img width="1920" height="1032" alt="Screenshot (180)" src="https://github.com/user-attachments/assets/b09aab9a-534d-42f0-a4dc-2e31b9a46de1" />

   Target Group for ASG AND ALB
<img width="1920" height="1032" alt="Screenshot (183)" src="https://github.com/user-attachments/assets/d238a60d-8bfc-4329-b80a-632254052b32" />

   User data script used within launch template
<img width="1920" height="1041" alt="Screenshot (182)" src="https://github.com/user-attachments/assets/61138319-6787-4812-9881-0d82522abec2" />

   CloudWatch Scaling Policy
<img width="1920" height="1030" alt="Screenshot (187)" src="https://github.com/user-attachments/assets/58b4fc97-feee-44fb-877a-815824808de3" />

   Custom AMI used for ec2 deployment
<img width="1920" height="1030" alt="Screenshot (186)" src="https://github.com/user-attachments/assets/e1aba280-6a95-4834-8aa8-f75bdc5102e9" />

   Auto-Scaling Group
<img width="1920" height="1036" alt="Screenshot (185)" src="https://github.com/user-attachments/assets/ac14dc58-8da7-4720-8a16-3ad2022cf87d" />

   Application Load-Balancer
<img width="1920" height="1028" alt="Screenshot (184)" src="https://github.com/user-attachments/assets/83b11d97-24c4-4d51-abec-dcc9bb0d5654" />






























                         
                                    
