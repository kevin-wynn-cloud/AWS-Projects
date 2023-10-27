# Integrating Serverless Applications

The customer desires a more robust solution than receiving data in an email from their robotic ants. They want a solution that can collect and process ant data in real-time. By using Amazon SQS, we can decouple and scale microservices, distributed systems, and serverless applications. Additionally, we'll set up a dead-letter queue to store messages that are not successfully processed for review.

# Lab Architecture

![0](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/6b0aa35f-c26d-4517-bd95-059bcedd3ace)

# Step 1: Creating an SQS Message Queue

I navigated to SQS and created a message queue. Afterward, I created a sample message and polled the queue for messages.

![1](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/f8ae2246-8fbe-4500-a63f-6dce7e688083)

# Step 2: Configuring a Dead-Letter Queue

I created a dead-letter queue for messages that failed to process. When setting up the dead-letter queue, make sure that your message queue points toward the dead-letter queue and that the dead-letter queue points back to your message queue.

![2](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/c27db398-f51a-46d4-998c-7db4a9c4edec)

# Step 3: Configuring Lambda Producer Function

Next, I went to Lambda and edited the producer function to include my queue's URL in the code. Then, I deployed the changes and created a test event for my message queue.

```python
import boto3
import json
import time

# Boto3 - DynamoDB Client - Mode Info: https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/dynamodb.html
sqs = boto3.resource('sqs')

# Replace Queue name with Queue URL from Account
queue = sqs.Queue('https://sqs.us-east-1.amazonaws.com/189964473925/MyMessageQueue')

# AWS Lambda Function that publishes a message at Queue
def lambda_handler(event, context):

    message = {
        "coder_id": "123",
        "spot_id": "321",
        "timestamp": round(time.time() * 1000)
    }
    response = queue.send_message(MessageBody=json.dumps(message))

    return response
```

# Step 4: Confirming Test Events in the Message Queue

I navigated to my message queue and confirmed that the test event successfully made its way to my queue.

![4](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/f70b8f9a-6689-489c-9104-07dc6b078e84)

# Step 5: Configuring Lambda Consumer Function

Next, I configured a Lambda trigger for my message queue, specifying my consumer function as the trigger. The consumer function, responsible for processing messages from Amazon SQS, is as follows:

```python
import json
import boto3

# Boto3 - DynamoDB Client - Mode Info: https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/dynamodb.html
sqs = boto3.resource('sqs')
dynamodb = boto3.resource('dynamodb')

# AWS Lambda Function that consumes event from Queue and writes at DynamoDB
def lambda_handler(event, context):
    if 'Records' in event:
        for record in event['Records']:
            payload = json.loads(record["body"])
            if 'coder_id' not in payload:
                raise ValueError('erro format')
            else:
                table = dynamodb.Table('checkinData')
                table.put_item(
                   Item={
                        'coderId': payload['coder_id'],
                        'timestamp': payload['timestamp'],
                        'spotID': payload['spot_id']
                        }
                )

    return {
        'statusCode': 200,
        'body': json.dumps(payload)
    }
```

# Step 6: Reviewing DynamoDB Data
I then navigated to my DynamoDB checkinData table to review the items that were returned.

![6](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/b99c843d-be72-402e-8c6f-67f7f0b92cee)

# Step 7: Monitoring and Troubleshooting
Lastly, I reviewed the logs of my invoked Lambda function in CloudWatch Logs. It's also possible to log output statements for quicker troubleshooting.

![8](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/410be93c-7eaa-4319-85b5-dbae43a71ef3)
