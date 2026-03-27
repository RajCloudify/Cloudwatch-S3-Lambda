
# CloudWatch Logs to S3 Automation Using AWS Lambda

This project automates the process of exporting **Amazon CloudWatch Logs** to **Amazon S3** using an **AWS Lambda function** triggered by an **EventBridge Rule**. The Lambda function decodes and processes log data before storing it in the designated S3 bucket for archiving, analysis, or compliance purposes. This solution offers a scalable, serverless approach to automate log management with minimal maintenance effort.

For the full step-by-step implementation process, visit the detailed guide ----> (https://medium.com/@rajattingal1/serverless-automation-automate-cloudwatch-logs-to-s3-with-lambda-event-rules-%EF%B8%8F-9ab330ae70fc).

![Cloudwatch-Lambda-S3](https://github.com/user-attachments/assets/1baa6a5a-1d59-4e2f-8c31-85b76716d666)

# ☁️ CloudWatch → S3 → Lambda Pipeline

A serverless AWS pipeline that **exports CloudWatch logs to S3** and triggers a **Lambda function** for automated log processing and monitoring.

---

## 📌 Architecture Overview

```
CloudWatch Logs
      │
      ▼
  S3 Bucket  ──────►  Lambda Function
  (Log Storage)         (Processing / Alerts)
```

### Flow:
1. **CloudWatch** collects logs from AWS services / EC2 / applications
2. **Export Task** sends logs to an **S3 Bucket**
3. **S3 Event Trigger** invokes a **Lambda Function**
4. **Lambda** processes, filters, or forwards the logs

---

## 🛠️ Services Used

| Service | Purpose |
|---|---|
| **Amazon CloudWatch** | Log collection and monitoring |
| **Amazon S3** | Log storage and archival |
| **AWS Lambda** | Serverless log processing |
| **IAM** | Roles and permissions |

---

## 📁 Project Structure

```
Cloudwatch-S3-Lambda/
├── lambda/
│   └── lambda_function.py       # Lambda handler code
├── policies/
│   ├── lambda-role-policy.json  # IAM policy for Lambda
│   └── s3-bucket-policy.json    # S3 bucket policy
├── screenshots/
│   └── architecture.png         # Architecture diagram
└── README.md
```

---

## 🚀 Setup & Deployment

### Prerequisites
- AWS Account
- AWS CLI configured (`aws configure`)
- Python 3.x

---

### Step 1: Create S3 Bucket

```bash
aws s3 mb s3://your-cloudwatch-logs-bucket --region us-east-1
```

---

### Step 2: Set S3 Bucket Policy

Attach the bucket policy from `policies/s3-bucket-policy.json`:

```bash
aws s3api put-bucket-policy \
  --bucket your-cloudwatch-logs-bucket \
  --policy file://policies/s3-bucket-policy.json
```

---

### Step 3: Create IAM Role for Lambda

```bash
aws iam create-role \
  --role-name CloudWatch-S3-Lambda-Role \
  --assume-role-policy-document file://policies/lambda-role-policy.json
```

Attach permissions:

```bash
aws iam attach-role-policy \
  --role-name CloudWatch-S3-Lambda-Role \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

aws iam attach-role-policy \
  --role-name CloudWatch-S3-Lambda-Role \
  --policy-arn arn:aws:iam::aws:policy/CloudWatchLogsFullAccess
```

---

### Step 4: Deploy Lambda Function

Zip and upload the Lambda function:

```bash
cd lambda
zip function.zip lambda_function.py

aws lambda create-function \
  --function-name CloudWatch-S3-Processor \
  --runtime python3.12 \
  --role arn:aws:iam::<ACCOUNT_ID>:role/CloudWatch-S3-Lambda-Role \
  --handler lambda_function.lambda_handler \
  --zip-file fileb://function.zip \
  --region us-east-1
```

---

### Step 5: Add S3 Trigger to Lambda

```bash
aws lambda add-permission \
  --function-name CloudWatch-S3-Processor \
  --statement-id s3-trigger \
  --action lambda:InvokeFunction \
  --principal s3.amazonaws.com \
  --source-arn arn:aws:s3:::your-cloudwatch-logs-bucket
```

Then configure the S3 bucket notification to trigger Lambda on object creation.

---

### Step 6: Export CloudWatch Logs to S3

```bash
aws logs create-export-task \
  --log-group-name "/aws/lambda/your-log-group" \
  --from 1609459200000 \
  --to 1609545600000 \
  --destination your-cloudwatch-logs-bucket \
  --destination-prefix "cloudwatch-exports/"
```

---

## ⚙️ Lambda Function

```python
import json
import boto3
import gzip

def lambda_handler(event, context):
    s3 = boto3.client('s3')
    
    # Get bucket and file info from the S3 event
    bucket = event['Records'][0]['s3']['bucket']['name']
    key    = event['Records'][0]['s3']['object']['key']
    
    print(f"Processing log file: s3://{bucket}/{key}")
    
    # Download and read the log file
    response = s3.get_object(Bucket=bucket, Key=key)
    content  = gzip.decompress(response['Body'].read()).decode('utf-8')
    
    # Process logs
    for line in content.splitlines():
        print(line)  # Extend: send to SNS, filter errors, push to DB, etc.
    
    return {
        'statusCode': 200,
        'body': json.dumps('Logs processed successfully!')
    }
```

---

## 🔐 IAM Permissions Required

Lambda needs the following permissions:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::your-cloudwatch-logs-bucket",
        "arn:aws:s3:::your-cloudwatch-logs-bucket/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "*"
    }
  ]
}
```

---

## 📊 Monitoring

- View Lambda logs in **CloudWatch → Log Groups → /aws/lambda/CloudWatch-S3-Processor**
- Set up **CloudWatch Alarms** for Lambda errors or timeouts
- Enable **S3 Server Access Logging** for audit trails

---

## 🧹 Cleanup

To avoid unnecessary AWS charges:

```bash
# Delete Lambda function
aws lambda delete-function --function-name CloudWatch-S3-Processor

# Empty and delete S3 bucket
aws s3 rm s3://your-cloudwatch-logs-bucket --recursive
aws s3 rb s3://your-cloudwatch-logs-bucket

# Delete IAM role
aws iam detach-role-policy --role-name CloudWatch-S3-Lambda-Role --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
aws iam delete-role --role-name CloudWatch-S3-Lambda-Role
```

---

## 👨‍💻 Author

**RajCloudify**
- GitHub: [@RajCloudify](https://github.com/RajCloudify)

---

## 📄 License

This project is licensed under the MIT License.

---

> ⭐ If this project helped you, please give it a star on GitHub!
