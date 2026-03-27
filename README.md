# ☁️ CloudWatch Logs to S3 Automation Using AWS Lambda

This project automates the process of exporting **Amazon CloudWatch Logs** to **Amazon S3** using an **AWS Lambda function** triggered by an **EventBridge Rule**. The Lambda function decodes and processes log data before storing it in the designated S3 bucket for archiving, analysis, or compliance purposes. This solution offers a scalable, serverless approach to automate log management with minimal maintenance effort.

For the full step-by-step implementation process, visit the detailed guide ----> [Medium Article](https://medium.com/@rajattingal1/serverless-automation-automate-cloudwatch-logs-to-s3-with-lambda-event-rules-%EF%B8%8F-9ab330ae70fc)

![Cloudwatch-Lambda-S3](https://github.com/user-attachments/assets/1baa6a5a-1d59-4e2f-8c31-85b76716d666)

---

## 📌 Architecture Overview

```
CloudWatch Logs
      │
      ▼
 EventBridge Rule
      │
      ▼
  Lambda Function
      │
      ▼
  S3 Bucket
  (Log Storage)
```

### Flow:
1. **CloudWatch** collects logs from AWS services / EC2 / applications
2. **EventBridge Rule** triggers the **Lambda Function** on a schedule or event
3. **Lambda** decodes and processes the log data
4. **S3 Bucket** stores the processed logs for archiving, analysis, or compliance

---

## 🛠️ Services Used

| Service | Purpose |
|---|---|
| **Amazon CloudWatch** | Log collection and monitoring |
| **Amazon EventBridge** | Scheduled rule to trigger Lambda |
| **AWS Lambda** | Serverless log processing & export |
| **Amazon S3** | Log storage and archival |
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

aws iam attach-role-policy \
  --role-name CloudWatch-S3-Lambda-Role \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess

aws iam attach-role-policy \
  --role-name CloudWatch-S3-Lambda-Role \
  --policy-arn arn:aws:iam::aws:policy/CloudWatchLogsFullAccess
```

---

### Step 4: Deploy Lambda Function

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

### Step 5: Create EventBridge Rule

```bash
aws events put-rule \
  --name CloudWatch-Log-Export-Rule \
  --schedule-expression "rate(1 day)" \
  --state ENABLED

aws events put-targets \
  --rule CloudWatch-Log-Export-Rule \
  --targets "Id=1,Arn=arn:aws:lambda:us-east-1:<ACCOUNT_ID>:function:CloudWatch-S3-Processor"
```

---

## ⚙️ Lambda Function

```python
import json
import boto3
import gzip
import base64

def lambda_handler(event, context):
    s3 = boto3.client('s3')

    # Decode log data from CloudWatch via EventBridge
    log_data = event.get('awslogs', {}).get('data', '')
    if log_data:
        decoded = gzip.decompress(base64.b64decode(log_data)).decode('utf-8')
        log_events = json.loads(decoded)
        print("Log Events:", log_events)

    # Store processed logs in S3
    bucket = 'your-cloudwatch-logs-bucket'
    key = 'cloudwatch-exports/logs.json'

    s3.put_object(
        Bucket=bucket,
        Key=key,
        Body=json.dumps(log_events),
        ContentType='application/json'
    )

    return {
        'statusCode': 200,
        'body': json.dumps('Logs exported to S3 successfully!')
    }
```

---

## 🔐 IAM Permissions Required

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
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
        "logs:PutLogEvents",
        "logs:CreateExportTask"
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

```bash
# Delete Lambda function
aws lambda delete-function --function-name CloudWatch-S3-Processor

# Delete EventBridge rule
aws events remove-targets --rule CloudWatch-Log-Export-Rule --ids 1
aws events delete-rule --name CloudWatch-Log-Export-Rule

# Empty and delete S3 bucket
aws s3 rm s3://your-cloudwatch-logs-bucket --recursive
aws s3 rb s3://your-cloudwatch-logs-bucket

# Delete IAM role
aws iam detach-role-policy --role-name CloudWatch-S3-Lambda-Role --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess
aws iam delete-role --role-name CloudWatch-S3-Lambda-Role
```

---

## 👨‍💻 Author

**RajCloudify**
- GitHub: [@RajCloudify](https://github.com/RajCloudify)
- Medium: [@rajattingal1](https://medium.com/@rajattingal1)

---

## 📄 License

This project is licensed under the MIT License.

---

> ⭐ If this project helped you, please give it a star on GitHub!
