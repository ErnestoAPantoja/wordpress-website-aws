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

<p align="center">
<img src="https://i.imgur.com/kFiYDfb.jpg" height="80%" width="80%" alt="Step 2"/>
</p>

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

<h3>&#9314; Create Security Groups</h3>

<p align="center">
<img src="https://i.imgur.com/yw8HU3r.jpg" height="80%" width="80%" alt="Step 3"/>
</p>

- The above image details all the security groups that need to be created to continue with the project. The Application Load Balancer will have a security group to allow internet traffic (HTTP and HTTPS). One security group will be dedicated to allow SSH access to EC2 instances using your IP address. (Any time an SSH security group is created, it is always best practice to limit the source to your IP address for safety.) A security group will be created for web servers in the Private App Subnets. The sources for this security group will be limited to the ALB and SSH security groups respectively. A security group will be created for the RDS database that will be hosted on the Private Data Subnets and the source will be from the Webserver security group. An EFS security group will be made for elastic file system and use previous security groups for the sources.
- On the AWS management console, navigate to the VPC service. On the VPC Dashboard, open the Security Groups tab. The first security group that will be created is the ALB Security Group. Click on Create security group to get started. Make sure the security group is in the Dev VPC. For Inbound rules, there will be two rules that will be added. For the Type, select HTTP and HTTPS. The Sources will come from Anywhere. To have this setting, input the CIDR block 0.0.0.0/0. Click Create security group to confirm the settings.

<p align="center">
<img src="https://i.imgur.com/RHjr9gP.png" height="80%" width="80%" alt="Step 3-1"/>
</p>

<p align="center">
<img src="https://i.imgur.com/Bafkoaa.png" height="80%" width="80%" alt="Step 3-2"/>
</p>

- Create the rest of the security groups with the following settings:
  - SSH Security Group - VPC: Dev VPC, Inbound rules: SSH, Source: My IP
  - Webserver Security Group - VPC: Dev VPC, Inbound rules: HTTP, Source: ALB Security Group, Inbound rules: HTTPS, Source: ALB Security Group, Inbound rules: SSH, Source: SSH Security Group.
  - Database Security Group - VPC: Dev VPC, Inbound rules: MySQL/Aurora, Source: Webserver Security Group.
  - EFS Security Group - VPC: Dev VPC, Inbound rules: NFS, Source: Webserver Security Group, Inbound rules: SSH, Source: SSH Security Group.
- After the EFS Security Group is created, click on Edit inbound rules to add one more important rule:
  - Add an additional NFS rule where the source is from the EFS Security Group. This rule could not be added unless the security group was already created.

<p align="center">
<img src="https://i.imgur.com/LF15HvK.png" height="80%" width="80%" alt="Step 3-3"/>
</p>

<h3>&#9315; Create the RDS Instance</h3>

<p align="center">
<img src="https://i.imgur.com/mx6xtMG.jpg" height="80%" width="80%" alt="Step 4"/>
</p>

- The next step is to create a RDS database in the Private Data Subnets. On the AWS management console, navigate to the RDS service to get started. Before creating the RDS instance, subnet groups need to be created. They specify which subnets the RDS database will be created in. Select Subnet groups on the RDS Dashboard and click Create DB subnet group.
  - Name the group database subnets and place it in the Dev VPC. Under the Add subnets section, select the us-east-1a and us-east-1b Availability Zones. For Subnets, select the subnets with the CIDR blocks 10.0.4.0/24 and 10.0.5.0/24. Click Create to make the subnet group.

<p align="center">
<img src="https://i.imgur.com/3N0vEt9.png" height="80%" width="80%" alt="Step 4-1"/>
</p>

