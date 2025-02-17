# =========================================
# Cloud Computing Guide
# =========================================

## Amazon Machine Image (AMI) Setup

### Steps to Create an AMI and Launch a New Instance

### 1. Launch an EC2 Instance:
- Go to **EC2 Dashboard** → Click **Launch Instance**.
- Enter the **Instance Name**.
- Select the desired **Amazon Machine Image (AMI)** (e.g., Ubuntu).
- Choose an **Instance Type** (e.g., `t2.micro`).
- Create a **Key Pair** or use an existing one for secure SSH access.
- Configure **Security Group**:
  - Allow **HTTP (port 80)** for web access.
  - Allow **SSH (port 22)** for remote access.
- Go to **Advanced Details**, add any custom user data script (if needed).
- Click **Launch Instance** and wait for it to initialize.

### 2. Configure the Server (Nginx, Python, and Directory Setup):
- Connect to the instance using **SSH**.
- Switch to root using `sudo su`.
- Create a project directory using `mkdir /proj`.
- Set appropriate permissions using `sudo chmod -R 755 /proj`.
- Verify the setup using `ls -l /proj`.

### 3. Create an Amazon Machine Image (AMI):
- In the **EC2 Dashboard**, select the configured instance.
- Click **Actions** → **Image and Templates** → **Create Image**.
- Provide an **Image Name** and optional **description**.
- Click **Create Image** and wait for the AMI to be available.

### 4. Launch a New Instance Using the Custom AMI:
- Go to **Launch Instance** → Enter **Instance Name**.
- Under **AMI Selection**, choose **My AMIs** and select the previously created AMI.
- Choose the **Instance Type**, **Key Pair**, and existing **Security Group**.
- Click **Launch Instance** and wait for initialization.
- Connect to the new instance via **SSH** and verify that `/proj` from the previous server exists.

#*********************************************************************************************************************************
## Simple Storage Service (S3) & Static Website Hosting

### 1. Create an S3 Bucket
- Navigate to **AWS S3** → Click **Create Bucket**.
- Enter a **Unique Bucket Name** (e.g., `my-bucket-name`).
- Select the **Region** (optional).
- Choose **Bucket Type**:
  - **General Purpose** (default)
  - **Directory (fast access)**
- Configure **Access Permissions**:
  - Set **Object Ownership**.
  - Enable/Disable **Block Public Access** (depends on usage).
  - Enable **Bucket Versioning** (optional, for file history tracking).
- Add **Tags** for better organization.
- Set **Default Encryption** (choose an encryption type for security).
- Enable/Disable **Bucket Key** to reduce encryption costs.
- Configure **Advanced Settings** (Object Lock for data retention).
- Click **Create Bucket**.

### 2. Upload Files to S3
- Open the **Bucket** → Click **Upload Files**.
- Select the files you want to store.
- Choose the **Storage Class** under **Properties**.
- Click **Upload**.

### 3. Grant Public Access to Files
- **Edit Bucket Permissions**:
  - Disable **Block Public Access**.
  - Enable **Object Ownership**.
- **Two methods to make files public**:
  - **A. Using ACL (Access Control List) – Per File Basis**
    - Select the **File** → Click **Actions** → **ACL** → **Make Public**.
  - **B. Using Bucket Policy – Apply to Entire Bucket**
    - Open **Permissions** → **Edit Bucket Policy**.
    - Copy the **ARN (Amazon Resource Name)** of the bucket.
    - Use **AWS Policy Generator**:
      - Service: **S3**
      - Principal: `*` (allows public access)
      - Action: `s3:GetObject`
      - ARN: `arn:aws:s3:::your-bucket-name/*`
    - Add the **Statement**, generate, copy, and save the policy.

### 4. Enable Static Website Hosting in S3
- Open the **Bucket** → **Properties** → Scroll to **Static Website Hosting**.
- Enable **Static Website Hosting**.
- Select **Index Document** (e.g., `index.html`).
- *(Optional)* Set **Error Document** (e.g., `error.html`).
- Save changes.

### 5. Manage Bucket Lifecycle Rules
- Navigate to **Bucket** → **Management** → **Lifecycle Rule**.
- Configure **automatic transitions** or **expiration rules** for objects.

#*********************************************************************************************************************************
## Virtual Private Cloud (VPC) Setup

### 1. Create a VPC
- Log in to **AWS Console** → Search **VPC** → Click **Create VPC**.
- Provide a **Name**.
- Specify **IPv4 CIDR Block** (e.g., `10.0.0.0/16`).
- Click **Create VPC**.

### 2. Create an Internet Gateway (IGW)
- Navigate to **Internet Gateways** → Click **Create Internet Gateway**.
- Provide a **Name** → Click **Create Internet Gateway**.
- Select the **IGW** → Click **Actions** → **Attach to VPC**.
- Choose the **VPC** and attach it.

### 3. Create Public & Private Subnets
- Open **Subnets** → Click **Create Subnet**.
- Select the **VPC** created earlier.
- Enter **Subnet Name**.
- Choose an **Availability Zone** (e.g., `us-east-1a`).
- Assign **IPv4 CIDR Block** (e.g., `10.0.1.0/24` for the first subnet).
- Repeat for additional **Subnets** (e.g., `10.0.2.0/24` for a private subnet).
- Click **Create Subnet**.

### 4. Configure Route Tables
- Navigate to **Route Tables** → Click **Create Route Table**.
- Provide a **Name** → Select the **VPC** → Click **Create**.
- Select the **Route Table** → Click **Edit Routes**.
- Click **Add Route**:
  - **Destination**: `0.0.0.0/0` (allows internet access).
  - **Target**: Select the **Internet Gateway**.
- Save changes.

#*********************************************************************************************************************************
## Auto Scaling Setup

### 1. Create a Launch Template
- Go to **EC2 Dashboard** → Click **Launch Templates** → **Create Launch Template**.
- Enter **Template Name** and optional **Description**.
- Choose **AMI (Amazon Machine Image)** (e.g., Ubuntu).
- Select **Instance Type** (e.g., `t2.micro`).
- Configure **Network Settings** and **Security Group**.
- Click **Create Launch Template**.

### 2. Create an Auto Scaling Group
- Navigate to **Auto Scaling Groups** → Click **Create Auto Scaling Group**.
- Enter **Auto Scaling Group Name**.
- Select the **Launch Template**.
- Choose **Availability Zones** and configure settings.
- Set **Scaling Policies** (e.g., Desired, Min, and Max Capacity).
- Review and create the **Auto Scaling Group**.

### 3. Verify Auto Scaling
- Navigate to **EC2 Instances**.
- Confirm the **desired number of instances** are running.

This setup ensures efficient resource management, security, and auto-scaling for AWS cloud services. 🚀

