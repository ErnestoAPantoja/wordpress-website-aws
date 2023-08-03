<h1>Deploying a Wordpress website on AWS</h1>

I have created a dynamic website using AWS services. The purpose behind this project is to build on what I have learned from the AWS Certified Cloud Practitioner certification and flesh out concepts I will be familiar with for future AWS projects and certification exams. I used a variety of different AWS services to build the website, and I took the website down after building it to save money. I spent about $30 to do this project over the span of a few days. I am using my Windows 10 computer in order to do this project and am using PuTTY to get familiar with using the program. 

_<b>NOTE:</b> This project is done exclusively in the N. Virginia region (us-east-1) in order to ensure all services are available. Not all services are available in multiple regions. If you want to replicate this project, research the services that are used and see if they are all available in a specific region._

<h2>Environments and Technologies Used</h2>

- AWS
- PuTTY
- VPC (Public and Private subnets)
- Security Groups
- EC2 instances
- Auto Scaling Groups
- NAT Gateways
- RDS
- Application Load Balancers
- Route 53
- Certificate Manager
- EFS

<h2>High-Level Deployment and Configuration Steps</h2>

- Setup a VPC.
- Setup NAT Gateways.
- Create security groups.
- Launch an RDS instance.
- Enable EFS.
- Create a key pair and install PuTTY
- Setup a server
- SSH instance
- Install WordPress
- Configure an Application Load Balancer
- Claim a domain name on Route 53
- Create route records on Route 53
- Obtain an SSL certificate
- Launch a bastion host
- SSH into private subnets
- HTTPS listener
- Create Auto Scaling Groups

<h2>Step Process</h2>

<h3>&#9312; Create a VPC</h3>

<p align="center">
<img src="https://i.imgur.com/Tqq0xAr.jpg" height="80%" width="80%" alt="VPC Reference"/>
</p>

- A three-tier VPC will serve as the architecture for the project. The first tier will have the public subnets. The public subnets will host resources such as NAT Gateways, Application Load Balancers, and eventually a bastion host. The second tier will host a private subnet. The web servers (EC2 instances ) will be hosted there. The third tier will have another private subnet which will host the database necessary to complete the project. The subnets will be duplicated across multiple availability zones to increase fault tolerance and high availability. An Internet Gateway and route table will also be created to allow resources in the VPC to access the internet.

- The VPC will be created in the N. Virginia region. From the AWS Management console, navigate to the VPC service. In the VPCs menu, click Create VPC.
  - Give a name to the VPC (in my case, it is Dev VPC) and enter the IPv4 CIDR block (10.0.0.0/16). Leave the rest of the settings as default and click Create VPC.

<p align="center">
<img src="https://i.imgur.com/4bpt43d.png" height="80%" width="80%" alt="Step 1-1"/>
</p>
