
# Cloud Computing Guide
---
## AWS Load Balancer Setup Guide (Application Load Balancer)

This guide walks through setting up an Application Load Balancer in AWS with two Amazon Linux EC2 instances. The goal is to distribute incoming traffic across multiple instances for better scalability and availability.

---

## 1. Launch EC2 Instances

1. **Create 2 EC2 instances**:
    - **AMI**: Amazon Linux (Ubuntu also works, but install Apache2 instead).
    - Select **two different subnets**, each in a **different Availability Zone (AZ)**.
    - Edit **network settings**:
        - Add **Security Group** with **HTTP (port 80)** and **SSH (port 22)** enabled.

2. **Setup HTTP server on each instance**:
    - Use the following **User Data** script during instance launch or run manually via SSH:
    ```bash
    #!/bin/bash
    yum update -y
    yum install httpd -y   # For Ubuntu, use: apt install apache2 -y
    systemctl start httpd
    systemctl enable httpd
    echo "IP Address: $(hostname -I)" > /var/www/html/index.html
    systemctl restart httpd
    ```

---

## 2. Create Target Group

1. Navigate to **EC2 Dashboard → Target Groups**.
2. Click **Create target group**.
3. **Type**: Choose `Instances`.
4. Provide a **name** for your Target Group (e.g., `my-tg`).
5. **Protocol**: HTTP
6. **IP address type**: IPv4
7. **VPC**: Select your VPC.
8. **Protocol version**: HTTP1 (slow) or HTTP2 (fast) — choose as per requirement.
9. **Health check settings**:
    - Protocol: HTTP
    - Path: `/`
10. Click **Next**, then:
    - Add your previously launched EC2 instances.
    - Click **Register** to attach instances to the Target Group.

---

## 3. Create Application Load Balancer (ALB)

1. Go to **EC2 Dashboard → Load Balancers**.
2. Click **Create Load Balancer** → Select **Application Load Balancer**.
3. **Name** your load balancer.
4. **Scheme**:
    - `Internet-facing` (for public access)
    - `Internal` (for private access within VPC)
5. **Availability Zones**:
    - Select all AZs where your EC2 instances are running.
6. **Security Group**:
    - Create a new Security Group or select an existing one.
    - Allow:
        - HTTP (port 80)
        - SSH (port 22)
7. **Listener and Routing**:
    - Listener protocol: HTTP on port 80
    - Default action: Select the **Target Group** created earlier.

8. Click **Create Load Balancer**.

---

## 4. Access Your Server

- After the ALB is successfully created:
    - Go to **Load Balancers → Your ALB**.
    - Copy the **DNS name** from the description tab.
    - Paste it into your browser.
    - You're using a Load Balancer to distribute incoming traffic between two EC2 instances. When you "access the server", you're actually visiting the Load Balancer, not the EC2s directly. The Load Balancer then forwards your request to one of the instances based on its algorithm.
    - You should see one of the two instance pages (showing hostname/IP) — this verifies load balancing is working.
    - when you refresh or open it in incognito then you might be see the ip address will be change due to this line.
    - echo "IP Address: $(hostname -I)" > /var/www/html/index.html


---
***



## Amazon DynamoDB Setup

### Steps to Create a DynamoDB Table and Perform CRUD Operations

### 1. Create a DynamoDB Table:
- Go to **AWS DynamoDB Console** → Click **Create Table**.
- Enter the **Table Name**.
- Define the **Partition Key** (e.g., `roll` as a `Number`).
- (Optional) Define the **Sort Key** (Composite Key).
- Click **Create Table** and wait for it to be active.

### 2. Insert Data into the Table:
- Select the created **Table**.
- Click **Actions** → **Create Item**.
- Click **Add New Attribute** → Enter **Key-Value Pairs**.
- Click **Create** to insert data.

---

## Amazon DynamoDB CLI Operations (CRUD)

### 1. List Available Tables:

```sh
aws dynamodb list-tables
```

## CRUD Operations in Amazon DynamoDB (CLI)

