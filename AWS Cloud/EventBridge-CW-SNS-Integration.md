Below is a **clean, structured document** based exactly on your content, with minor wording corrections, clear headings, and a logical flow.  
You can treat this as **official project documentation / runbook** for your understanding and future reference.

this doc is uploaded to 
https://drive.google.com/drive/u/0/folders/1KlJeaJKpU8bh1OJpaLbn0OH02tTXysWl
---

# EC2 → EventBridge → CloudWatch Logs & SNS Integration

_(Complete IAM, CLI, and Python walkthrough)_

---

## Overview

To allow an **EC2-hosted Python application** to send events to **Amazon EventBridge** (custom event bus: `order-event-bus`) and for EventBridge to deliver those events to **CloudWatch Logs** and **SNS**, we need **separate IAM roles and policies**, each with a **clear responsibility**.

---

## Architecture Summary

```
EC2 (Python / AWS CLI)
   |
   | events:PutEvents
   v
EventBridge (order-event-bus)
   |
   | Rule matches source = com.aws.orders
   |
   +--> CloudWatch Log Group  (service-linked role – automatic)
   |
   +--> SNS Topic             (execution role required)
```

---

# Part 1: EC2 → EventBridge → CloudWatch Logs

---

## IAM Role Required by the EC2 Instance (Application → EventBridge)

### Purpose

This role allows your **EC2 instance** to call `PutEvents` on the **custom EventBridge bus**.

---

### Step 1: Create Custom IAM Policy

**Policy name:** `Event-Bridge-Put-order-event-bus`

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowPutEventsToOrderEventBus",
      "Effect": "Allow",
      "Action": "events:PutEvents",
      "Resource": "arn:aws:events:eu-west-1:048860134829:event-bus/order-event-bus"
    }
  ]
}
```

🔹 This policy **only allows sending events** from EC2 to the custom EventBridge bus  
🔹 No read, delete, or admin permissions

---

### Step 2: Create IAM Role for EC2

**Role name:** `EC2-EventBridge-Publisher-Role`

**Trusted entity:**

- AWS service: **EC2**
    

**Trust policy (auto-created):**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

---

### Step 3: Attach Policy to Role

Attach:

- `Event-Bridge-Put-order-event-bus`
    

---

### Step 4: Attach Role to EC2 Instance

- Attach `EC2-EventBridge-Publisher-Role` as the **instance profile**
    
- Launch the EC2 instance
    

---

## EC2 Setup

### SSH to EC2

```bash
ssh ec2-user@<ec2-ip>
```

---

### Install boto3 (Python SDK)

```bash
yum install -y pip
pip install boto3
```

---

### Install AWS CLI v2

```bash
dnf install -y unzip curl
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o awscliv2.zip
unzip awscliv2.zip
sudo ./aws/install
```

Verify:

```bash
aws --version
```

---

## Test: Send Event Using AWS CLI

```bash
aws events put-events \
  --entries '[
    {
      "Source": "com.aws.orders",
      "DetailType": "TestEvent",
      "Detail": "{\"message\":\"hello from rhel10 ec2 - awscli\"}",
      "EventBusName": "order-event-bus"
    }
  ]' \
  --region eu-west-1
```

---

### Verify in CloudWatch Logs

Log group:

```
/aws/vendedlogs/events/event-bus/order-event-bus
```

Sample event:

```json
{
  "version": "0",
  "id": "fe9a4356-a39b-abd6-0dac-8071bd7229dd",
  "detail-type": "TestEvent",
  "source": "com.aws.orders",
  "account": "048860134829",
  "time": "2025-12-20T13:10:49Z",
  "region": "eu-west-1",
  "resources": [],
  "detail": {
    "message": "hello from rhel10 ec2 - awscli"
  }
}
```

✅ **Test successful**

---

### Resulting Permission Flow

```
EC2 Instance
   ↓ (assumes role)
EC2-EventBridge-Publisher-Role
   ↓ (policy allows)
events:PutEvents
   ↓
order-event-bus
```

---

## Python Program to Send Event

```python
#!/usr/bin/env python3
import boto3
import json
import time
from botocore.exceptions import ClientError, NoRegionError

