# Serverless Image Recognition with AWS Lambda & Step Functions

## Overview
This project implements a **serverless image recognition system** using AWS Lambda, Step Functions, and Amazon Rekognition. It continuously monitors an S3 bucket for new image uploads, analyzes them, and stores the results in another S3 bucket.

## Architecture
1. **Image Upload**: Users upload images to an S3 source bucket.
2. **Event Trigger**: S3 Event Notification triggers an AWS Lambda function.
3. **Step Function Execution**: The Lambda function starts an AWS Step Function.
4. **Image Recognition**: Amazon Rekognition processes the image.
5. **Storage**: Recognition results are stored in a destination S3 bucket.
6. **Monitoring**: AWS CloudWatch logs the execution and alerts failures via SNS.

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
