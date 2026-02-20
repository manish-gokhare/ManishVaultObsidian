Project Goal: Trigger a Lambda function automatically when a file is uploaded to S3 and log file details.

**Flow:**

1. User uploads a file to **Amazon S3**
    
2. S3 sends an event notification
    
3. **AWS Lambda** is triggered
    
4. Lambda processes the file and logs output to CloudWatch

## Step 1: Create an S3 Bucket

1. Go to **S3 → Create bucket**
    
2. Bucket name: `s3-lambda-mini-project-01
    
3. Region: same as Lambda
    
4. Keep default settings → Create bucket

## Step 2: Create IAM Role for Lambda

Lambda needs permission to:

- Read S3 objects
    
- Write CloudWatch logs
Role : Lambda-S3-CW-Role

## Step 3: Create Lambda Function

1. Go to **Lambda → Create function**
    
2. Function name: `S3FileUploadProcessor`
    
3. Runtime: **Python 3.10**
    
4. Execution role: select the IAM role created above

Step 4: Lambda Function Code

```
import json
import boto3

s3 = boto3.client('s3')

def lambda_handler(event, context):
    # Log entire event
    print("Received event:", json.dumps(event, indent=2))

    # Extract bucket & object key
    bucket_name = event['Records'][0]['s3']['bucket']['name']
    object_key = event['Records'][0]['s3']['object']['key']

    # Use boto3 to get object metadata
    response = s3.head_object(
        Bucket=bucket_name,
        Key=object_key
    )

    file_size = response['ContentLength']
    content_type = response.get('ContentType', 'Unknown')
    last_modified = str(response['LastModified'])

    print("File Details:")
    print(f"Bucket Name : {bucket_name}")
    print(f"Object Key : {object_key}")
    print(f"File Size  : {file_size} bytes")
    print(f"Type       : {content_type}")
    print(f"Modified   : {last_modified}")

    return {
        'statusCode': 200,
        'body': 'S3 file metadata processed successfully'
    }

```

## Step 5: Configure S3 Trigger

1. Open your **Lambda function**
    
2. Click **Add trigger**
    
3. Trigger type: **S3**
    
4. Bucket: your S3 bucket
    
5. Event type: **PUT (Object Created)**
    
6. Prefix (optional): `uploads/`
    
7. Acknowledge permission → Add trigger


## Step 6: Test the Integration

1. Upload any file to your S3 bucket
    
2. Go to **CloudWatch → Log groups**
    
3. Open `/aws/lambda/S3FileUploadProcessor`
    
4. You should see logs like:

![[Screenshot 2025-12-22 at 23.13.47.png]]