# Task 1: Cloud Portfolio, MySQL Database & VPC Setup — Step-by-Step Guide

## Overview

Task 1 covers three components across cloud platforms:

1. **GCP Portfolio Dashboard** — Aashutosh Wani's portfolio page with API, AI, and GCP sections
2. **MySQL Database** — Setting up `company_db` with a `customers` table and querying records
3. **AWS VPC** — Creating a VPC with three subnets: Public-Subnet, APP-Subnet, and DB-Subnet

---

## Part 1: GCP Portfolio Dashboard

### Screenshot Reference: Portfolio Header

The first image shows a GCP-themed portfolio dashboard with the following elements:

- **Name:** Aashutosh Wani
- **Navigation tabs:** API, AI, GCP
- **Grid menu icon:** For additional navigation options

### Step-by-Step: Setting Up a GCP Portfolio Dashboard

1. **Log in to Google Cloud Console**
   - Navigate to https://console.cloud.google.com/
   - Sign in with your Google account

2. **Create a New Project**
   - Click on the project selector dropdown at the top of the page
   - Click **New Project**
   - Enter a project name (e.g., "aashutosh-wani-portfolio")
   - Click **Create**

3. **Enable Required APIs**
   - Navigate to **APIs & Services → Library**
   - Search for and enable APIs relevant to your portfolio (e.g., Cloud AI APIs, Compute Engine API)

4. **Deploy Your Portfolio Website**
   - Option A: Host on **Cloud Storage** as a static website
   - Option B: Deploy on **App Engine** or **Cloud Run**
   - Option C: Use **Firebase Hosting** for a quick deployment

5. **Configure Navigation Tabs**
   - Add API, AI, and GCP sections to your portfolio HTML/CSS
   - Link each tab to relevant projects or documentation

---

## Part 2: MySQL Database Setup

### Screenshot Reference: MySQL Terminal

The second image shows MySQL commands executed in a terminal:

```
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| company_db         |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.02 sec)

mysql> use company_db;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> select * from customers;
+------+-------------+
| name | number      |
+------+-------------+
| yash | 1234567890  |
| DD   | 789546321   |
+------+-------------+
2 rows in set (0.00 sec)
mysql>
```

### Step-by-Step: Setting Up MySQL Database

#### Step 2a: Install MySQL

**On Ubuntu/Debian:**
```bash
sudo apt update
sudo apt install mysql-server
sudo mysql_secure_installation
```

**On macOS (with Homebrew):**
```bash
brew install mysql
brew services start mysql
```

**On Windows:**
- Download and install MySQL Installer from https://dev.mysql.com/downloads/installer/
- Follow the installation wizard and set a root password

#### Step 2b: Log in to MySQL

```bash
mysql -u root -p
```

Enter your root password when prompted.

#### Step 2c: Create the Database

```sql
mysql> show databases;
mysql> create database company_db;
mysql> use company_db;
```

#### Step 2d: Create the Customers Table

```sql
mysql> create table customers (
    id int auto_increment primary key,
    name varchar(50),
    number varchar(20)
);
```

#### Step 2e: Insert Sample Records

```sql
mysql> insert into customers (name, number) values ('yash', '1234567890');
mysql> insert into customers (name, number) values ('DD', '789546321');
```

#### Step 2f: Query the Data

```sql
mysql> select * from customers;
```

Expected output:
```
+------+-------------+
| name | number      |
+------+-------------+
| yash | 1234567890  |
| DD   | 789546321   |
+------+-------------+
2 rows in set (0.00 sec)
```

---

## Part 3: AWS VPC & Subnet Setup

### Screenshot Reference: AWS VPC Subnets Dashboard

The third image shows the AWS VPC Subnets page with three subnets:

