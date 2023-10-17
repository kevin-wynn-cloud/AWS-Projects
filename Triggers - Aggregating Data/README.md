# Triggers - Aggregating Data

The customer's requirement is to develop a voting app that allows citizens to vote on city improvements. The primary goal is to create a dashboard that displays the most recently cast votes and the number of votes per region within the city. Additionally, it's important to count votes as soon as they are recorded in the vote table and update the dashboard in real-time while minimizing resource usage. To address these requirements, we have implemented an AWS solution that leverages Amazon DynamoDB Streams and AWS Lambda to capture and aggregate votes efficiently.

# Architecture

![0](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/34b9e7fc-3cc3-4950-9c9e-0b06e79b4e79)

# Step 1: Creating the DynamoDB Table

In this first step, I navigated to the AWS Management Console and created a DynamoDB table named "votes" with a partition key named "id" (data type: string).

# Step 2: Enabling DynamoDB Streams

Next, I went to the "Exports and Streams" tab of the "votes" table and selected the "Turn on" option for DynamoDB streams. I opted to capture the "New Image" option.

![1](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/d0606c52-3bea-4969-b851-03eaba9396f6)

# Step 3: Creating the Aggregation Table

To store aggregated data, I created a new DynamoDB table named "total_votes" with "region" as the partition key and "improvement" as the sort key. This table also includes an attribute called "total_votes" to track the number of votes.

# Step 4: Configuring the AWS Lambda Function

In the AWS Lambda console, I created a new Lambda function and selected the "process updated made to a DDB table" blueprint. I named the function "dynamodbstream-newvotes" and assigned the appropriate IAM role for DynamoDB stream access. I provided the Amazon Resource Name (ARN) of the "votes" DynamoDB table in the Lambda function's configuration.

Here's the Python code for the Lambda function:

```python
# Runtime: Python 3.7
import boto3

# This will instantiate DynamoDB client for later use.
dynamodb = boto3.resource('dynamodb')

def check_record_pattern(record):
    if 'dynamodb' in record and 'NewImage' in record['dynamodb']:
        new_image = record['dynamodb']['NewImage']
        return 'improvement' in new_image and 'region' in new_image and 'S' in new_image['improvement'] and 'S' in new_image['region']
    return False

def lambda_handler(event, context):

    print("Received Stream Event: " + str(event))
    for record in event['Records']:
        print(record['eventID'])
        print(record['eventName'])

        if check_record_pattern(record):
            improvement = record['dynamodb']['NewImage']['improvement']['S']
            region = record['dynamodb']['NewImage']['region']['S']
            print("Vote for " + improvement + " from region " +region)

            table = dynamodb.Table('total_votes')
            table.update_item(
                Key={
                    'improvement': improvement,
                    'region': region
                },
                UpdateExpression="ADD total_votes :votevalue",
                ExpressionAttributeValues={
                    ':votevalue': 1
                }
            )
        
    return 'Successfully processed {} records.'.format(len(event['Records']))
```

# Step 5: Monitoring and Deployment

In the Lambda console, I reviewed the trigger configuration, and I deployed the Lambda function with the provided code.

# Step 6: Testing with Sample Data

Within DynamoDB, I explored table items and created an item to invoke my Lambda function with sample data.

```json
Copy code
{
  "id": "234",
  "improvement": "MORE_TREES",
  "region": "SOUTH"
}
```

# Step 7: Monitoring and Logging

To monitor the function's execution, I accessed the CloudWatch logs via the Lambda console, selecting the most recent log stream.

![3](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/55b3c360-5767-40b6-90cc-e5de0bada315)

# Step 8: Deploying the Updated Lambda Code

I removed the comment markers in the Lambda code, redeployed the code, and created another item, "MORE_FLOWERS," to trigger the Lambda function.

```json
Copy code
{
  "region": {
    "S": "test_region"
  },
  "improvement": {
    "S": "test_improvement"
  },
  "total_votes": {
    "N": "1"
  }
}
```
