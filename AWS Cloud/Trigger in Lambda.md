
Create EC2 instance

Create a role for Lambda
Role :   lambda-ec2-recovery-role
Policy: [AWSLambdaBasicExecutionRole] and [AmazonEC2FullAccess]

Instance ID : i-0712136ccc11178da
AMI ID : ami-0b93e28d1386a8580 


## Step 0: Prerequisites

Make sure you have:

- An AWS account
    
- At least **one EC2 instance** created
    
- The **Instance ID** (e.g., `i-0123456789abcdef0`)
    
- Region selected (Lambda and EC2 must be in the **same region**)
    

---

## Step 1: Create IAM Role for Lambda (Very Important)

Lambda **must have permission** to start EC2.

### 1.1 Create IAM Role

1. Go to **IAM → Roles → Create role**
    
2. Trusted entity: **AWS service**
    
3. Use case: **AWS Lambda**
    
4. Click **Next**
    

### 1.2 Attach Permissions

Attach these policies:

- `AWSLambdaBasicExecutionRole`
    
- Create a **custom policy** with this JSON:
    

`{   "Version": "2012-10-17",   "Statement": [     {       "Effect": "Allow",       "Action": [         "ec2:StartInstances",         "ec2:DescribeInstances"       ],       "Resource": "*"     }   ] }`

5. Name the role: `LambdaEC2StartRole`
    
6. Create role
    

---

## Step 2: Create Lambda Function (Start EC2)

### 2.1 Create Function

1. Go to **Lambda → Create function**
    
2. Choose **Author from scratch**
    
3. Function name: `StartEC2Instance`
    
4. Runtime: **Python 3.10**
    
5. Execution role: **Use existing role**
    
6. Select: `LambdaEC2StartRole`
    
7. Click **Create function**

## Step 3: Lambda Code (Python)

Replace the default code with this:

``` python
def lambda_handler(event, context):

print("Lambda triggered")

print("Event received:", json.dumps(event))

  

ec2 = boto3.client('ec2')

INSTANCE_ID = 'i-0712136ccc11178da'

  

try:

response = ec2.start_instances(

InstanceIds=[INSTANCE_ID]

)

print("StartInstances response:", response)

  

except Exception as e:

print("Error starting instance:", str(e))

raise e

  

return {

'statusCode': 200,

'body': f'EC2 instance {INSTANCE_ID} start initiated'

}

```

## Step 4: Test Lambda Manually (First Step Completed ✅)

### 4.1 Create Test Event

1. Click **Test**
    
2. Event name: `ManualTest`
    
3. Use default JSON (`{}`)
    
4. Click **Test**
    

### 4.2 Verify

- Go to **Amazon EC2**
    
- Your instance state should change from:
    
    `stopped → pending → running`
    

🎉 **Your first step is complete!**

## Step 5: Create Trigger (Auto Start When EC2 Stops)

Now let’s automate it.

## Step 6: Create EventBridge Rule

### 6.1 Go to EventBridge

1. Open **Amazon EventBridge**
    
2. Click **Create rule**
    
3. Name: `EC2StoppedTrigger`
    
4. Event bus: `default`
    
5. Rule type: **Rule with an event pattern**
    

---

### 6.2 Event Pattern (IMPORTANT)

Choose:

- Event source: **AWS services**
    
- AWS service: **EC2**
    
- Event type: **EC2 Instance State-change Notification**
    
- Specific state: **stopped**

### 6.3 Set Target

1. Target type: **AWS service**
    
2. Select **Lambda function**
    
3. Function: `StartEC2Instance`
    
4. Create rule

## Step 8: Final Test (End-to-End)

1. Go to **EC2**
    
2. Manually **Stop the instance**
    
3. Watch:

stopped → pending → running


![[Pasted image 20251221204055.png]]