# AWS Lambda

## 1. Overview: What, Where, and Why

- **Problem it solves**: Runs code without provisioning or managing servers.
- **Where it's used**: Event-driven applications, microservices, serverless backend processing.
- **Why use it**: Automatically scales, integrates with other AWS services, and reduces operational overhead.

## 2. Steps to Set Up

1. Navigate to AWS Lambda → Click **Create Function**.
2. Choose author from scratch / container / blueprint.
3. Enter Function Name and select Runtime (e.g., Node.js 18.x).
4. Configure permissions:
   - Create or choose an IAM role.
   - Attach permissions like access to S3, CloudWatch, Kinesis.
5. Write or paste your code → Click **Deploy**.
6. **Create Function URL** (optional):
   - Go to function → Function URL → Create.
   - Click the URL to test.
7. Set Environment Variables (if needed).
8. Add Trigger:
   - E.g., S3: Create bucket → Lambda → Add Trigger → Select S3 bucket.
   - E.g., Kinesis: Create Data Stream → Add as Trigger.

## 3. Output
The Lambda function processes data and sends the output to configured destination (e.g., S3, CloudWatch).

---
***


# AWS Kinesis

## 1. Overview: What, Where, and Why

- **Problem it solves**: Real-time data ingestion and processing.
- **Where it's used**: Streaming analytics, log ingestion, real-time dashboards.
- **Why use it**: Handles large amounts of real-time data with low latency.

## 2. Steps to Set Up

### IAM Role Setup
1. Go to IAM → Create Role → Choose Lambda.
2. Add permissions for S3, Kinesis, and CloudWatch.
3. Name the role and create it.

### Producer Lambda
1. Go to Lambda → Create Function → Author from scratch.
2. Set runtime to Node.js 18.
3. Use the IAM Role created earlier.
4. Write/paste code for producing to Kinesis → Deploy.

### Consumer Lambda
1. Similar steps as above, with logic for consuming Kinesis stream.

### Create Kinesis Stream
1. Go to Kinesis → Create Data Stream → On-demand mode → Give a name → Create.

### Trigger Configuration
1. Go to Lambda → Select Function → Add Trigger.
2. For Producer: Select Kinesis.
3. For Consumer: Select S3 bucket or another destination.

---
***

# AWS CloudFront

## 1. Overview: What, Where, and Why

- **Problem it solves**: Content delivery with low latency.
- **Where it's used**: Static/dynamic website distribution, video streaming, API caching.
- **Why use it**: Global edge network, DDoS protection, faster content delivery.

## 2. Steps to Set Up

### EC2 Setup
1. Launch EC2 Instance → Amazon Linux.
2. Add user data script to install NGINX:
   ```bash
   #!/bin/bash
   apt update -y
   apt install nginx -y
   systemctl start nginx
   systemctl enable nginx
   echo $(hostname -I) >/var/www/html/index.html
   systemctl restart nginx
   ```
3. Copy the public DNS.

### CloudFront Distribution
1. Go to CloudFront → Create Distribution.
2. Origin domain: Use EC2 public DNS.
3. Protocol: HTTP and HTTPS.
4. HTTP Methods: Select as per app need.
5. Create Cache Policy (if needed).
6. Skip WAF (optional) → Click Create.

### VPC Prefix List (Optional)
1. Go to VPC → Manage Prefix List → Map IPs to EC2.

---
***

# AWS Key Management Service (KMS)

## 1. Overview: What, Where, and Why

- **Problem it solves**: Encryption key creation and management.
- **Where it's used**: Data protection in S3, EBS, RDS, etc.
- **Why use it**: Centralized, auditable key management with fine-grained access control.

## 2. Steps to Set Up

### KMS Key Creation
1. Go to KMS → Create Key.
2. Choose Key Type: Symmetric.
3. Key Usage: Encrypt/Decrypt.
4. Advanced Option: Choose External (if required).
5. Provide key name and administrative permissions.
6. Finish setup and download the Wrapping Key & Token ZIP file.

### EC2 Setup
1. Launch EC2 Instance (Amazon Linux).
2. Name it and create a key-value pair.
3. Attach an IAM Role with S3 full access.
4. SSH into EC2 and verify AWS CLI:
   ```bash
   aws --version
   ```

### Upload & Download Files
1. Create S3 bucket → Upload files.
2. Use EC2 CLI to download files:
   ```bash
   aws s3 cp s3://<bucket-name> /home/ec2-user --recursive
   ```

### Encryption
1. Generate plaintext:
   ```bash
   openssl rand -out plaintext.bin 32
   ```
2. Use AWS CLI or OpenSSL to encrypt using the downloaded wrapping key and token.
