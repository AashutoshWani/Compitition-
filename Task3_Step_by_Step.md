# Task 3: VPC, EC2, ALB & RDS Setup — Step-by-Step Guide

## Overview

Task 3 requires setting up a full network and application architecture in AWS:

1. Configure a VPC with **2 public subnets** and **2 private subnets**
2. Launch **2 EC2 instances** (t3.micro, free tier) in the public subnets
3. Set up an **Application Load Balancer (ALB)** to route traffic to the instances
4. Create an **RDS MySQL database** in the private subnets
5. Configure appropriate **Security Groups** for each component

**Region:** us-east-2 (Ohio)

---

## Step 1: Create the VPC

1. Open the **AWS Management Console** and navigate to **VPC**.
2. Click **Create VPC**.
3. Fill in the following details:
   - **Name tag:** `task3vpc`
   - **IPv4 CIDR:** `10.0.0.0/16`
   - **IPv6 CIDR:** (Optional — leave blank)
   - **Tenancy:** Default
4. Under **Add subnets**, add the following:

   | Subnet Name | Type       | AZ      | CIDR         |
   |-------------|------------|---------|--------------|
   | public-1    | Public     | us-east-2a | 10.0.1.0/24  |
   | public-2    | Public     | us-east-2b | 10.0.2.0/24  |
   | private-1   | Private    | us-east-2a | 10.0.3.0/24  |
   | private-2   | Private    | us-east-2b | 10.0.4.0/24  |
   | DB-1        | Private    | us-east-2a | 10.0.5.0/24  |
   | DB-2        | Private    | us-east-2b | 10.0.6.0/24  |

5. **Add an IGW (Internet Gateway):** Check the option **"Add an IGW"**.
6. Under **Route tables**, check:
   - **"Add a route table for each subnet"** (for individual subnets)
   - **"Create public subnets"** (public subnets will get a route to the IGW)
7. Click **Create VPC**.

### Verify Subnets
1. In the VPC dashboard, click on **Subnets**.
2. You should see all 6 subnets listed:
   - `public-1` (10.0.1.0/24)
   - `public-2` (10.0.2.0/24)
   - `private-1` (10.0.3.0/24)
   - `private-2` (10.0.4.0/24)
   - `DB-1` (10.0.5.0/24)
   - `DB-2` (10.0.6.0/24)

### VPC Subnets Dashboard
*Shows all 6 subnets (public-1, public-2, private-1, private-2, DB-1, DB-2) with their CIDR blocks and Availability Zones.*

### Resource Map
*The VPC resource map shows the network structure with subnets across two Availability Zones (us-east-2a and us-east-2b) and their associated route tables (task3 and rtb-0b5b939ce3b9ad169).*

---

## Step 2: Create Security Groups

Create **three** security groups in the VPC:

### 2a. ALBSG (for the Application Load Balancer)

1. Navigate to **EC2 → Security Groups**.
2. Click **Create security group**.
3. Name it: `ALBSG`
4. Select VPC: `task3vpc`
5. Add an **Inbound rule:**
   - Type: **HTTP**
   - Port: **80**
   - Source: **Anywhere-IPv4 (0.0.0.0/0)**
6. Leave outbound rules at default (allow all).
7. Click **Create security group**.

### 2b. APP-EC2 (for the EC2 instances)

1. Click **Create security group**.
2. Name it: `APP-EC2`
3. Select VPC: `task3vpc`
4. Add **Inbound rules:**
   - Type: **HTTP** | Port: **80** | Source: `ALBSG` (select the security group)
   - Type: **Custom TCP** | Port: **22** | Source: **Anywhere-IPv4** (for SSH)
5. Leave outbound rules at default.
6. Click **Create security group**.

### 2c. DB_SG (for the RDS MySQL database)

1. Click **Create security group**.
2. Name it: `DB_SG`
3. Select VPC: `task3vpc`
4. Add **Inbound rule:**
   - Type: **MySQL/Aurora** | Port: **3306** | Source: `APP-EC2` (select the security group)
5. Leave outbound rules at default.
6. Click **Create security group**.

![Security Groups](images/6.png)

---

## Step 3: Launch EC2 Instances (t3.micro)

### 3a. Launch Instance 1 — APP-1

1. Navigate to **EC2 → Launch instances**.
2. **Choose an AMI:**
   - Select **Amazon Linux 2023** (or Ubuntu).
3. **Choose instance type:**
   - Select **t3.micro** (free tier eligible).
4. **Configure instance:**
   - **Instance name:** `APP-1`
   - **IAM role:** Create or attach an IAM role that allows the instance to access RDS (for connecting to the database).
     - The role should have a policy allowing `rds-db:*` or the appropriate RDS access.
   - **Key pair:** Select an existing key pair or create a new one.
5. **Configure network:**
   - **VPC:** `task3vpc`
   - **Subnet:** `public-1` (10.0.1.0/24)
   - **Assign public IP:** Yes
   - **Auto-assign public IP:** Select the option to always assign
   - **Security group:** Select `APP-EC2`
6. **Configure storage:**
   - Leave default (8 GB gp3).
7. **Review and launch.**

### 3b. Launch Instance 2 — APP-1 (Second Instance)

1. Repeat the same steps as above.
2. **Subnet:** Select `public-2` (10.0.2.0/24).
3. **Instance name:** `APP-1`
4. **Security group:** Select `APP-EC2`
5. **Review and launch.**

### Verify Instances
Navigate to **EC2 → Instances**. You should see both instances in **Running** state.

![EC2 Instances](images/3.png)

---

## Step 4: Configure the EC2 Instances

SSH into each instance to set up a basic web server:

### Connect via SSH
```bash
ssh ec2-user@<public-ip-address>
```

