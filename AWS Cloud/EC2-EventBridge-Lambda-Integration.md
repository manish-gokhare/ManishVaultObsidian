![[Screenshot 2025-12-21 at 15.21.54.png]]


```php
EC2 (terminate event)
        ↓
EventBridge rule
        ↓
Lambda function
        ↓
EC2 (launch new instance)

```


**EC2 does NOT directly call Lambda**  
**EventBridge is the trigger**  
**Lambda must be configured with EventBridge as its trigger**

Lets first build the component seperately.

Lets build the lambda function to start the EC2 instance.

# ✅ STEP 1: Confirm EC2 Instance

You already have an EC2 instance:

- **Instance ID**: `i-09596cb76a4dae1ed`
    

	Nothing else to configure on EC2.

# ✅ STEP 2: Create Lambda Execution Role

### IAM → Roles → Create role > lambda-ec2-recovery-role

- Trusted entity: **Lambda**
    
- Permissions (attach policy): or attach full-ec2-permission.



```
{
  "Effect": "Allow",
  "Action": [
    "ec2:RunInstances",
    "ec2:DescribeInstances"
  ],
  "Resource": "*"
}

```

Also add -
AWSLambdaBasicExecutionRole policy to lambda-ec2-recovery-role.

So there will be two policies:
Full-EC2-Access
AWSLambdaBasicExecutionRole

# ✅ STEP 3: Create Lambda Function

### Lambda → Create function

- Name: `ec2-auto-recovery`
    
- Runtime: **Python 3.10**
    
- Execution role: **Use existing**
    
- Select: `lambda-ec2-recovery-role`

Oncel lambda function - is created change the lambda timeout from default 3 sec to 2 mins.

```python
import boto3
ec2 = boto3.client("ec2")
def lambda_handler(event, context):

instance_id = event["detail"]["instance-id"]

state = event["detail"]["state"]

if instance_id == "i-09596cb76a4dae1ed" and state == "terminated":

response = ec2.run_instances(

ImageId="ami-0b93e28d1386a8580", # replace with valid AMI

InstanceType="t2.micro",

MinCount=1,

MaxCount=1

)

  

return {

"message": "New EC2 instance launched",

"new_instance_id": response["Instances"][0]["InstanceId"]

}


return {
"message": "Event ignored"
}
```


# ✅ STEP 4: Create EventBridge Rule (THIS IS THE TRIGGER)

### EventBridge → Rules → Create rule

- Name: `ec2-terminate-trigger`
    
- Event bus: **default**
    
- Rule type: **Event pattern**

```
{
  "source": ["aws.ec2"],
  "detail-type": ["EC2 Instance State-change Notification"],
  "detail": {
    "state": ["terminated"],
    "instance-id": ["i-09596cb76a4dae1ed"]
  }
}

```
## Step-by-Step (Exact UI Choices)

In **EventBridge → Rules → Create rule → Event pattern**:

### 1️⃣ Event source

- ✅ **AWS service**
    

### 2️⃣ AWS service

- Select: **EC2**
    

### 3️⃣ Event type

- Select: **EC2 Instance State-change Notification**
    

### 4️⃣ Specific state

- Check: **terminated**
    

### 5️⃣ (Optional but recommended) Specific instance

- Instance ID:

# ✅ STEP 5: Attach Lambda as Target

In the **same rule**:

- Target type: **Lambda**
    
- Select: `ec2-auto-recovery`
    

👉 Save the rule.

Terminate the Ec2 instance > Even bridge send the event to lambda .Lambda start the new ec2 instance.
Check the logs in CloudWatch or you can see the new instance is getting created..