| Name          | Subnet ID                        | State   | VPC                              | IPv4 CIDR    |
|---------------|----------------------------------|---------|----------------------------------|--------------|
| Public-Subnet | subnet-049be2a0d722e6fa0        | Available | vpc-01060bf042c9ac469 | 10.0.1.0/24 |
| APP-Subnet    | subnet-077a8d3669bcaf324        | Available | vpc-01060bf042c9ac469 | 10.0.3.0/24 |
| DB-Subnet     | subnet-0b4f6654eeb46c9cc        | Available | vpc-01060bf042c9ac469 | 10.0.2.0/24 |

All subnets are in the same VPC: `vpc-01060bf042c9ac469`

### Step-by-Step: Creating the VPC and Subnets

#### Step 3a: Create the VPC

1. Log in to the **AWS Management Console**
2. Navigate to **VPC → Your VPCs**
3. Click **Create VPC**
4. Configure:
   - **Name tag:** (e.g., `task1-vpc`)
   - **IPv4 CIDR block:** `10.0.0.0/16`
   - **Tenancy:** Default
5. Click **Create**

#### Step 3b: Create Subnets

Navigate to **VPC → Subnets** and create three subnets:

**Public-Subnet:**
- VPC: Select the VPC created above
- Subnet name: `Public-Subnet`
- Availability Zone: Choose your preferred AZ (e.g., us-east-1a)
- IPv4 CIDR block: `10.0.1.0/24`

**APP-Subnet:**
- VPC: Same VPC
- Subnet name: `APP-Subnet`
- Availability Zone: Choose a different AZ (e.g., us-east-1b)
- IPv4 CIDR block: `10.0.3.0/24`

**DB-Subnet:**
- VPC: Same VPC
- Subnet name: `DB-Subnet`
- Availability Zone: Choose your preferred AZ (e.g., us-east-1a)
- IPv4 CIDR block: `10.0.2.0/24`

#### Step 3c: Configure Internet Gateway (for Public-Subnet)

1. Navigate to **VPC → Internet Gateways**
2. Click **Create internet gateway**
3. Name it: `task1-igw`
4. Click **Create**, then attach it to your VPC
5. Navigate to **VPC → Route Tables**
6. Select the route table associated with Public-Subnet
7. Add a route:
   - Destination: `0.0.0.0/0`
   - Target: Internet Gateway (`task1-igw`)

#### Step 3d: Verify Subnets

Navigate to **VPC → Subnets** and confirm:

| Name          | State   | IPv4 CIDR    | VPC                              |
|---------------|---------|--------------|----------------------------------|
| Public-Subnet | Available | 10.0.1.0/24 | vpc-01060bf042c9ac469 |
| APP-Subnet    | Available | 10.0.3.0/24 | vpc-01060bf042c9ac469 |
| DB-Subnet     | Available | 10.0.2.0/24 | vpc-01060bf042c9ac469 |

---

## Architecture Summary

```
┌─────────────────────────────────────────────────────────────┐
│                    GCP Portfolio                            │
│              Aashutosh Wani (API · AI · GCP)                │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    MySQL Database                           │
│              Database: company_db                           │
│              Table: customers (name, number)                │
│              Records: yash (1234567890), DD (789546321)     │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    AWS VPC (vpc-01060bf042c9ac469)          │
│                                                             │
│   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │
│   │ Public-Subnet│  │  APP-Subnet  │  │  DB-Subnet   │    │
│   │ 10.0.1.0/24  │  │ 10.0.3.0/24  │  │ 10.0.2.0/24  │    │
│   └──────────────┘  └──────────────┘  └──────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

---

## Verification Checklist

- [ ] GCP Portfolio dashboard loads with Aashutosh Wani's name
- [ ] API, AI, and GCP navigation tabs are present
- [ ] MySQL `company_db` database exists
- [ ] `customers` table has `name` and `number` columns
- [ ] Two records are inserted: `yash` and `DD`
- [ ] AWS VPC contains three subnets: Public-Subnet, APP-Subnet, DB-Subnet
- [ ] All subnets show **Available** state
- [ ] Public-Subnet has an Internet Gateway route configured

---

*All screenshots referenced above are attached as images 8.png, 9.png, and 10.png.*
