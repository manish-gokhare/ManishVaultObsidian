
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