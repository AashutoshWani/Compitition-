# Task 2: AWS Security Groups Configuration — Step-by-Step Guide

## Overview

Task 2 covers AWS Security Groups configuration for a multi-tier application architecture. The screenshots show:

1. **Security Group Details** — `DB_DG` security group with an inbound MySQL/Aurora rule on port 3306
2. **Security Groups List** — Multiple security groups including `WEB_SG`, `APP_SG`, `DB_DG`, `ALBSG`, `RDS SG`, and several launch-wizard and default groups

---

## Part 1: Security Groups Overview

### Screenshot Reference: Security Groups List

The AWS VPC console shows **4 of 15 security groups** selected. The key security groups in this architecture are:

| Security Group Name | Security Group ID | Description | VPC ID | Owner |
|---------------------|-------------------|-------------|--------|-------|
| WEB_SG | sg-08348749196013ed0 | allow all | vpc-01060bf042c9ac469 | 797989098077 |
| APP_SG | sg-0bad11da0532f15ff6 | all to app | vpc-01060bf042c9ac469 | 797989098077 |
| DB_DG | sg-0375688bfaf885c5 | all to db | vpc-01060bf042c9ac469 | 797989098077 |
| ALBSG | sg-01ca8da9adb3b0a17 | allow http | vpc-01060bf042c9ac469 | 797989098077 |

Additional security groups visible in the list:
- `launch-wizard-1` through `launch-wizard-5` (auto-generated)
- `default` (default VPC security groups)
- `AutoScalingSecurityGroup`
- `RDS SG` (sg-08c8d8235f0fa1f2f) — description: allow for ec2

---

## Part 2: DB_DG Security Group Details

### Screenshot Reference: DB_DG Details Page

The `DB_DG` security group (`sg-0375688bfaf885c5`) details page shows:

| Field | Value |
|-------|-------|
| Security group name | DB_DG |
| Security group ID | sg-0375688bfaf885c5 |
| Description | all to db |
| VPC ID | vpc-01060bf042c9ac469 |
| Owner | 797989098077 |
| Inbound rules count | 1 Permission entry |
| Outbound rules count | 1 Permission entry |

### Inbound Rules for DB_DG

The DB_DG security group has **1 inbound rule**:

| Rule ID | Type | Protocol | Port Range | Source | Description |
|---------|------|----------|------------|--------|-------------|
| sgr-0d63d06071e0087cb | MySQL/Aurora | TCP | 3306 | sg-0bad11da0532f15ff6/A... | — |

This rule allows **MySQL/Aurora traffic on TCP port 3306** from the `APP_SG` security group (`sg-0bad11da0532f15ff6`), enabling application instances to connect to the database.

---

## Step-by-Step: Creating Security Groups

### Step 1: Create WEB_SG (Web Security Group)

1. Navigate to **VPC → Security Groups** in the AWS Console.
2. Click **Create security group**.
3. Configure:
   - **Security group name:** `WEB_SG`
   - **Description:** `allow all`
   - **VPC:** `vpc-01060bf042c9ac469`
4. Add **Inbound rules:**
   - Type: HTTP | Port: 80 | Source: 0.0.0.0/0 (Anywhere-IPv4)
   - Type: HTTPS | Port: 443 | Source: 0.0.0.0/0 (Anywhere-IPv4)
5. Leave **Outbound rules** at default (allow all traffic).
6. Click **Create security group**.

### Step 2: Create ALBSG (Load Balancer Security Group)

1. Click **Create security group**.
2. Configure:
   - **Security group name:** `ALBSG`
   - **Description:** `allow http`
   - **VPC:** `vpc-01060bf042c9ac469`
3. Add **Inbound rules:**
   - Type: HTTP | Port: 80 | Source: 0.0.0.0/0 (Anywhere-IPv4)
4. Leave **Outbound rules** at default.
5. Click **Create security group**.

### Step 3: Create APP_SG (Application Security Group)

1. Click **Create security group**.
2. Configure:
   - **Security group name:** `APP_SG`
   - **Description:** `all to app`
   - **VPC:** `vpc-01060bf042c9ac469`