- Now that the subnet group is created, it is time to make the database itself. Click on Databases on the left-hand menu and click on Create database. Use the following parameters to create the database:
  - Creation method: Standard create
  - Engine options: MySQL
  - Engine Version: MySQL 5.7 (The latest version of 5.7 as in the future, more updated versions will be released beyond when I created the website.)
  - Templates: Dev/Test
  - DB instance identifier: dev-rds-db
  - Master username: (Whatever you choose, in my case it is ernesto.)
  - Master password: (Whatever you choose, in my case it is Password1. Make sure you remember this password as there will be no way to retrieve it afterward.)
  - DB instance class: Burstable classes (db.t2.micro)
  - VPC: Dev VPC
  - Subnet group: database subnets
  - VPC security group: Choose existing (Database Security Group)
  - Availability Zone: us-east-1b
  - Database authentication: Password authentication
  - Initial database name: applicationdb (Make sure you expand Additional configuration to see this parameter, you must specify a name or else RDS will not make the database.)
- After the database is created (it will take a few minutes for AWS to create), click on the database indentifier name. Under the Connectivity & security tab, take note of the Endpoint of the database. This information will be used later when connecting to the database using an EC2 instance. Under the Configuration tab, take note of the DB instance ID and DB name as they will also be used to connect to the database.

<p align="center">
<img src="https://i.imgur.com/PGn58sg.png" height="80%" width="80%" alt="Step 4-2"/>
</p>

<p align="center">
<img src="https://i.imgur.com/IIeUG0w.png" height="80%" width="80%" alt="Step 4-3"/>
</p>

<h3>&#9316; Create the Elastic File System (EFS)</h3>

- Now that the RDS database is in place, it is time to create an EFS file system with mount targets in the Private Data Subnets in both Availability Zones. This is to ensure the web servers can have access to shared files.
- On the AWS management console, navigate to the EFS service and click Create file system and Customize. Use the following parameters to create the file system:
  - Name: Dev-EFS
  - Encryption: Check off Enable encryption of data at rest (This is to ensure we do not get charged for the encryption.)
  - Tag key: Name, Tag value: Dev-EFS
  - VPC: Dev VPC
  - Mount targets: us-east-1a, Private Data Subnet AZ1, EFS Security Group and us-east-1b, Private Data Subnet AZ2, EFS Security Group
  - File system policy: Leave everything as default

<p align="center">
<img src="https://i.imgur.com/8gXgWA4.png" height="80%" width="80%" alt="Step 5-1"/>
</p>

<p align="center">
<img src="https://i.imgur.com/T6798l6.png" height="80%" width="80%" alt="Step 5-2"/>
</p>

- Now that the elastic file system is created, click on the File system ID and click on Attach. This information will be used later in the project to mount the file system.

<p align="center">
<img src="https://i.imgur.com/9XEGzAk.png" height="80%" width="80%" alt="Step 5-3"/>
</p>

<p align="center">
<img src="https://i.imgur.com/2ISmlXF.png" height="80%" width="80%" alt="Step 5-4"/>
</p>

<h3>&#9317; Create a Key Pair</h3>

- A key pair will now have to be created in order to progress further with the project. On the AWS management console, navigate to the EC2 service. On the left-hand menu, click on Key Pairs and click Create key pair.
  - Name the key pair (in my case, myec2key) and make sure the Key pair type is RSA. The file format will be kept as .ppk because I will be using the key pair for use with PuTTY.

<p align="center">
<img src="https://i.imgur.com/NHsrLTe.png" height="80%" width="80%" alt="Step 6-1"/>
</p>

<p align="center">
<img src="https://i.imgur.com/iXgObty.png" height="80%" width="80%" alt="Step 6-2"/>
</p>

- When a key pair is made, two keys are generated: a public key and a private key. The key on the AWS console is the public key and it will be used in the EC2 instance when it is launched. The key that is downloaded on the computer is the private key and it will be used whenever SSH is used to access an instance.

<h3>&#9318; Launching a Setup Server</h3>

- An EC2 instance will be launched in Public Subnet AZ1 in order to install the website and move files to the EFS. On the AWS management console, navigate to the EC2 service and select Instances (running). Click on Launch instances to get started. Use the following parameters for the instance:
  - Name: Setup Server
  - Application and OS Images: Amazon Linux
  - AMI: Amazon Linux 2 AMI (Free tier eligible)
  - Instance type: t2.micro
  - Key pair (login): myec2key (the key pair that you created)
  - VPC: Dev VPC
  - Subnet: Public Subnet AZ1
  - Firewall (security groups): SSH Security Group, ALB Security Group, Webserver Security Group

