# Capstone-Project-Wordpress-site-on-AWS
Deploying a WordPress site on AWS. The project will make use of VPC, Internet Gateway, 2 AZs, public &amp; private subnets, NAT Gateway, MYSQL RDS, Amazon EFS, EFS Mount Targets, EC2 Instances, ALB, Autoscaling group, Route 53

Step by step of project execution:
1. Networking (This is configured in order to isolate and secure the WordPress infrastructure)
- I created a VPC on AWS and I named it 'Wordpress_VPC'.
- I created an internet gateway called 'Wordpress_IGW' and attached it to the 'wordpress_VPC'.
- I created a public subnet called 'Public_subnet1' within the 'Wordpress_VPC' at US East (Ohio) /us-east-2a Availability Zone.
- I created another public subnet called 'Public_subnet2' within the 'Wordpress_VPC' at US East (Ohio) /us-east-2b Availability Zone.
- I created 2 public subnets in different Availability zones because of high availability and fault tolerance.
- I created a private subnet called 'Private_subnet1' within the 'Wordpress_VPC' at US East (Ohio) /us-east-2a Availability Zone.
- I created a private subnet called 'Private_subnet2' within the 'Wordpress_VPC' at US East (Ohio) /us-east-2b Availability Zone.
- I created 2 private subnets in different Availability zones because of high availability and fault tolerance.
- I created a Route table in the 'Wordpress_VPC' for the public subnets called 'Public-subnet-RT'.
- I edited the Route table to add routes for internet access at 0.0.0.0/0 and I added the earlier created 'Wordpress_IGW' as the target.
- I edited the subnet association to associate the 'Public-subnet-RT' Route Table with the Public Subnets 1 & 2.
- I created another Route table in the 'Wordpress_VPC' for the private subnets called 'Private-subnet-RT'.
- I edited the subnet association to associate the 'Private-subnet-RT1' Route Table with the Private Subnets 1.
- I created another Route table and associated it with called 'private-subnet-RT2' and associated it with private subnet 2.
- I did not add a route for the internet access for the 'private-subnet-RT' because it will only access the internet from the NAT gateway.
- I created a NAT gateway called 'public-NAT1' in 'Public_subnet1' to connect 'Private_subnet1' to the internet. I created an elastic IP for this.
- I created another NAT gateway called 'public-NAT2' in 'Public_subnet2' to connect 'Private_subnet2' to the internet. I created an elastic IP for this.
- I associated the 'private-subnet-RT1' Route table with the 'Public-NAT1'.
- I associated the 'private-subnet-RT2' Route table with the 'Public-NAT2'.

2. Security Group Architecture for the 'Wordpress_VPC' (This is configured to ensure maximum security in access resources)
   - I created a Security group for the Application Load Balancer called 'ALB_Security_Group'. It allows inbound traffic from port HTTP 80 and HTTPS port 443.
   - I created a Security group for SSH called 'SSH_Security_Group'. It allows inbound traffic from port 22, source from my IP.
   - I created a Webserver Security group called 'Webserver_Security_Group'. It allows inbound traffic from ports HTTP 80 and HTTPS port 443, source from the Application Load Balancer security group. It also allows traffic from port 22, source SSH Security Group.  
   - I created a security group for the database called 'DB_Security_gp'. It allows inbound traffic from port 3306 for MYSQL/Aurora, source Webserver Security Group.
   - I created a security group for the EFS called 'EFS_Security_Group'. It allows inbound traffic from port 2049 for NFS, source Webserver Security Group. It also allows inbound traffic from port 22, source SSH Security Group.

  3. Database (This is configured to as a database storage for WordPress data storage)
   - I created an RDS MySQL database free tier in 'Wordpress_VPC' vpc under the US East (Ohio) /us-east-2a Availability Zone.
   - I added the database to the 'DB_Security_Group' and enabled it for public access.
   - I created 2 Ubuntu EC2 RDS instance for the database within the 'Wordpress_VPC' for 'Private_subnet1' and 'private_subnet2'.
   - I created a security group for the instance to allow SSH (port 22) from your IP (for administration), allow HTTP (port 80) and HTTPS (port 443) and allow MySQL (port 3306) from the RDS security group.
   - I found it difficult to SSH into the EC2 instance because it is in Private subnet so I created an IAM role 'AmazonSSMManagedInstanceCore" permissions to the EC2 Instance so I was able to connect from the session manager.
   - The instance was connected to the RDS MySQL Database.
   - I SSH into the instance using the Session Manager and performed the following.
     a. Installed Apache
     b. Installed PHP 7.4
     c. Installed MySQL 5.7
     d. Set permissions for Apache
     e. Downloaded WordPress files
     f. Created the wp-config.php file
     g. Edited the the wp-config.php file to add DB_NAME, USERNAME, PASSWORD AND HOST NAME
     h. Restarted the webserver
- Created another database for US East (Ohio) /us-east-2b Availability Zone for maximum availability and repeated the above steps in the Database section. I could have made the first RDS MySQL Database multi-AZ to save this configuration but doing that will incur more cost as multi-AZ configuration is beyond the scope of the free tier.
  
4. EFS setup for WordPress Files (This is configured to store WordPress files for scalable and shared access)
   - Created an EFS file system called 'Wordpress_EFS' within the 'Wordpress' VPC.
   - To mount the EFS on the EC2s I had to first install amazon-efs-utils on the EC2 instances.
   -  Created a file called 'efs' in home directory.
   -  Used the Mount via DNS to mount the Efs on the EC2 instances.
  
5. Application Load Balancer (This is configured to distribute incoming traffic among multiple instances to ensure high availability and fault tolerance)
   - I created an Application Load Balancer called 'Wordpress-ALB'.
   - Created it as multi-AZ for US East (Ohio) /us-east-2a Availability Zone and US East (Ohio) /us-east-2b Availability Zone.
   - Used the 'ALB_Security_Group' configured earlier.
   - Created a target group called 'Wordpress' that included all the EC2 instances for both AZs and associated it with the Application Load Balancer. 
   - Configured the listeners rule for routing traffic to the instances.
  
6. Auto Scaling Group (This is configured to automatically adjust the number of instances based on traffic load)
   - created an auto-scaling group named 'WordPress-AutoScaleG'.
   - I did not have a launch template so I created one named 'WordPress-temp'.
   - I used the AMI for my recently configured EC2 instance for the template.
   - I configured a Target tracking scaling policy for the auto-scaling group. Metric type = Average CPU utilization; Target value = 50; Instance warm-up = 300 seconds.

TESTING:
I tested my load balancer and auto scaling group by stopping my instances and checking if a new instances will be started and the test passed.
