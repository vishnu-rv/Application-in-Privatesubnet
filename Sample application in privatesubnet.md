# Deploying an Application in Private Subnet with AWS ALB

## Overview
This document outlines the process of deploying an application in a **private subnet** while making it accessible via an **AWS Application Load Balancer (ALB)**. The architecture includes:
- **2 Public Subnets**
- **1 Private Subnet**
- **Internet Gateway (IGW)** for public subnets
- **NAT Gateway** to allow private subnet instances to access the internet
- **Bastion Host** in public subnet for SSH access to private subnet instances
- **ALB with Target Group** for routing traffic to the application

---

## Architecture Overview

### VPC and Subnets
- **VPC Name:** `demo-vpc`
- **Public Subnets:**
  - `demo-subnet-public1-us-west-1a`
  - `demo-subnet-public2-us-west-1b`
- **Private Subnets:**
  - `demo-subnet-private1-us-west-1a`
  - `demo-subnet-private2-us-west-1b`

### Networking Components
- **Internet Gateway:** `demo-igw` (Attached to `demo-vpc` for internet access)
- **NAT Gateway:** `demo-nat-public1-us-west-1a` (Allows private subnet instances to access the internet)
- **Route Tables:**
  - `demo-rtb-public` (Associated with public subnets, routes internet traffic via IGW)
  - `demo-rtb-private1-us-west-1a` (Associated with private subnet, routes internet traffic via NAT Gateway)
  - `demo-rtb-private2-us-west-1b` (Same as above for the other private subnet)

---

## Deployment Steps

### 1. Create VPC and Subnets
1. Navigate to **AWS VPC Dashboard**.
2. Create a new **VPC** (`demo-vpc`).
3. Create **two public subnets** and **one private subnet** inside the VPC.
4. Attach an **Internet Gateway (IGW)** (`demo-igw`) to the VPC.


### 2. Configure Route Tables
1. **Public Route Table** (`demo-rtb-public`):
   - Associate it with **public subnets**.
   - Add a default route (`0.0.0.0/0`) pointing to the **Internet Gateway**.

2. **Private Route Table** (`demo-rtb-private1-us-west-1a` & `demo-rtb-private2-us-west-1b`):
   - Associate them with **private subnets**.
   - Add a default route (`0.0.0.0/0`) pointing to the **NAT Gateway** (`demo-nat-public1-us-west-1a`).

![Screenshot from 2025-03-08 23-45-03](https://github.com/user-attachments/assets/405021fe-2b01-4be1-92f8-1ffbedb34961)

### 3. Launch EC2 Instances
1. **Bastion Host (Public Subnet)**:
   - Launch an EC2 instance in `demo-subnet-public1-us-west-1a`.
   - Assign a **Public IP**.
   - Attach a **Key Pair** (e.g., `demo-key`).

2. **Application Server (Private Subnet)**:
   - Launch an EC2 instance in `demo-subnet-private1-us-west-1a`.
   - **Do not assign a public IP**.
   - Use the same **Key Pair** (`demo-key`).


### 4. Setup SSH Access to Private Instance via Bastion Host
1. Copy the **private key** (`demo-key.pem`) from your local system to the **Bastion Host**.
2. SSH into the **Bastion Host**:
   ```bash
   ssh -i demo-key.pem ec2-user@<public-ip-of-bastion>
   ```
3. From the **Bastion Host**, SSH into the **Private Instance**:
   ```bash
   ssh -i demo-key.pem ec2-user@<private-ip-of-app-server>
   ```


### 5. Deploy the Application
1. Copy your application files to the private EC2 instance.
2. Start the application (assuming a simple web server):
   ```bash
   sudo yum install -y httpd
   sudo systemctl start httpd
   sudo systemctl enable httpd
   echo "<h1>Application Deployed Successfully</h1>" | sudo tee /var/www/html/index.html
   ```

📌 *Add Screenshot of Running Application*

### 6. Configure ALB and Target Group
1. Navigate to **AWS EC2 > Load Balancers**.
2. Create an **Application Load Balancer (ALB)**.
   - Name: `demo-alb`
   - Scheme: **Internet-facing**
   - Select the **two public subnets** (`demo-subnet-public1-us-west-1a` & `demo-subnet-public2-us-west-1b`).
3. Create a **Target Group**.
   - Name: `demo-target-group`
   - Target Type: **Instances**
   - Add the **Private EC2 Instance** as a target.
4. Attach the **Target Group** to the **ALB Listener**.

![image](https://github.com/user-attachments/assets/eff0a7ca-a0a3-47d7-98a5-169d2708ee03)
![image](https://github.com/user-attachments/assets/37f7ce24-aa20-4e2d-9239-f61ad301702f)


### 7. Test the Application
1. Find the **ALB DNS Name** from AWS Console.
2. Open a browser and enter:
   ```
   http://[<ALB-DNS-Name>](http://test-788704465.us-west-1.elb.amazonaws.com/index.html#intro)
   ```
3. You should see the deployed application.

![image](https://github.com/user-attachments/assets/49dc4226-0b1a-41fa-a433-71ea10d2c880)

---