<p align="center">
<img src="https://i.imgur.com/0ZSMbeb.png" height="80%" width="80%" alt="Step 7-1"/>
</p>

<p align="center">
<img src="https://i.imgur.com/X87q45d.png" height="80%" width="80%" alt="Step 7-2"/>
</p>

<p align="center">
<img src="https://i.imgur.com/ut8LC58.png" height="80%" width="80%" alt="Step 7-3"/>
</p>

<h3>&#9319; Accessing the Public Subnet EC2 Instance</h3>

- Because I am using a Windows computer, I will be using PuTTY to SSH into the instance that was created. While it is possible to not use PuTTY since I am using a Windows 10 computer, I will still use PuTTY for practice.
- To SSH into the instance, copy the instance's Public IPv4 address. Within the Session tab of PuTTY, enter the Host Name ec2-user@(Public IPv4 address). In the Connection tab, expand SSH and expand Auth. Select Credentials under the Auth tab. Enter the private key that was downloaded to the computer when the key pair was created earlier in the project. After you click Open, you will successfully access the EC2 instance.

<p align="center">
<img src="https://i.imgur.com/P3r8ZZR.png" height="80%" width="80%" alt="Step 8-1"/>
</p>

<p align="center">
<img src="https://i.imgur.com/0UaATYQ.png" height="80%" width="80%" alt="Step 8-2"/>
</p>

<p align="center">
<img src="https://i.imgur.com/a7V4UkB.png" height="80%" width="80%" alt="Step 8-3"/>
</p>

<h3>&#9320; Installing WordPress</h3>

- Once the EC2 instance has been accessed through SSH, commands will have to be run in order to install the WordPress website. Before continuing, make sure that the relevant EFS mount data has been copied from a previous step in the project. In the EFS that was created earlier, the Attach menu will show the code that is necessary to mount the EFS. Make sure to copy the highlighted section in the image below.

<p align="center">
<img src="https://i.imgur.com/snqtoNi.png" height="80%" width="80%" alt="Step 9-1"/>
</p>

- Within the PuTTY session, run the following commands (and make sure to place the EFS code where specified and remove the parentheses around it):
  - sudo su
  - yum update -y
  - mkdir -p /var/www/html
  - sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport (EFS code):/ /var/www/html

- Now that the EFS has been mounted, Apache will have to be installed. Run the following commands:
  - sudo yum install -y httpd httpd-tools mod_ssl
  - sudo systemctl enable httpd
  - sudo systemctl start httpd

- Next, PHP 7.4 will be installed with the following commands:
  - sudo amazon-linux-extras enable php7.4
  - sudo yum clean metadata
  - sudo yum install php php-common php-pear -y
  - sudo yum install php-{cgi,curl,mbstring,gd,mysqlnd,gettext,json,xml,fpm,intl,zip} -y

- MySQL 5.7 will be installed with these commands:
  - sudo rpm -Uvh https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
  - sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022
  - sudo yum install mysql-community-server -y
  - sudo systemctl enable mysqld
  - sudo systemctl start mysqld

- Some web files will need to have their permissions changed. Run these commands to set the permissions:
  - sudo usermod -a -G apache ec2-user
  - sudo chown -R ec2-user:apache /var/www
  - sudo chmod 2775 /var/www && find /var/www -type d -exec sudo chmod 2775 {} \;
  - sudo find /var/www -type f -exec sudo chmod 0664 {} \;
  - chown apache:apache -R /var/www/html

