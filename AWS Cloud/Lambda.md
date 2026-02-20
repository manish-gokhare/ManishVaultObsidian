
## 1️⃣ What is AWS Lambda? (Concept First)

### Simple Definition

AWS Lambda is a **serverless compute service** that lets you:

- Run code **without managing servers**
    
- Automatically scale
    
- Pay **only when your code runs**
    

👉 You upload code → AWS runs it → AWS stops it.


|Traditional Server|AWS Lambda|
|---|---|
|You manage servers|No servers|
|Always running|Runs only on events|
|Pay for uptime|Pay per execution|
|Manual scaling|Auto scaling|

Lambda is:

- **Event-driven**
    
- **Stateless**
    
- **Short-lived**
    

Lambda **does nothing by itself** — it runs **only when an event happens**.

## 2️⃣ Core Lambda Components (Must Understand)

Every Lambda function has **4 core parts**:

Event → Lambda Function → Execution → Response

## 3️⃣ Runtime (What Language Runs Your Code)

### What is a Runtime?

A **runtime** is the programming language environment Lambda uses to execute your code.

Examples:

- Python 3.9 / 3.10 / 3.11
    
- Node.js
    
- Java
    
- Go
    
**Important:**

- You write Python
    
- AWS manages Python installation
    

You don’t install Python on servers — AWS already has it.

## 4️⃣ Handler (The Entry Point of Lambda)

### What is a Handler?

The **handler** is the **function AWS Lambda calls first** when your Lambda runs.

👉 Think of it as:

> “This is where execution starts.”



### Handler Format (Python)

`file_name.function_name`

Example:

`lambda_function.lambda_handler`

Meaning:

- File name: `lambda_function.py`
- Function name: `lambda_handler`

When you created a Lambda function in the AWS Console:

- **Function name** (AWS resource):  
    👉 `my_first_lambda`
    

⚠️ IMPORTANT  
This is **NOT** the handler name.

This is just the **Lambda resource name** in AWS.

##  Actual Python File (FILE NAME)

In the Lambda **Code** section, you see a file named:

`lambda_function.py`

👉 This is your **Python file name**.

## The Python Function (FUNCTION NAME)

Inside `lambda_function.py`, you see this code:

`def lambda_handler(event, context):     return "Hello from Lambda"`

Here:

- Python function name = `lambda_handler`



Think of Lambda like this:


my_first_lambda   (AWS Lambda resource)
│
└── lambda_function.py   ← FILE
     │
     └── lambda_handler() ← FUNCTION


AWS does **not** care about the Lambda resource name for execution.  
It only cares about the **handler path**.


## 5️⃣ Event (Input to Lambda)

### What is `event`?

`event` is the **input data** sent to Lambda.

It can come from:

- API Gateway (HTTP request)
    
- S3 (file upload)
    
- DynamoDB (DB change)
    
- Manual test input
	```JSON
{
  "name": "Lambda"
}
```

## 6️⃣ Context (Metadata About Execution)

### What is `context`?

`context` is information **about the Lambda execution**, like:

- Request ID
    
- Remaining execution time
    
- Function name

## 7️⃣ Response (What Lambda Returns)

What Lambda returns depends on **who triggered it**.

For now:

- We return a simple string
    
- Later, APIs return JSON + status code


```python
import json   # import python JSON module

def lambda_handler(event, context):
    return {
        'statusCode': 200,
        'body': json.dumps('Hello from Lambda!')
    }

```



### import json

- Imports Python’s built-in `json` module
- Used to convert Python objects → JSON strings
    
Why this matters:
- HTTP APIs work with **JSON**
- API Gateway expects the response body as a **string**
## `def lambda_handler(event, context):`

This defines the **Lambda handler function**.

### `lambda_handler`

- Function name AWS Lambda calls
    
- Must match the **Handler setting**
    

### `event`

- Input data to Lambda
    
- Comes from:
    
    - API Gateway
        
    - Test event
        
    - S3
        
    - DynamoDB, etc.
        

Example event:

`{   "key": "value" }`

---

### `context`

- Metadata about execution
    
- Includes:
    
    - request ID
        
    - remaining execution time
        
    - function name
        

Most of the time:

- Beginners don’t use it
    
- Advanced logging & tracing uses it
    

---

## `return { ... }` (IMPORTANT)

You are returning a **dictionary**, not plain text.

This dictionary has a **special meaning** when Lambda is integrated with **API Gateway**.

---

## statusCode': 200`

### What this means

- HTTP status code
    
- `200` = success
    

Common values:

- `200` → OK
    
- `201` → Created
    
- `400` → Bad request
    
- `500` → Server error
    

API Gateway reads this and sends it to the client.

---

## 'body': json.dumps('Hello from Lambda!')`

### Why `json.dumps()`?

API Gateway expects:

- `body` to be a **string**
    
- NOT a Python object
    

So we convert:

`'Hello from Lambda!'  →  "Hello from Lambda!"`

If you skip `json.dumps()`:

- Lambda runs
    
- API Gateway may fail or return incorrect output
    

---

## What API Gateway Actually Sends to Client

The client (browser/Postman) receives:

`HTTP/1.1 200 OK Content-Type: application/json  "Hello from Lambda!"`

---

## Full Execution Flow (Very Important)

```pgsql
Client Request
   ↓
API Gateway
   ↓
Lambda runs lambda_handler()
   ↓
Lambda returns dictionary
   ↓
API Gateway converts it to HTTP response

```



 Why This Format Exists (Historical Reason)

AWS created this format for:

- Simple integration
    
- Consistent HTTP responses
    
- Language-agnostic behavior
    

This is called:  
👉 **Lambda Proxy Integration Response**

You don’t need to memorize the name yet — just understand the structure.

---

## Common Beginner Mistakes ❌

### ❌ Mistake 1

Returning plain string:

`return "Hello"`

Works in test, but fails with API Gateway.

---

### ❌ Mistake 2

Returning body without `json.dumps`

`'body': {'msg': 'Hello'}`

❌ Not valid — body must be string.

---


---

## 🧠 Mental Model (Remember This)

|Part|Who Uses It|
|---|---|
|`event`|Lambda input|
|`statusCode`|API Gateway|
|`body`|Client|
|`json.dumps()`|JSON serialization|



## Project 1

List S3 bucket using lambda function with AWS SDK boto3

```
import boto3

def lambda_handler(event, context):
    s3 = boto3.client('s3')
    
    response = s3.list_buckets()
    
    buckets_with_region = []

    for bucket in response['Buckets']:
        bucket_name = bucket['Name']
        
        location = s3.get_bucket_location(Bucket=bucket_name)
        region = location.get('LocationConstraint')

        if region is None:
            region = 'us-east-1'

        buckets_with_region.append({
            "bucket_name": bucket_name,
            "region": region
        })

    return {
        "statusCode": 200,
        "buckets": buckets_with_region
    }

```

Return from lambda:
``` Response from lambda
{

"statusCode": 200,

"buckets": [

{

"bucket_name": "aws-cloudwatch-metric-demo-0001",

"region": "eu-west-1"

},

{

"bucket_name": "config-bucket-048860134829",

"region": "eu-west-1"

},

{

"bucket_name": "manish-demo-test-year2025",

"region": "eu-west-1"

},

{

"bucket_name": "manish-learning-001",

"region": "eu-west-1"

},

{

"bucket_name": "my-ireland-bkt-2",

"region": "eu-west-1"

}

]

}
```