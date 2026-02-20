AWS EventBridge is a serverless event bus service from Amazon Web Services (AWS) that enables event-driven architectures by routing events from sources like AWS services, custom apps, and third-party SaaS applications to targets such as Lambda functions or SQS queues.

AWS EventBridge event buses function as brokers by receiving events from multiple sources—such as AWS services, custom applications, or SaaS partners—and routing them intelligently to one or more targets based on predefined rules, decoupling producers and consumers in an event-driven system

Event bus does the filtering based on the p

Filtering occurs via event patterns in rules, which match specific event attributes like source, detail-type, or JSON fields (e.g., EC2 state changes to “running”).

To allow your **EC2-hosted Python application** to send events to **Amazon EventBridge** (custom event bus: `order-event-bus`) and for EventBridge to deliver those events to **CloudWatch Logs**, you need **two separate IAM roles/policies**, each with a clear responsibility.

#### IAM Role Required by the EC2 Instance (Application → EventBridge)

**Purpose**

This role allows your **EC2 instance** to call `PutEvents` on the **custom EventBridge bus**.

1) Create custom IAM Policy - Event-Bridge-Put-order-event-bus

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

This policy **only allows sending events**  from EC2 to your custom EventBridge bus  (order-event-bus) — nothing more.

2) Create a role - EC2-EventBridge-Publisher-Role

**Trusted entity (important):**

- AWS service
    
- **EC2**
    
- Trust policy automatically becomes:
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

Attach policy - "Event-Bridge-Put-order-event-bus" to   a role - "EC2-EventBridge-Publisher-Role"

Attach a role - EC2-EventBridge-Publisher-Role to EC2 instance as a instance profile.

Launch EC2 instance 
SSH to EC2.
To send the events via python we need boto3.
[root@ip-172-31-26-186 ~]# yum install pip
[root@ip-172-31-26-186 ~]# pip install boto3


Install AWS CLI.

Send the event from AWS CLI.
```bash
aws events put-events \
  --entries '[
    {
      "Source": "com.aws.orders",
      "DetailType": "TestEvent",
      "Detail": "{\"message\":\"hello from rhel10 ec2 - awscli\"}",
      "EventBusName": "order-event-bus"
    }
  ]'
```

Verify the log group in cloudWatch I am able to see. Test is success.

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

**Resulting permission flow:**

EC2 Instance
   ↓ (assumes role)
EC2-EventBridge-Publisher-Role
   ↓ (policy allows)
events:PutEvents
   ↓
order-event-bus

Python program to send the event :

```
#!/usr/bin/env python3
import boto3
import json
import time
from botocore.exceptions import ClientError, NoRegionError

def send_event(event_bus_name, source, detail_type, detail, max_retries=3, delay=2):
    """
    Send a single event to EventBridge with retries.
    
    Parameters:
    - event_bus_name: str, name of the EventBridge bus
    - source: str, event source (must match rule)
    - detail_type: str, descriptive type of event
    - detail: dict, payload
    - max_retries: int, number of retry attempts
    - delay: int, seconds between retries
    """
    try:
        client = boto3.client('events', region_name='eu-west-1')  # <-- your region

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
                    print(f"Attempt {attempt}: Some events failed: {response['Entries']}")
                else:
                    print(f"Event sent successfully: {response}")
                return response

            except ClientError as e:
                print(f"Attempt {attempt} failed: {e}")
                if attempt < max_retries:
                    print(f"Retrying in {delay} seconds...")
                    time.sleep(delay)
                else:
                    raise

    except NoRegionError:
        print("Error: No AWS region specified.")
        raise
    except Exception as e:
        print(f"Unexpected error: {e}")
        raise


if __name__ == "__main__":
    # Configuration
    EVENT_BUS_NAME = "order-event-bus"  # Your custom event bus
    SOURCE = "com.aws.orders"           # Must match EventBridge rule
    DETAIL_TYPE = "OrderCreated"
    
    # Example payload
    detail = {
        "orderId": "12345",
        "customer": "John Doe",
        "amount": 99.99
    }
    
    send_event(EVENT_BUS_NAME, SOURCE, DETAIL_TYPE, detail)

```

Event Posted on Cloud-watch log.

So when the target is "Cloudwatch log group" . I am role is not required for EventBridge.


##### How to send the event to SNS topic from EC2 AWS CLI 


- Create a policy : EC2-SNS-policy-list-write and attach to the EC2 role -EC2-EventBridge-Publisher-Role

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

- Create a top in SNS - 

```
[ec2-user@ip-172-31-26-186 ~]$ aws sns create-topic --name order-event-sns-topic

{

    "TopicArn": "arn:aws:sns:eu-west-1:048860134829:order-event-sns-topic"

}


```

- List SNS Topic
```
[ec2-user@ip-172-31-26-186 ~]$ aws sns list-topics --region eu-west-1

{

    "Topics": [

        {

            "TopicArn": "arn:aws:sns:eu-west-1:048860134829:Default_CloudWatch_Alarms_Topic"

        },

        {

            "TopicArn": "arn:aws:sns:eu-west-1:048860134829:order-event-sns-topic"

        },

        {

            "TopicArn": "arn:aws:sns:eu-west-1:048860134829:test-topic.fifo"

        }

    ]

}
```

Need to subscribe this topic - update a policy and add the action Subscribe
```
aws sns subscribe \
    --topic-arn arn:aws:sns:eu-west-1:048860134829:order-event-sns-topic \
    --protocol email \
    --notification-endpoint manishgokhare.learning@gmail.com \
    --region eu-west-1
```

```
[ec2-user@ip-172-31-26-186 ~]$ aws sns subscribe \

    --topic-arn arn:aws:sns:eu-west-1:048860134829:order-event-sns-topic \

    --protocol email \

    --notification-endpoint manishgokhare.learning@gmail.com \

    --region eu-west-1

{

    "SubscriptionArn": "pending confirmation"

}
```

- Create a policy for eventbridge to SNS to publish the events - Event-Bridfe-to-sns-publish

```
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

ASSign this policy to role - EventBridge-to-SNS-role

Attach this role to the EventBridge target which SNS.

```
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

We should get this event on SNS and Cloudwatch log group.
#### IAM Role Required by EventBridge (EventBridge → CloudWatch Logs)

Allows **EventBridge** to write matching events to the **CloudWatch Log Group**:
log group name : /aws/vendedlogs/events/event-bus/order-event-bus

Create aa policy - event-bridge-cloudwatch-log
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:eu-west-1:048860134829:log-group:/aws/vendedlogs/events/event-bus/order-event-bus:*"
    }
  ]
}

```



Create a Role: EventBridge-To-CloudWatchLogs-Role

Create a role using Custom trust policy
Use this trust policy
```
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
attach policy event-bridge-cloudwatch-log to role -  EventBridge-To-CloudWatchLogs-Role.


