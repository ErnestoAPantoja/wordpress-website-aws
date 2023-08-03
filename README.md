<h1>Deploying a WordPress website on AWS</h1>

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
- Setup NAT gateways.
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

- Next, DNS host names have to be enabled for the VPC that was created. Under Actions, select Edit VPC settings. Under DNS settings, make sure Enable DNS resolution and Enable DNS hostnames are checked and save the changes.

<p align="center">
<img src="https://i.imgur.com/RXp9haj.png" height="80%" width="80%" alt="Step 1-2"/>
</p>

<p align="center">
<img src="https://i.imgur.com/nnqQFcZ.png" height="80%" width="80%" alt="Step 1-3"/>
</p>

- An internet gateway will now be created for the VPC. On the left-hand menu, select Internet Gateways. Click Create internet gateway.
  - Give a name for the internet gateway (in my case, it is Dev Internet Gateway) and create it.

<p align="center">
<img src="https://i.imgur.com/P984xtj.png" height="80%" width="80%" alt="Step 1-4"/>
</p>

- After creating the internet gateway, it will have to be attached to the VPC. This is to ensure the VPC can communicate with the internet. There will be an option that says to Attach to a VPC after the internet gateway has been created.
  - One thing to note is that you can only attach one internet gateway to one VPC. When you go to attach an internet gateway to a VPC on AWS, you can only select VPCs that do not have internet gateways.

<p align="center">
<img src="https://i.imgur.com/VaRicio.png" height="80%" width="80%" alt="Step 1-5"/>
</p>

- Now that the internet gateway is attached to the VPC, public subnets will be created in two availability zones (us-east-1a and us-east-1b).
  - Select the Subnets tab on the left-hand menu. Click Create subnet. When creating your public subnets, make sure the Dev VPC is selected. For the first public subnet, name it Public Subnet AZ1 and make sure it is in the us-east-1a availability zone. Its IPv4 CIDR block should be 10.0.0.0/24. For the second public subnet, name it Public Subnet AZ2 and make sure it is in the us-east-1b availability zone. Its IPv4 CIDR block should be 10.0.1.0/24.

<p align="center">
<img src="https://i.imgur.com/1QhrXhb.png" height="80%" width="80%" alt="Step 1-6"/>
</p>

<p align="center">
<img src="https://i.imgur.com/toddnWF.png" height="80%" width="80%" alt="Step 1-7"/>
</p>

- After the public subnets are created, the auto enable IP settings need to be enabled for both subnets. This means when an EC2 instance is launched in the subnets, the instances will be assigned an appropriate public IP address in order to communicate with the internet.
  - For each subnet, select them and click on Edit subnet settings. Make sure Enable auto-assign public IPv4 address is turned on for both subnets and save the changes.

<p align="center">
<img src="https://i.imgur.com/YJbkxaN.png" height="80%" width="80%" alt="Step 1-8"/>
</p>

<p align="center">
<img src="https://i.imgur.com/TxhpCUJ.png" height="80%" width="80%" alt="Step 1-9"/>
</p>

- A public route table will now be created.
  - Select the Route Tables tab on the left-hand menu. A route table was already created when the VPC was made. This is referred to as the main route table and is private by default. Click Create route table and name the new route table Public Route Table. It will be attached to the Dev VPC.
 
<p align="center">
<img src="https://i.imgur.com/s1gIgpk.png" height="80%" width="80%" alt="Step 1-10"/>
</p>

- A public route will be added to the route table that was made. This public route will route traffic to the internet.
  - Under the Routes tab for the Public Route Table, click Edit Routes. Add a new route where the Destination is 0.0.0.0/0 (this means all traffic) and the Target is the Dev Internet Gateway. Save the changes.

<p align="center">
<img src="https://i.imgur.com/5Nt9aoP.png" height="80%" width="80%" alt="Step 1-11"/>
</p>

<p align="center">
<img src="https://i.imgur.com/wuOursD.png" height="80%" width="80%" alt="Step 1-12"/>
</p>

- The next thing that needs to be done is to associate the public subnets with the public route table.
  - While under the menu for Public Route Table, open the Subnet associations tab and scroll to Explicit subnet associations. Click on Edit subnet associations. Select both public subnets and save the associations.

 <p align="center">
<img src="https://i.imgur.com/0csGYLF.png" height="80%" width="80%" alt="Step 1-13"/>
</p>

 <p align="center">
<img src="https://i.imgur.com/0zVDZug.png" height="80%" width="80%" alt="Step 1-14"/>
</p>