### 1. Create a Table

```sh
aws dynamodb create-table --table-name <table_name> \
--attribute-definitions AttributeName=<partition-key>,AttributeType=<S/N> \
--key-schema AttributeName=Id,KeyType=HASH \
--provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=6
```
### (*) If it short-ket -> then keyType=RANGE
### 2. Insert Data in Table

```sh
aws dynamodb put-item --table-name <table_name> \
--item '{"partition_key":{"N":"101"}, "name":{"S":"John Doe"}}'
```

### 3. Retrieve All Data (Read - Scan)

```sh
aws dynamodb scan --table-name <table_name>
```

### 4. Retrieve a Specific Item (Read - Get)

```sh
aws dynamodb get-item --table-name <table_name> \
--key '{"partition_key":{"S":"101"}}'
```

### 5. Update an Item

```sh
aws dynamodb update-item --table-name <table_name> \
--key '{"partition_key":{"N":"101"}}' \
--update-expression "SET name = :new_name" \
--expression-attribute-values '{":new_name":{"S":"Jane Doe"}}'
```

### 6. Delete an Item

```sh
aws dynamodb delete-item --table-name <table_name> \
--key '{"partition_key":{"N":"101"}}'
```

### 7. Delete a Table

```sh
aws dynamodb delete-table --table-name <table_name>
```

---
***

## Amazon RDS (SQL Database) Setup

### Steps to Create an RDS Database

### 1. Create an RDS Database:
- Go to **AWS RDS Console** → Click **Create Database**.
- Choose **Standard** (manual settings) or **Easy Create** (default settings).
- Select **Database Engine**: `MySQL`.
- Choose **Version** (with or without Multi-AZ deployment).
- Enable **RDS Extended Support** (optional).
- Select **Template**: `Production`, `Development`, or `Free Tier`.
- Configure **Availability Zone**:
  - `Single Zone` (for cost-effectiveness).
  - `Multi-AZ` (for high availability).
- Set **DB Instance Identifier** (Database Name).
- Enter **Master Username**.
- Set **Password** (`AWS-generated` or `User-defined`).
- Choose **Instance Class**:
  - `Burstable Class` (`t4.micro`, `t3.micro` for Free Tier).
- Configure **Storage**:
  - **Type**: `General Purpose (gp3/gp2)` or `Provisioned IOPS (io1)`.
  - **Allocation**: `20GB` (default).
  - Enable **Auto Scaling** (if needed).
- Configure **Connectivity**:
  - Choose **VPC**.
  - Enable/Disable **Public Access**:
    - `EC2 → No` (for security).
    - `Terminal → Yes` (for direct access).
  - Configure **VPC Security Group**:
    - Allow **port 3306** for MySQL.
- (Optional) Enable **RDS Proxy**.
- Set **Certificate** (default AWS SSL certificate).
- Set **Backup Settings**:
  - **Backup Period** (default or custom).
  - **Delete Protection** (`Enable` → No accidental deletion).
- Click **Create Database** and wait for it to initialize.

---

## Connecting to RDS from Different Environments

# Connecting to RDS from Different Environments

## 1. Connect via Terminal (MariaDB Client)
- **Download MariaDB Client**:
  ```sh
  sudo yum install -y mariadb105
  ```
- **Connect to RDS**:
  ```sh
  mysql -h <endpoint> -u <username> -p <password> -P <port>
  ```
  - Replace `<endpoint>` with the **RDS endpoint** (found in the AWS console).
  - **Default port**: `3306`.

---

## 2. Connect from an EC2 Instance

### 1. Launch an EC2 Instance:
- Go to **AWS EC2 Console** → Click **Launch Instance**.
- Select an **Amazon Machine Image (AMI)**.
- Choose an **Instance Type** (e.g., `t2.micro`).
- Configure **Security Group** to allow **MySQL (3306)**.
- Click **Launch**.

### 2. Connect via SSH:
```sh
ssh -i <your-key>.pem ec2-user@<EC2-IP>
```