### Install and Start Web Server (Apache)
```bash
# Update the package manager
sudo yum update -y

# Install Apache
sudo yum install -y httpd

# Create a simple web page
echo "Hello from APP-1 - Instance 1" | sudo tee /var/www/html/index.html

# Start and enable Apache
sudo systemctl start httpd
sudo systemctl enable httpd

# Verify it's running
curl http://localhost
```

> **Note:** On the second instance, you can use a different message like `"Hello from APP-1 - Instance 2"` to differentiate traffic distribution.

---

## Step 5: Create the Application Load Balancer (ALB)

### 5a. Create a Target Group

1. Navigate to **EC2 → Target Groups** (under Load Balancing).
2. Click **Create target group**.
3. Fill in the details:
   - **Target group name:** `task3TG`
   - **Target type:** Instance
   - **Protocol:** HTTP
   - **Port:** 80
   - **VPC:** `task3vpc`
4. Click **Next** to go to **Configure targets**.
5. Under **Registered targets**, click **Add instance**.
6. Select both **APP-1** instances.
7. Click **Register instance**.

### 5b. Create the ALB

1. Navigate to **EC2 → Load Balancers**.
2. Click **Create load balancer**.
3. Select **Application Load Balancer**.
4. **Name:** `TASK3ALB`
5. **Scheme:** Internet-facing
6. **Listeners:**
   - Protocol: **HTTP**
   - Port: **80**
7. **Availability zones:**
   - Select `us-east-2a` with subnet `public-1`
   - Select `us-east-2b` with subnet `public-2`
8. **VPC:** `task3vpc`
9. **Security groups:** Select `ALBSG`
10. Click **Next: Register targets**
    - Select the target group `task3TG`
11. **Review and create.**

### Verify ALB
After creation, the ALB should be in **Active** status.

![ALB Details](images/4.png)

### Verify Target Group
Navigate to **Target Groups → task3TG**. Both targets should show as **Healthy**.

![Target Group](images/7.png)

### Test the Load Balancer
Open a browser and navigate to the ALB's DNS name:
```
http://TASK3ALB-xxxxx.us-east-2.elb.amazonaws.com
```
You should see responses alternating between the two instances (round-robin load balancing).

---

## Step 6: Create the RDS MySQL Database

1. Navigate to **RDS → Databases**.
2. Click **Create database**.
3. Under **Standard create**:
   - **Engine:** MySQL (Community)
   - **Edition:** Standard
   - **Version:** 8.0.x (latest)
4. **Template:** General purpose
5. **DB instance identifier:** `database-2`
6. **Master username:** `admin` (or as specified)
7. **Master password:** Set a strong password
8. **DB instance class:** **db.t4g.micro** (free tier eligible)
9. **Storage:**
   - Type: gp3
   - Size: 20 GB (default)
10. **Connectivity:**
    - **VPC:** `task3vpc`
    - **Public accessibility:** **No** (must be private)
    - **Subnet group:**
      - If no custom subnet group exists, create one:
        - Name: `task3-db-subnet-group`
        - VPC: `task3vpc`
        - Subnets: `DB-1` and `DB-2`
      - Select the custom subnet group.
    - **Availability Zone:** Choose `us-east-2b` (or preferred)
    - **Multi-AZ:** No
11. **Security:**
    - **Security group:** Select `DB_SG`
12. **Database settings:**
    - **Initial database name:** (optional, e.g., `appdb`)
    - **Port:** 3306
13. Under **Additional configuration**:
    - **Automated backups:** Enable (Retention: 1 day)
    - **Deletion policy:** Delete automatically
    - **Deletion protection:** Disable
14. Click **Create database**.

### Verify Database
After a few minutes, the RDS instance should be **Available**.

![RDS Database](images/5.png)

### Connect to the Database
From the RDS **Connectivity & security** tab, use the provided connection details.
You can connect from the EC2 instances using the MySQL client:

```bash
# On the EC2 instance, install MySQL client
sudo yum install -y mysql

# Connect to the database
mysql -h <RDS-internal-endpoint> -u admin -p

# Example:
mysql -h database-2.xxxxxx.us-east-2.rds.amazonaws.com -u admin -p
```

---

## Step 7: End-to-End Verification

1. **ALB:** Access `http://TASK3ALB-xxxxx.us-east-2.elb.amazonaws.com` in a browser.
   - Verify the page loads and rotates between the two EC2 instances.
2. **Target Group:** Confirm both targets in `task3TG` show **Healthy**.
3. **EC2 → RDS:** SSH into either EC2 instance and connect to the RDS MySQL database using the internal endpoint.
4. **VPC:** Verify in the VPC dashboard that all subnets are in the correct zones and the IGW is attached.

---

## Architecture Summary

```
Internet
    |
    v
[ TASK3ALB (ALB - Internet-facing) ]
    |        (subnets: public-1, public-2)
    |
    +--- task3TG (Target Group)
    |
    +---> [ APP-1 #1 ]  (public-1, us-east-2a)
    +---> [ APP-1 #2 ]  (public-2, us-east-2b)

    [ APP-1 #1 / #2 ] -----> [ database-2 (RDS MySQL) ]
         (DB_SG: 3306)            (DB-1/DB-2 subnets, private)
```

### Security Group Rules Summary

| SG        | Inbound Rule                     | Source     |
|-----------|----------------------------------|------------|
| ALBSG     | HTTP (80)                        | 0.0.0.0/0  |
| APP-EC2   | HTTP (80)                        | ALBSG      |
| APP-EC2   | SSH (22)                         | 0.0.0.0/0  |
| DB_SG     | MySQL (3306)                     | APP-EC2    |

---

*All resources are in the **us-east-2 (Ohio)** region within **VPC: task3vpc**.*