- The WordPress files will now be downloaded and moved to the html directory with the following commands:
  - wget https://wordpress.org/latest.tar.gz
  - tar -xzf latest.tar.gz
  - cp -r wordpress/* /var/www/html/
 
- A WordPress configuration file will have to be created and modified. Run these commands:
  - cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php
  - nano /var/www/html/wp-config.php

<p align="center">
<img src="https://i.imgur.com/oWHtG8G.png" height="80%" width="80%" alt="Step 9-2"/>
</p>

- Within the text editor for the configuration file, some information needs to be inserted from the RDS instance that was created earlier in the project. Go to the RDS console from AWS to get this information. In the database that was created, open the Configuration tab to get the necessary information.
  - Copy the DB name from the Configuration tab and replace it where database_name_here is.

_<b>NOTE:</b> Make sure to copy the DB name and NOT the DB instance ID. They refer to different things and are not the same thing. Make sure what you are copying is the DB name. Refer to the image below. The Database instance ID is highlighted here. DB name is located underneath it._

<p align="center">
<img src="https://i.imgur.com/ikK6jvP.png" height="80%" width="80%" alt="Step 9-3"/>
</p>

- The next things to change in the file are the username and password for the RDS database. Enter the master username and password for the database when it was created. Replace username_here and password_here respectively.
- The next thing to change is the database hostname in the file. The database hostname will be the endpoint of the RDS instance. Return to the RDS console and open the Connectivity & security. Copy the endpoint and replace localhost within the configuration file.

<p align="center">
<img src="https://i.imgur.com/SzI29kR.png" height="80%" width="80%" alt="Step 9-4"/>
</p>

- Now that the necessary information is inserted in the configuration file, the EC2 instance will now be able to connect to the RDS instance. Save all the changes and run the last command to restart the Apache web server:
  - service httpd restart
- Return to the EC2 console and copy the Public IPv4 address of the Setup Server. Open a new tab in the web browser and paste the IPv4 address. When everything has been configured correctly, a WordPress welcome page will be shown. Enter the necessary information to create the admin account and website. The Setup Server cannot be deleted yet as the next step is to create the application load balancer.

<p align="center">
<img src="https://i.imgur.com/TFawYpa.png" height="80%" width="80%" alt="Step 9-5"/>
</p>

<p align="center">
<img src="https://i.imgur.com/xW5Phri.png" height="80%" width="80%" alt="Step 9-6"/>
</p>

<h3>&#9321; Create the Application Load Balancer</h3>

- An application load balancer will be created to distribute web traffic across EC2 instances in the Private App Subnets in the VPC. Before creating the application load balancer, new EC2 instances will be launched in the Private App Subnets. Navigate to the EC2 service to get started. Launch an instance with the following configurations:
  - Name and Tags: Name, Webserver AZ1
  - Application and OS Images: Amazon Linux 2 AMI (free tier eligible)
  - Instance type: t2.micro
  - Key pair: myec2key (the key pair that you created earlier)
  - VPC: Dev VPC
  - Subnet: Private App Subnet AZ1
  - Firewall (security groups): Web Server Security Group
- For the user data, some commands will be pasted in. This means that the commands will be run whenever the instance is booting up. Before pasting the commands in the user data, return to the EFS console and obtain the mount data that was previously used to install WordPress earlier in the project.

<p align="center">
<img src="https://i.imgur.com/mnUdGeu.png" height="80%" width="80%" alt="Step 10-1"/>
</p>

- Paste the following script into the user data section of the EC2 instance creation menu (and replace the EFS data where specified):
  - #!/bin/bash
  - yum update -y
  - sudo yum install -y httpd httpd-tools mod_ssl
  - sudo systemctl enable httpd
  - sudo systemctl start httpd
  - sudo amazon-linux-extras enable php7.4
  - sudo yum clean metadata
  - sudo yum install php php-common php-pear -y
  - sudo yum install php-{cgi,curl,mbstring,gd,mysqlnd,gettext,json,xml,fpm,intl,zip} -y
  - sudo rpm -Uvh https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
  - sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022
  - sudo yum install mysql-community-server -y
  - sudo systemctl enable mysqld
  - sudo systemctl start mysqld
  - echo "(EFS data):/ /var/www/html nfs4 nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2 0 0" >> /etc/fstab
  - mount -a
  - chown apache:apache -R /var/www/html
  - sudo service httpd restart
 
<p align="center">
<img src="https://i.imgur.com/rWC269n.png" height="80%" width="80%" alt="Step 10-2"/>
</p>

- Launch a second EC2 instance while the first one is being made and use the following configurations:
  - Name and Tags: Key - Name, Value - Webserver AZ2
  - Application and OS Images: Amazon Linux 2 AMI (free tier eligible)
  - Instance type: t2.micro
  - Key pair: myec2key (the key pair that you created earlier)
  - VPC: Dev VPC
  - Subnet: Private App Subnet AZ2
  - Firewall (security groups): Web Server Security Group
  - User data: the same user data script that was used in the first instance

- After creating the two EC2 instances, the next step is to create the target group and put the instances in the target group to allow the application load balancer to route traffic to them. On the left-hand menu, open the Target Groups tab and click on Create target group. Use the following configurations to make the target group:
  - Target type: Instances
  - Name: Dev-TG
  - Protocol: HTTP
  - VPC: Dev VPC
  - Advanced health check settings - Success codes: 200,301,302
  - Register targets: Webserver AZ1 and Webserver AZ2 (click on Include as pending below to confirm the choices)

<p align="center">
<img src="https://i.imgur.com/NtCmQyg.png" height="80%" width="80%" alt="Step 10-3"/>
</p>

- The next step is to create the application load balancer. Select Load Balancers on the left-hand menu and click on Create load balancer. Use these configurations to create the application load balancer:
  - Load balancer name: Dev-ALB
  - Scheme: Internet-facing
  - IP address type: IPv4
  - VPC: Dev VPC
  - Mappings: us-east-1a - Public Subnet AZ, us-east-1b - Public Subnet AZ2
  - Security groups: ALB Security Group
  - Listener HTTP 80 Default Action: Forward to Dev-TG

- After the application load balancer is active, copy the DNS name and paste it in a new browser tab. The website can now be accessed using the DNS name of the application load balancer.

<p align="center">
<img src="https://i.imgur.com/D2plyij.png" height="80%" width="80%" alt="Step 10-4"/>
</p>

<p align="center">
<img src="https://i.imgur.com/vC2fNyf.png" height="80%" width="80%" alt="Step 10-5"/>
</p>

- Any time the address is changed, it is necessary to go into the WordPress settings as an admin and change the domain address there. Before accessing the settings, copy the domain name of the application load balancer. After the domain name, type /wp-admin and press Enter. You will be prompted to log in as the admin using the WordPress crendentials when the website was first made. Click on Settings and paste the domain address in the WordPress Address and Site Address boxes (remove the / at the end of the address if it is retained).

<p align="center">
<img src="https://i.imgur.com/NSlCbst.png" height="80%" width="80%" alt="Step 10-6"/>
</p>

<p align="center">
<img src="https://i.imgur.com/p3LzW2V.png" height="80%" width="80%" alt="Step 10-7"/>
</p>

- Now that the instances are launched in the Private App Subnets and the website can be accessed via the DNS name of the application load balancer, there is no need to have the Setup Server running. Terminate the Setup Server on the EC2 console.

<p align="center">
<img src="https://i.imgur.com/rVWN8te.png" height="80%" width="80%" alt="Step 10-8"/>
</p>

<h3>&#9322; Register a Domain Name</h3>

- A domain name will be registered with Route 53 to be used as the url for the WordPress website. This domain name will be used instead of the DNS name of the application load balancer. Navigate to the Route 53 service on AWS to get started. Click on Registered domains to get started.
  - I am registering ernestoawswebsitelab.com for the purposes of the project. It will cost $13 to register the domain name. Enter the contact information to complete the transaction and make sure privacy protection is enabled. Give at least 15 minutes for the domain name to be registered. It may take longer for the registration to go through, just be patient.
 
<p align="center">
<img src="https://i.imgur.com/axFpFkN.png" height="80%" width="80%" alt="Step 11-1"/>
</p>

<p align="center">
<img src="https://i.imgur.com/yaxmGAz.png" height="80%" width="80%" alt="Step 11-2"/>
</p>

<p align="center">
<img src="https://i.imgur.com/2pBLEFk.png" height="80%" width="80%" alt="Step 11-3"/>
</p>

<h3>&#9323; Create a Record Set</h3>

- After getting a registered domain name, a record set will be created in Route 53 to access the website with the domain name. Navigate to the Route 53 service and click on Hosted zones to get started. Select the domain name and click on Create record. Use the following configurations to create the record:
  - Record name: www
  - Record type: A
  - Toggle Alias next to Route Traffic to
  - Route Traffic to: Alias to Application and Classic Load Balancer
  - Region: US East (N. Virginia) [us-east-1]
  - Load balancer: The application load balancer created earlier

<p align="center">
<img src="https://i.imgur.com/NNt8evg.png" height="80%" width="80%" alt="Step 12-1"/>
</p>

- Now that the record set is made, the website can now be accessed using the domain name. Select the record that was created and copy the record name. Paste the record name into a new browser tab and the website will be accessed.

<p align="center">
<img src="https://i.imgur.com/RBD6WfA.png" height="80%" width="80%" alt="Step 12-2"/>
</p>

<p align="center">
<img src="https://i.imgur.com/ecG8l6L.png" height="80%" width="80%" alt="Step 12-3"/>
</p>

- Since the domain name has changed once again, it is time to update the WordPress URL settings to reflect this. Repeat the steps from updating the URL name after creating the application load balancer.

<p align="center">
<img src="https://i.imgur.com/IKBY67i.png" height="80%" width="80%" alt="Step 12-4"/>
</p>

<p align="center">
<img src="https://i.imgur.com/okdHD17.png" height="80%" width="80%" alt="Step 12-5"/>
</p>

<p align="center">
<img src="https://i.imgur.com/MfEgxox.png" height="80%" width="80%" alt="Step 12-6"/>
</p>

- The site URL will now be the domain name!

<h3>&#9324; Register an SSL Certificate</h3>

- SSL certificates are necessary to encrypt traffic between the web servers and web browser. This is a concept referred to as encryption in transit. All traffic from the website is currently not secure. The website will now have an appropriate SSL certificate using the Certificate Manager service on AWS. Request a public certificate from Certificate Manager to get started.

<p align="center">
<img src="https://i.imgur.com/Q5qIP9u.png" height="80%" width="80%" alt="Step 13-1"/>
</p>

- For domain names, enter the domain name that you have. Enter a second domain name and include the *. wildcard before the domain name again. Refer to the image below to see how to input the domain names. Make sure to select DNS validation and the RSA 2048 key algorithm before requesting the certificate. 

<p align="center">
<img src="https://i.imgur.com/jYpOVNq.png" height="80%" width="80%" alt="Step 13-2"/>
</p>

- When the certificate is pending validation, record sets need to be created in Route 53. This is to validate that the domain name belongs to the rightful owner. Click Create records in Route 53 and select the two domain names (this includes the wildcard that was created) to create the records. Wait a few minutes and refresh the page to see that the certificate has been issued.

<p align="center">
<img src="https://i.imgur.com/Iw6W4Px.png" height="80%" width="80%" alt="Step 13-3"/>
</p>

<p align="center">
<img src="https://i.imgur.com/lcyJQpC.png" height="80%" width="80%" alt="Step 13-4"/>
</p>

<p align="center">
<img src="https://i.imgur.com/08pcLbA.png" height="80%" width="80%" alt="Step 13-5"/>
</p>

<h3>&#9325; Launch a Bastion Host</h3>

- In order to SSH into the instances in the private subnets, an EC2 instance needs to be launched in a public subnet. This istance is called a bastion host. First, the instance in the public subnet needs to be accessed with SSH. From the public subnet instance, SSH into the private subnet. Navigate to the EC2 service and create a new instance to get started. Use the following configurations to make the bastion host:
  - Name: Bastion Host
  - Application and OS Images: Amazon Linux
  - Amazon Machine Image: Amazon Linux 2 AMI (free tier eligible)
  - Instance type: t2.micro
  - Key pair: myec2key (the key pair that was created earlier)
  - VPC: Dev VPC
  - Subnet: Public Subnet AZ1
  - Auto-assign Public IP: Enable
  - Firewall (security groups): SSH Security Group
 
<h3>&#9326; SSH into Private Subnets</h3>