- In order to finish creating the VPC, the four private subnets need to be created.
  - On the left-hand menu, click on Subnets and create the private subnets for the VPC. When creating your private subnets, make sure the Dev VPC is selected. For the first private subnet, name it Private App Subnet AZ1 and make sure it is in the us-east-1a availability zone. Its IPv4 CIDR block should be 10.0.2.0/24. For the second private subnet, name it Private App Subnet AZ2 and make sure it is in the us-east-1b availability zone. Its IPv4 CIDR block should be 10.0.3.0/24. For the third private subnet, name it Private Data Subnet AZ1 and make sure it is in the us-east-1a availability zone. Its IPv4 CIDR block should be 10.0.4.0/24. For the fourth private subnet, name it Private Data Subnet AZ2 and make sure it is in the us-east-1b availability zone. Its IPv4 CIDR block should be 10.0.5.0/24.

<p align="center">
<img src="https://i.imgur.com/t5sHdIT.png" height="80%" width="80%" alt="Step 1-15"/>
</p>

<p align="center">
<img src="https://i.imgur.com/Frc068s.png" height="80%" width="80%" alt="Step 1-16"/>
</p>

<p align="center">
<img src="https://i.imgur.com/6UgkdLh.png" height="80%" width="80%" alt="Step 1-17"/>
</p>

<p align="center">
<img src="https://i.imgur.com/3wbbJrt.png" height="80%" width="80%" alt="Step 1-18"/>
</p>

- Before you continue, make sure all 6 subnets are in the correct Availability Zones. The project will rely heavily on all the subnets and all resources and data will flow across the VPC.

_<b>NOTE:</b> When you create a route to a route table, all the subnets associated within the route table will automatically become public. Subnets are private when the route table does not have a route. In the Route Tables tab, check each route table to confirm each subnet are where they belong as shown below. The private subnets should be in the main route table that was automatically created when the VPC was first made. This is because these subnets do not have explicit associations, unlike the public subnets. The main route table is routing traffic locally within the VPC._

<p align="center">
<img src="https://i.imgur.com/8wTlXJy.png" height="80%" width="80%" alt="Step 1-19"/>
</p>

<h3>&#9313; Create NAT gateways</h3>

- Two NAT gateways will be created within the first and second Availability Zones. One will be in Public Subnet AZ1 and will be tied to a new private route table via a route that will connect the two together. The route table will also be associated with the Private App Subnet AZ1 and Private Data Subnet AZ1 subnets within the VPC. The second NAT gateway wil be created in Public Subnet AZ1 and tied to a new private route table with a route. The second route table will be associated with the Private App Subnet AZ2 and Private Data Subnet AZ2 subnets within the VPC.
- On the AWS management console, navigate to the VPC service. Select NAT Gateways on the VPC Dashboard. Create the first NAT gateway in Public Subnet AZ1. Name it NAT Gateway AZ1. Make sure to click Allocate Elastic IP before creating the NAT gateway.

<p align="center">
<img src="https://i.imgur.com/xy6mj0E.png" height="80%" width="80%" alt="Step 2-1"/>
</p>

- Now that the NAT gateway is created, a private route table and the appropriate route will be created so there will be access to the internet. Call this new route table Private Route Table AZ1 and put it in the Dev VPC. For the route, make sure the Destination is 0.0.0.0/0 and the Target is NAT Gateway AZ1.

<p align="center">
<img src="https://i.imgur.com/ZB8sq4W.png" height="80%" width="80%" alt="Step 2-2"/>
</p>

<p align="center">
<img src="https://i.imgur.com/GsrBCwU.png" height="80%" width="80%" alt="Step 2-3"/>
</p>

- The next step is to associate the route table with Private App Subnet AZ1 and Private Data Subnet AZ1. In Private Route Table AZ1, open the Subnet associations tab and click on Edit subnet associations. Select Private App Subnet AZ1 and Private Data Subnet AZ1 and save the associations.

<p align="center">
<img src="https://i.imgur.com/VNPTmid.png" height="80%" width="80%" alt="Step 2-4"/>
</p>

- Repeat the previous steps in order to create a NAT gateway in Public Subnet AZ2.
  - Name the second NAT gateway NAT Gateway AZ2.
  - Name the second route table Private Route Table AZ2 and put it in the Dev VPC.
  - Add a route where the Destination is 0.0.0.0/0 and the Target is NAT Gateway AZ2.
  - Associate the route table with Private App Subnet AZ2 and Private Data Subnet AZ2.
