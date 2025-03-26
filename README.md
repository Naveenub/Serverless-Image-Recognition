# Serverless Image Recognition with AWS Lambda & Step Functions

## 🚀 Project Overview

Serverless Image Recognition with AWS Lambda & Step Functions

This project implements an event-driven serverless image recognition system using AWS Lambda, Step Functions, and Amazon Rekognition. The system continuously monitors an S3 bucket for new image uploads, processes them using Amazon Rekognition, and stores the results in another S3 bucket for further analysis.

🛠️ Technologies Used


✅ AWS Lambda - Serverless computing for event-driven execution

✅ AWS Step Functions - Orchestration of image processing workflow

✅ Amazon S3 - Storage for source images and processed results

✅ Amazon Rekognition - AI-based image recognition service

✅ AWS IAM - Secure role-based access control

✅ AWS CloudWatch - Monitoring and logging execution

✅ AWS SNS (Optional) - Real-time notifications on process completion

✨ Features


🚀 Fully Serverless - No need to manage servers, scales automatically

📸 Automated Image Recognition - Detects objects, labels, and faces in images
⚡ Event-Driven Processing - Triggers on new image uploads to S3
🔄 Step Functions Workflow - Efficiently manages processing logic
📂 Results Storage in S3 - Stores image analysis results in a separate bucket
📊 Logging & Monitoring - Tracks execution flow using CloudWatch
🔔 Real-time Notifications - (Optional) Sends alerts using SNS

## 📷 Architecture Diagram


![AWS CI/CD Architecture](https://files.oaiusercontent.com/file-Dtg87Q9iHbhKSRGJR7JF58?se=2025-03-26T17%3A49%3A13Z&sp=r&sv=2024-08-04&sr=b&rscc=max-age%3D604800%2C%20immutable%2C%20private&rscd=attachment%3B%20filename%3Df7aad705-4975-433b-8da3-2aa0dd1a900d.webp&sig=QYepXnByLzxgVkiy3Fkm33u5dtpogCYgmVjUYLoL9sk%3D)

## Prerequisites

- An **AWS account**
- AWS **CLI installed** (optional)
- AWS **IAM permissions** to create and manage S3, Lambda, Step Functions, and Rekognition

## Deployment Steps

### 1. Create Two S3 Buckets
- **Source Bucket** (e.g., `image-upload-bucket`)
- **Destination Bucket** (e.g., `image-result-bucket`)

Using AWS CLI:
```sh
aws s3 mb s3://image-upload-bucket
aws s3 mb s3://image-result-bucket
```

### 2. Create an IAM Role for Lambda & Step Functions
- Go to **AWS IAM** → Create **Role** → Select **AWS Service** → Choose **Lambda**.
- Attach policies:
  - `AmazonS3FullAccess`
  - `AWSLambdaBasicExecutionRole`
  - `AWSStepFunctionsFullAccess`
  - `AmazonRekognitionFullAccess`
- Name the role **LambdaStepFunctionRole**.

### 3. Create AWS Lambda Functions
- **S3 Trigger Function** (Invokes Step Function)
- **Image Recognition Function** (Uses Amazon Rekognition)
- **Store Results Function** (Stores results in S3)

#### S3 Trigger Lambda Code (`s3_trigger.py`):
```python
import json
import boto3

def lambda_handler(event, context):
    sfn = boto3.client('stepfunctions')
    for record in event['Records']:
        bucket = record['s3']['bucket']['name']
        image_key = record['s3']['object']['key']
        input_payload = {"bucket": bucket, "image_key": image_key}
        
        response = sfn.start_execution(
            stateMachineArn="arn:aws:states:<REGION>:<ACCOUNT_ID>:stateMachine:ImageRecognitionStateMachine",
            input=json.dumps(input_payload)
        )
        return {'statusCode': 200, 'body': 'Step Function Started!'}
```

### 4. Set Up S3 Event Notifications
- **S3 Console** → **image-upload-bucket** → **Properties** → **Event Notifications**.
- Create a new event with the **PUT** event and select `S3ImageTrigger` Lambda.

### 5. Create the AWS Step Function
Define the **state machine**:
```json
{
  "StartAt": "ImageRecognition",
  "States": {
    "ImageRecognition": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:<REGION>:<ACCOUNT_ID>:function:ImageRecognitionFunction",
      "Next": "StoreResults"
    },
    "StoreResults": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:<REGION>:<ACCOUNT_ID>:function:StoreResultsFunction",
      "End": true
    }
  }
}
```

### 6. Deploy & Test
1. Upload an image to `image-upload-bucket`.
2. Monitor **Step Functions** execution.
3. Check `image-result-bucket` for stored results.

Using AWS CLI:
```sh
aws s3 cp test-image.jpg s3://image-upload-bucket/
```

## Future Enhancements
- Extend to support **multiple image types**.
- Improve **error handling** and retry mechanisms.
- Integrate with **SNS for real-time notifications**.

---
### 📌 Contributing
Feel free to fork, open issues, or submit PRs to improve this project!

### 📜 License
This project is licensed under the **Apache License**.

🚀 Happy Coding!