3. Add **Inbound rules:**
   - Type: HTTP | Port: 80 | Source: `ALBSG` (sg-01ca8da9adb3b0a17)
   - Type: SSH | Port: 22 | Source: 0.0.0.0/0 (for management access)
4. Leave **Outbound rules** at default.
5. Click **Create security group**.

### Step 4: Create DB_DG (Database Security Group)

1. Click **Create security group**.
2. Configure:
   - **Security group name:** `DB_DG`
   - **Description:** `all to db`
   - **VPC:** `vpc-01060bf042c9ac469`
3. Add **Inbound rules:**
   - Type: MySQL/Aurora | Port: 3306 | Protocol: TCP | Source: `APP_SG` (sg-0bad11da0532f15ff6)
4. Leave **Outbound rules** at default.
5. Click **Create security group**.

### Step 5: Create RDS SG (RDS Security Group)

1. Click **Create security group**.
2. Configure:
   - **Security group name:** `RDS SG`
   - **Description:** `allow for ec2`
   - **VPC:** `vpc-01060bf042c9ac469`
3. Add **Inbound rules:**
   - Type: MySQL/Aurora | Port: 3306 | Source: `APP_SG` (sg-0bad11da0532f15ff6)
4. Leave **Outbound rules** at default.
5. Click **Create security group**.

---

## Security Group Architecture

```
┌─────────────┐      ┌─────────────┐      ┌─────────────┐
│   ALBSG     │      │   WEB_SG    │      │   APP_SG    │
│ sg-01ca8... │      │ sg-08348... │      │ sg-0bad11...│
│ Port 80     │      │ Port 80/443 │      │ Port 80     │
│ 0.0.0.0/0   │      │ 0.0.0.0/0   │      │ ALBSG + SSH │
└──────┬──────┘      └──────┬──────┘      └──────┬──────┘
       │                    │                    │
       └────────────────────┴────────────────────┘
                              │
                              ▼
                    ┌─────────────────┐
                    │     DB_DG       │
                    │ sg-037568...    │
                    │ Port 3306 (MySQL│
                    │ from APP_SG)    │
                    └─────────────────┘
```

### Traffic Flow Summary

| From | To | Protocol | Port | Purpose |
|------|----|----------|------|---------|
| Internet (0.0.0.0/0) | ALBSG | TCP | 80 | External HTTP access |
| Internet (0.0.0.0/0) | WEB_SG | TCP | 80, 443 | External web access |
| ALBSG | APP_SG | TCP | 80 | Load balancer to app |
| Internet (0.0.0.0/0) | APP_SG | TCP | 22 | SSH management |
| APP_SG | DB_DG | TCP | 3306 | App to database |
| APP_SG | RDS SG | TCP | 3306 | App to RDS |

---

## Verification Checklist

- [ ] **WEB_SG** created with HTTP (80) and HTTPS (443) open to 0.0.0.0/0
- [ ] **ALBSG** created with HTTP (80) open to 0.0.0.0/0
- [ ] **APP_SG** created with HTTP (80) from ALBSG and SSH (22) from 0.0.0.0/0
- [ ] **DB_DG** created with MySQL/Aurora (3306) from APP_SG only
- [ ] **RDS SG** created with MySQL/Aurora (3306) from APP_SG
- [ ] All security groups are in VPC `vpc-01060bf042c9ac469`
- [ ] DB_DG inbound rule count: 1 (MySQL/Aurora on 3306 from APP_SG)
- [ ] DB_DG outbound rule count: 1 (default allow all)

---

## Notes

- The `DB_DG` security group restricts database access to only `APP_SG`, following the principle of least privilege.
- The `WEB_SG` and `ALBSG` allow traffic from anywhere (0.0.0.0/0) on port 80 for public web access.
- All security groups share the same VPC: `vpc-01060bf042c9ac469`.
- Owner ID for all security groups: `797989098077`.

---

*All screenshots referenced above are attached as images 11.png (DB_DG details) and 12.png (Security Groups list).*
