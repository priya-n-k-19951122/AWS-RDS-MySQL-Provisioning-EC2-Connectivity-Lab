# AWS RDS (MySQL) Provisioning & EC2 Connectivity Lab

This repository documents the end-to-end deployment of a managed **Amazon RDS MySQL database instance** and its secure connection to an **Amazon EC2 Linux client server** inside a shared VPC.

---

## Architecture Overview

| Component | Configuration / Specification |
| :--- | :--- |
| **Database Engine** | Amazon RDS for MySQL (`8.4.10`) |
| **DB Instance Class** | `db.t4g.micro` (Free Tier, Single-AZ) |
| **DB Identifier** | `priyadbs1` |
| **Compute Client** | Amazon EC2 (`t3.small`, Amazon Linux) — `RDS-EC2` |
| **Security Group** | `RDS-SG` (Inbound TCP `3306` & SSH) |

---

## Step 1: Security Group Configuration

Create a dedicated firewall boundary to allow remote administration and MySQL database traffic.

1. Open the **Amazon EC2 Console** and navigate to **Network & Security** -> **Security Groups**.
2. Click **Create security group** and name it `RDS-SG` (Description: `for RDS project`).
3. Configure the **Inbound Rules**:

| Type / Protocol | Port | Source | Purpose |
| :--- | :--- | :--- | :--- |
| **All TCP** (or **MySQL/Aurora**) | `3306` | `0.0.0.0/0` (or `RDS-SG`) | Allows EC2 client to connect to the MySQL RDS endpoint |
| **SSH** | `22` | `0.0.0.0/0` | Allows remote terminal login to the EC2 instance |

4. Click **Create security group**.

---

## Step 2: Provisioning the Amazon RDS MySQL Database

Deploy the managed relational database instance using the Free Tier template.

1. Navigate to the **Amazon RDS Console** -> **Databases** -> **Create database**.
2. Configure the core database settings:
   * **Creation method:** `Full configuration` (Standard create)
   * **Engine type:** `MySQL`
   * **Engine version:** `8.4.10`
   * **Templates:** `Free tier`
   * **Availability and durability:** `Single-AZ DB instance`
3. Set **Settings & Credentials**:
   * **DB instance identifier:** `priyadbs1`
   * **Master username:** `root`
   * **Master password:** `Admin@123` *(Note: Replace with a secure secret in production environments)*
4. Set **Instance configuration & Storage**:
   * **DB instance class:** `db.t4g.micro`
   * **Storage autoscaling:** Enable storage autoscaling
5. Under **Connectivity**, select **Choose existing VPC security groups** and attach `RDS-SG`.
6. Click **Create database**.

---

## Step 3: Launching the EC2 Client Instance

Provision the Linux compute server that will run the MySQL command-line client.

1. Navigate to **EC2** -> **Instances** -> **Launch instance**.
2. Configure the instance details:
   * **Name:** `RDS-EC2`
   * **Instance type:** `t3.small`
   * **Security group:** Select existing security group -> `RDS-SG`
   * **Storage:** `30 GB`
3. Click **Launch instance**.

---

## Step 4: Installing the MySQL Client on EC2

Connect to your `RDS-EC2` server via SSH and install the MariaDB/MySQL client package.

```bash
# 1. Check if MySQL client is pre-installed
mysql --version

# 2. Install the MariaDB 10.5 client utility (includes the mysql CLI)
sudo dnf install mariadb105 -y

# 3. Verify installation
mysql --version
```


---

## Step 5: Connecting to the RDS Endpoint & Verification

Retrieve the database host address from the AWS Console and establish an active SQL session.

1. Open the **Amazon RDS Console** -> **Databases** -> click **`priyadbs1`**.
2. Under the **Connectivity & security** tab, copy the **Endpoint** URL (e.g., `://amazonaws.com`).
3. Return to your EC2 SSH terminal and connect to the database:

```bash
mysql -h <PASTE-YOUR-RDS-ENDPOINT> -P 3306 -u root -p
```


4. When prompted for the password, enter `Admin@123`.
5. Once logged into the `MySQL [(none)]>` prompt, verify database access by running:

```sql
SHOW DATABASES;
```