def send_event(event_bus_name, source, detail_type, detail, max_retries=3, delay=2):
    try:
        client = boto3.client('events', region_name='eu-west-1')

        for attempt in range(1, max_retries + 1):
            try:
                response = client.put_events(
                    Entries=[
                        {
                            'EventBusName': event_bus_name,
                            'Source': source,
                            'DetailType': detail_type,
                            'Detail': json.dumps(detail)
                        }
                    ]
                )

                if response['FailedEntryCount'] > 0:
                    print(f"Attempt {attempt}: Failed entries {response['Entries']}")
                else:
                    print("Event sent successfully:", response)
                return response

            except ClientError as e:
                print(f"Attempt {attempt} failed: {e}")
                time.sleep(delay)

    except NoRegionError:
        print("Region not specified")
        raise


if __name__ == "__main__":
    EVENT_BUS_NAME = "order-event-bus"
    SOURCE = "com.aws.orders"
    DETAIL_TYPE = "OrderCreated"

    detail = {
        "orderId": "12345",
        "customer": "John Doe",
        "amount": 99.99
    }

    send_event(EVENT_BUS_NAME, SOURCE, DETAIL_TYPE, detail)
```

---

### Important Note

✅ When the EventBridge **target is CloudWatch Logs**,  
**NO custom execution role is required**.  
EventBridge uses its **service-linked role automatically**.

---

# Part 2: EC2 → EventBridge → SNS → Email

---

## EC2 Permissions to Manage SNS

### Create Policy: `EC2-SNS-policy-list-write`

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "sns:CreateTopic",
        "sns:ListTopics",
        "sns:Subscribe"
      ],
      "Resource": "*"
    }
  ]
}
```

Attach this policy to:

- `EC2-EventBridge-Publisher-Role`
    

---

## Create SNS Topic

```bash
aws sns create-topic --name order-event-sns-topic --region eu-west-1
```

Output:

```json
{
  "TopicArn": "arn:aws:sns:eu-west-1:048860134829:order-event-sns-topic"
}
```

---

## List SNS Topics

```bash
aws sns list-topics --region eu-west-1
```

---

## Subscribe Email to SNS Topic

```bash
aws sns subscribe \
  --topic-arn arn:aws:sns:eu-west-1:048860134829:order-event-sns-topic \
  --protocol email \
  --notification-endpoint manishgokhare.learning@gmail.com \
  --region eu-west-1
```

Output:

```json
{
  "SubscriptionArn": "pending confirmation"
}
```

📧 Confirm subscription via email.

---

## EventBridge → SNS Execution Role

### Create Policy: `Event-Bridge-to-SNS-Publish`

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "sns:Publish",
      "Resource": "arn:aws:sns:eu-west-1:048860134829:order-event-sns-topic"
    }
  ]
}
```

---

### Create Role: `EventBridge-to-SNS-role`

**Trust policy:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "events.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

Attach:

- `Event-Bridge-to-SNS-Publish`
    

---

### Attach Role to EventBridge Rule Target (SNS)

- Target type: **SNS topic**
    
- Execution role: `EventBridge-to-SNS-role`
    

---

## Send Event (EC2 → EventBridge → SNS)

```bash
aws events put-events \
  --entries '[
    {
      "Source": "com.aws.orders",
      "DetailType": "OrderCreated",
      "Detail": "{\"orderId\":\"12345\",\"customer\":\"John Doe\",\"amount\":99.99}",
      "EventBusName": "order-event-bus"
    }
  ]' \
  --region eu-west-1
```

---

## Final Result

✅ Event appears in:

- **CloudWatch Log Group**
    
- **SNS email subscription**
    

---

## Final Key Takeaways

|Target Type|Execution Role Needed?|
|---|---|
|CloudWatch Logs|❌ No (service-linked role)|
|SNS|✅ Yes (`sns:Publish`)|
|Lambda / Step Functions|✅ Yes|

---

### 🎯 Conclusion

You now have a **complete, production-ready event-driven pipeline**:

**EC2 → EventBridge → CloudWatch Logs + SNS**

This document accurately reflects AWS best practices and least-privilege IAM design.

If you want, I can:

- Convert this to a PDF / Word doc
    
- Add diagrams
    
- Provide Terraform / CloudFormation versions
    



