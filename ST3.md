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
# 3. Sample Integration: Kinesis → Lambda → S3

### 1. IAM Role Setup

```bash
IAM → Create Role → Select Lambda → Attach Policies (S3, Kinesis, CloudWatch)
```

>  **Tip**: Always attach the minimum permissions needed.

### 2. Kinesis Stream

```bash
Kinesis → Create Data Stream → On-Demand → Name it → Create
```

>  **Tick**: Think of "Kine-Create-On-Demand" as KOD: "Kinesis On Demand"

### 3. Lambda Function (Producer Example)

```js
const AWS = require('aws-sdk');
const kinesis = new AWS.Kinesis();

exports.handler = async (event) => {
  const record = {
    Data: JSON.stringify({ message: "Hello from Lambda" }),
    PartitionKey: "key1",
    StreamName: "my-kinesis-stream"
  };
  await kinesis.putRecord(record).promise();
  return { statusCode: 200, body: "Data sent" };
};
```

**Explanation**:
- `require('aws-sdk')`: Load AWS SDK.
- `new AWS.Kinesis()`: Create service object.
- `putRecord(...)`: Push data to stream.
- `PartitionKey`: Used to distribute records.
- `StreamName`: Your stream name.

>  **Tip**: Use `JSON.stringify()` for structured logs.

### 4. Trigger Lambda with Kinesis

```bash
Lambda → Add Trigger → Choose Kinesis → Select Stream
```

### 5. Output to S3

```js
const AWS = require('aws-sdk');
const s3 = new AWS.S3();

exports.handler = async (event) => {
  const data = event.Records.map(record => Buffer.from(record.kinesis.data, 'base64').toString('utf-8'));
  await s3.putObject({
    Bucket: 'my-output-bucket',
    Key: `output-${Date.now()}.json`,
    Body: JSON.stringify(data)
  }).promise();
  return { statusCode: 200 };
};
```

**Explanation**:
- `event.Records`: Kinesis records.
- `Buffer.from(...).toString('utf-8')`: Decode Base64.
- `putObject`: Upload file to S3.

---

##  Memory Tricks

- **L-K-S**: Lambda → Kinesis → S3 (flow of data)
- **PPT**: Permission → Producer → Trigger
- **KOD**: Kinesis On-Demand
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

# AWS CloudFormation

## 1. Overview: What, Where, and Why

- **What**: Infrastructure as Code (IaC) service to model and provision AWS resources.
- **Where it's used**: Automate infrastructure setup in a reliable, repeatable way.
- **Why use it**:
  - Version-controlled infrastructure.
  - Easier replication across environments.
  - Integration with CI/CD pipelines.
  - Rollback capabilities on failure.

## 2. Problem it Solves

- Manual setup of cloud infrastructure is error-prone and slow.
- CloudFormation automates resource provisioning and enforces consistency.

## 3. Where it's Used

- Automating EC2, S3, RDS, VPC setup.
- Deploying scalable web apps or microservices.
- Infrastructure replication in multi-account environments.

## 4. Steps to Set Up

### Create a CloudFormation Template

Use YAML or JSON. Example: EC2 with Security Group

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: EC2 instance with Security Group and NGINX

Resources:
  MySecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allow HTTP
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0

  MyEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t2.micro
      ImageId: ami-0c02fb55956c7d316  # Update to a valid region-specific Amazon Linux 2 AMI
      SecurityGroupIds:
        - !Ref MySecurityGroup
      UserData:
        Fn::Base64: |
          #!/bin/bash
          yum update -y
          yum install nginx -y
          systemctl start nginx
          systemctl enable nginx
          echo $(hostname -I) >/usr/share/nginx/html/index.html
```

### Deploy the Template

1. Go to **CloudFormation Console**.
2. Click **Create stack** → With new resources (standard).
3. Choose:
   - **Template source**: Upload a template file or S3 URL.
4. Enter stack name (e.g., `NginxEC2Stack`).
5. Configure options (optional tags, rollback, etc.).
6. Click **Next** → **Create stack**.

### Post Deployment

- Go to **EC2 Console** → Get public IP of the instance.
- Open in browser: `http://<Public-IP>` to confirm NGINX is running.

## 5. Optional Enhancements

- Add output section to get instance IP:
```yaml
Outputs:
  InstancePublicIp:
    Description: "Public IP of EC2 instance"
    Value: !GetAtt MyEC2Instance.PublicIp
```

- Use Parameters section for reusable values (AMI, instance type, etc.).
- Add RDS, S3, or Auto Scaling with other resource blocks.

## 6. Tips

- Always validate template with `cfn-lint` or AWS Console before deploy.
- Use Change Sets to preview modifications.
- Store templates in Git for version control.

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

---
***

# AWS Lex

## 1. Overview: What, Where, and Why

- **Problem it solves**: Creates conversational interfaces using voice and text.
- **Where it's used**: Chatbots, customer support, virtual assistants.
- **Why use it**: Uses the same technology as Amazon Alexa. Integrates with Lambda and other AWS services.

## 2. Steps to Set Up

1. Go to Amazon Lex → **Create Bot**.
2. Choose creation method: **Traditional**.
3. Provide Bot Name and Description.
4. IAM Permissions: Choose "Create a new role" or use an existing one.
5. Set Session Timeout.
6. Choose Language.
7. Select Voice Interaction or Text Based.
8. Click **Create Bot**.
9. After bot creation, click **Build** to deploy.