### 3. Install MariaDB Client:
```sh
sudo yum update -y
sudo yum install -y mariadb105
```

### 4. Connect to the RDS Database:
```sh
mysql -h <endpoint> -u <username> -p <password> -P 3306
```

---

## 3. Connect from a Local Machine

### 1. Install MySQL Workbench or MariaDB Client:
- **For MySQL Workbench**: [Download Here](https://dev.mysql.com/downloads/workbench/)
- **For MariaDB Client**:
  ```sh
  sudo apt install mariadb-client  # (Ubuntu/Debian)
  brew install mariadb            # (MacOS)
  ```

### 2. Open MySQL Workbench and Create a New Connection:
- **Host**: `<RDS Endpoint>`
- **Username**: `<Master Username>`
- **Password**: `<Your Password>`
- **Port**: `3306`
- Click **Test Connection** → **Connect**.

---

# Basic SQL Operations in RDS

## 1. Show Available Databases:
```sql
SHOW DATABASES;
```

## 2. Create a Database:
```sql
CREATE DATABASE <database_name>;
```

## 3. Use a Database:
```sql
USE <database_name>;
```

## 4. Create a Table:
```sql
CREATE TABLE <table_name> (
    id INT PRIMARY KEY,
    name VARCHAR(50)
);
```

## 5. Describe Table Structure:
```sql
DESCRIBE <table_name>;
```

## 6. Insert Data:
```sql
INSERT INTO <table_name> VALUES (101, 'Ankan');
```

## 7. Select Data:
```sql
SELECT * FROM <table_name>;
```


---
***
# AWS IAM User, Group, and Policy Setup

## 1. Creating an IAM User
- Navigate to **IAM Console** → **Users**.
- Click **Create User**.
- Enter a **Username**.
- Check **Custom Password** and set a password.
- Choose **Permission Settings**:
  - Select **Attach Policies Directly**.
  - Example: **AmazonEC2FullAccess** (for EC2 management).
- Click **Review and Create**.
- **[Login Details]**:
  - IAM users log in via:  
    `https://<aws-account-id>.signin.aws.amazon.com/console`
  - Use the **username** and **password** to access the AWS console.

---

## 2. Creating an IAM Group
- Navigate to **IAM Console** → **User Groups**.
- Click **Create Group**.
- Enter a **Group Name**.
- Select **Users** to add to the group.
- Attach necessary **Policies** (e.g., `AmazonS3FullAccess`, `AmazonEC2ReadOnlyAccess`).
- Click **Create Group**.

---

## 3. Creating an IAM Policy
- Navigate to **IAM Console** → **Policies**.
- Click **Create Policy**.
- Select **JSON Editor** and define the policy rules.
- Example: Read-only access to EC2:
  ```json
  {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Action": "ec2:Describe*",
        "Resource": "*"
      }
    ]
  }

---
***
# AWS Elastic Beanstalk Deployment Guide

## 1. Creating an Elastic Beanstalk Application

### Step 1: Create a New Application
- Navigate to **Elastic Beanstalk Console** → **Create Application**.
- Choose **Environment Type**:
  - `Web Server` (for web apps).
  - `Worker` (for background tasks).
- Enter **Application Name**.
- Set **Environment Name** (auto-generated, but can be modified).
- Choose a **Domain Name** and check availability.
- Select **Platform** (e.g., `Node.js`, `.NET`, `Python`).
  - Choose **Platform Branch** and **Version**.
- **Application Code**: Use a **Sample Application** or upload your ZIP file.
- Choose **Presets** if needed.

---
## If we want to more feature like security, control then do the below steps, other wise it creates EC2, LoadBalencer... automatically.
---

### Step 2: IAM Service Roles
- **Add Service Role**:
  - Navigate to **IAM Console** → **Roles** → **Create Role**.
  - Choose **EC2** as the trusted entity.
  - Attach policies:
    - **Elastic Beanstalk Enhanced Health**
    - **Managed Updates**
    - **Custom Policies (if required)**
- **Roles Based on Environment Type**:
  - **Web Tier** (for frontend applications).
  - **Worker Tier** (for background processing).
  - **Multi-container Docker** (for Docker-based deployments).
- **EC2 Key Pair**: Choose an existing key pair or create a new one.

---

### Step 3: Configure Networking
- Choose **VPC** (`Default` or **Custom VPC**).
- Enable/Disable **Public IP** (based on security needs).
- Select **AWS Region** for deployment.
- (Optional) Attach an **RDS Database** if needed.

---

### Step 4: Instance and Security Configurations
- Select **Root Volume Type** (`SSD`, `GP3`, `Magnetic`).
- Configure **Security Group**:
  - Allow **SSH (Port 22)** for remote access.
  - Allow **HTTP (Port 80)** for web access.
- Enable **Auto Scaling**:
  - **On-Demand Instances**.
  - Select **Instance Type** (`t2.micro`, `t3.medium`, etc.).
  - Default **AMI ID** is pre-selected.

---

### Step 5: Monitoring and Logs
- Use **Default Monitoring Settings**.
- Enable **Enhanced Health Monitoring** (optional).

---

### Step 6: Review and Create
- Review all configurations.
- Click **Create Environment**.
- Wait for deployment to complete.

---

## Uploading a Web Application
- **Prepare a ZIP file** of your web project.
- Navigate to **Elastic Beanstalk Console** → Select **Application**.
- Click **Upload and Deploy**.
- Select your ZIP file and deploy.

---

## Configuration Steps After Deployment
1. **Configure Environment Settings**.
2. **Set Up Service Access** (IAM, security roles).
3. **Configure Networking & Database** (if required).
4. **Manage Instance Traffic & Auto Scaling**.
5. **Set Up Monitoring, Logs, and Updates**.
6. **Review & Apply Changes**.

---
***
# AWS Simple Notification Service (SNS) Guide

## 1. Creating an SNS Topic
- Navigate to **AWS SNS Console** → Click **Create Topic**.
- Choose **Topic Type**:
  - `FIFO (First-In-First-Out)` (For ordered message delivery).
  - `Standard` (For high-throughput messaging, default).
- Enter **Topic Name**.
- Enable **Encryption** (if needed).
- Configure **Access Policy** (who can publish/subscribe).
- Enable **Data Protection** (mask sensitive data if required).
- Configure **Delivery Policy** (set retry policies for failed messages).
- Click **Create Topic**.

> Example Use Cases:
> - **SNS Topic** = Like a **YouTube Channel** (Users subscribe to receive updates).
> - **E-commerce Notifications** (Order confirmation, shipment updates).

---

## 2. Creating an SNS Subscription
- Select the **SNS Topic** from the console.
- Click **Create Subscription**.
- Choose the **Protocol** (Endpoint Type):
  - `Email`
  - `SMS`
  - `Amazon SQS` (for message queuing)
  - `HTTP/HTTPS` (for webhooks)
  - `Lambda` (to trigger functions)
- Enter the **Endpoint** (Email, Phone Number, or URL).
- Configure **Subscription Filter** (Optional, to filter specific messages).
- Enable **Redrive Policy** (Stores messages while users are inactive).
- Click **Create Subscription**.

> The subscriber must confirm the subscription via email/SMS.

---

## 3. Publishing a Message
- Select the **SNS Topic**.
- Click **Publish Message**.
- Enter **Subject** and **Message Text**.
- Click **Send Message**.

> Example Use Cases:
> - **E-commerce**: Send an order confirmation message.
> - **System Alerts**: Notify admins of critical system events.

---

## Additional Notes:
- SNS can be integrated with **SQS**, **Lambda**, and **CloudWatch** for automation.
- Use **FIFO** topics when message order is critical.
- Messages can be **encrypted** for security compliance.

---
***

# AWS CloudWatch Setup

### Steps to Create a CloudWatch Dashboard and Set Up Alarms  

### 1. Create a CloudWatch Dashboard:
- Navigate to **CloudWatch Console** → Click **Dashboards**.
- Click **Create Dashboard**.
- Enter a **Dashboard Name**.
- Select the **Widget Type** (e.g., Line, Table, Number).
- Choose the **AWS Service** to monitor.
- Select the metric **By Name** or **By Resource**.
- Click **Create** to finalize the dashboard.

### 2. Create a CloudWatch Alarm:
- Go to **CloudWatch Console** → Click **Alarms**.
- Click **Create Alarm**.
- Select a **Metric** (e.g., CPU Utilization for EC2).
- Choose a **Time Period** (e.g., 5 minutes).
- Set a **Threshold** (e.g., Trigger alarm when CPU > 80%).
- Select the **Alarm State Trigger**:
  - `In Alarm` (Threshold exceeded).
  - `OK` (Normal condition).
  - `Insufficient Data` (No metric data).
- Attach an **SNS Topic** for notifications (Email, SMS, or Lambda).
- Choose an **Alarm Action** (e.g., Stop, Reboot, or Scale Instance).
- Enter the **Alarm Name**.
- Click **Review and Create**.

---
***
# Docker: Deploying a MERN App

## 1. Overview: What, Where, and Why

- **Problem it solves**: Inconsistent environments between development and production.
- **Where it's used**: Microservices, portable app packaging.
- **Why use it**: Build once, run anywhere. Lightweight containers. Isolated environments.

## 2. Steps to Deploy a MERN App with Docker

### Folder Structure

```
mern-app/
├── backend/
│   ├── Dockerfile
│   └── server.js
├── frontend/
│   ├── Dockerfile
│   └── App.js
└── docker-compose.yml
```

### Backend Dockerfile

```Dockerfile
FROM node:18
WORKDIR /app
COPY backend/package*.json ./
RUN npm install
COPY backend/ .
EXPOSE 5000
CMD ["node", "server.js"]
```

### Frontend Dockerfile

```Dockerfile
FROM node:18
WORKDIR /app
COPY frontend/package*.json ./
RUN npm install
COPY frontend/ .
EXPOSE 3000
CMD ["npm", "start"]
```

### docker-compose.yml

```yaml
version: "3.8"
services:
  backend:
    build: ./backend
    ports:
      - "5000:5000"
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    depends_on:
      - backend
```

### Deploy

```bash
docker-compose up --build
```

Frontend: http://localhost:3000  
Backend: http://localhost:5000

---
***

# Kubernetes: Deploying a MERN App

## 1. Overview: What, Where, and Why

- **Problem it solves**: Managing and scaling containerized apps in production.
- **Where it's used**: Multi-container apps, high availability, microservices.
- **Why use it**: Automates scaling, rolling updates, service discovery, and self-healing.

## 2. Why Kubernetes Over Docker Compose

- **Compose** is suitable for development and simple apps.
- **Kubernetes** supports production-grade orchestration, replicas, monitoring, autoscaling.

## 3. Steps to Deploy MERN App

### Pre-requisites

- Install Minikube or set up Kubernetes cluster.
- Docker images for backend and frontend should be built and pushed to Docker Hub.

### Manifest Files

#### `backend-deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
        - name: backend
          image: <dockerhub-username>/mern-backend
          ports:
            - containerPort: 5000
```

#### `backend-service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  selector:
    app: backend
  ports:
    - port: 5000
      targetPort: 5000
```

#### `frontend-deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
        - name: frontend
          image: <dockerhub-username>/mern-frontend
          ports:
            - containerPort: 3000
```

#### `frontend-service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  type: NodePort
  selector:
    app: frontend
  ports:
    - port: 3000
      targetPort: 3000
      nodePort: 31000
```

### Deploy

```bash
kubectl apply -f backend-deployment.yaml
kubectl apply -f backend-service.yaml
kubectl apply -f frontend-deployment.yaml
kubectl apply -f frontend-service.yaml
```

Access frontend at: `http://<minikube-ip>:31000`

