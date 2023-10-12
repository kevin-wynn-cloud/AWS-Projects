# Real-Time Data Processing

In this lab, the customer manages several wind farms and would like to receive immediate alerts when wind speed becomes severe so that they can promptly notify their maintenance teams. We will utilize Amazon Kinesis Data Streams for Apache Flink to process real-time data. Kinesis Data Analytics will then analyze this data. We'll employ an AWS Lambda function to receive and store the data in an Amazon DynamoDB table, which is ideal for low-latency data storage. A second Lambda function will scan and filter for anomaly records, subsequently publishing notifications to an Amazon SNS topic.

# Lab Architecture

![1](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/f56b163a-6ef9-4dba-9943-87d23ecc176c)

# Step 1: Setting Up Kinesis Data Streams and Wind Turbine Simulator

To begin, I navigated to my kinesis-flink bucket and copied the AnomalyDetection.jar. Then, I accessed my instances and visited my Wind Turbine Simulator webpage using my instance's public IPv4 address. Afterward, I went to Amazon Kinesis and created a provisioned Kinesis data stream named WindDataStream. Once it was successfully created, I returned to the Wind Turbine Data Simulator webpage and entered the name of my Kinesis stream.

![2](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/467d8a06-e25f-44b8-b19f-adb16679d76f)

# Step 2: Configuring Data Streams and AnomalyDetectionStream

In the data viewer tab of the Kinesis console, I selected an available shard and chose the latest starting position to view the records. Subsequently, I created another provisioned data stream named AnomalyDetectionStream for ingesting the results of anomaly detections.

![3](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/f17b4396-b5f1-4e03-ba0a-0540ffca3147)

# Step 3: Creating and Configuring the AnomalyDetection Application

I proceeded to create my streaming application, selecting the "from scratch" option and naming it AnomalyDetection. I utilized my existing lab-kinesis-analytics-role service role and selected the development template. Next, I configured the streaming application, pointing it toward my kinesis-flink-application bucket to fetch the AnomalyDetection.jar file, and let Kinesis assume my lab-kinesis-analytics role. I configured the runtime properties as shown:

![4](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/319591e8-1169-43d1-a799-3eb2c2048872)

# Step 4: Running the AnomalyDetection Application and Configuring Lambda Function

I then executed my AnomalyDetection application with the latest snapshot. On the webpage, I initiated the wind speed anomaly dataset and verified that the simulator was producing wind data. Subsequently, I went to Lambda and selected my AnalyticsDestinationFunction, adding a trigger by selecting my Kinesis AnomalyDetectionStream stream as the trigger. The code accepts data from the analytics application in JSON format and parses the data to store it in a DynamoDB table. The function's code is as follows:

```python
Copy code
# Creating a Lambda package with runtime dependencies
# https://docs.aws.amazon.com/lambda/latest/dg/python-package-create.html#python-package-create-with-dependency
from datetime import datetime
import boto3
import os
import base64
import json

dynamodb = boto3.client('dynamodb')
output_table = os.environ.get('OUTPUT_TABLE_NAME')


def handler(event, context):
    # Response will be a list of records.
    response = {
        'records': []
    }
    
    for record in event['Records']:
        payload = json.loads(base64.b64decode(record.get('kinesis').get('data')).decode('utf-8'))
        try:
            dynamodb.put_item(TableName=output_table,
                    Item={
                        'timestamp': {
                            'S': str(datetime.now())
                        },
                        'location': {
                            'S': payload['location']
                        },
                        'wind_speed': {
                            'S': str(payload['speed'])
                        },
                        'anomaly_score': {
                            'S': str(payload['score'])
                        }
                    })
            response['records'].append({
                'recordId': record.get('recordId'),
                'result': 'Ok'
            })
        except Exception as e:
            print(e)
            response['records'].append({
                'recordId': record.get('recordId'),
                'result': 'DeliveryFailed'
            })

    return response
```

# Step 5: Monitoring DynamoDB and Setting Up SNS Notifications

I accessed my DynamoDB console, clicked on "Explore items," and selected the WindDataTable. It took several minutes for the table to propagate, so please be patient if it takes a while. This was the longest-waiting step of the lab. Afterward, I created an SNS topic and subscribed my email address. Then, I reviewed the code of the AnomalyMessageDeliveryFunction to ensure emails were sent to my account:

```python
Copy code
import json
import os
import boto3
from boto3.dynamodb.conditions import Key, Attr

dynamodb = boto3.resource('dynamodb')
sns = boto3.client('sns')

output_table = os.environ.get('OUTPUT_TABLE_NAME')
sns_topic_arn = os.environ.get('SNS_TOPIC_ARN')

def handler(event, context):
    table = dynamodb.Table(output_table)
    
    ddb_response = table.scan(FilterExpression=Attr('anomaly_score').gte('3'))
    
    items = ddb_response['Items']

    for item in items:
        location = item['location']
        wind_speed = item['max_wind_speed']
        
        message = '''\
                Abnormal wind turbine speed detected for {location} with a wind speed of {wind_speed}.
        
                NOTE: As per National Wind Watch, every wind turbine has a range of wind speeds, typically around 30 to 55 mph, in which it will produce maximum capacity. When wind is more than 70 mph, it is important to start shutdown procedures in order to protect the turbine from high wind.\
                '''.format(location=location, wind_speed=wind_speed)
       
        sns_response = sns.publish(
            TargetArn=sns_topic_arn,
            Message=(message),
            Subject='Abnormal Wind Turbine Speed Detected'
        )
        
        print(f"Abnormal wind turbine speed detected for {location} with a wind speed of {wind_speed}.")
```

# Step 6: Creating CalculateMaxSpeed Application and Configuring Triggers

Next, I created a new Kinesis Analytics for Apache Flink application called CalculateMaxSpeed. I configured it to point to my kinesis-flink bucket and used the MaxWindSpeed.jar. I created a new Kinesis Data Stream named MaxWindSpeed to ingest the output from the Apache Flink application. I added the new MaxWindSpeed stream as the trigger for the DIYFunction here:

```python
Copy code
from datetime import datetime
import boto3
import os
import base64
import json

dynamodb = boto3.client('dynamodb')
output_table = os.environ.get('OUTPUT_TABLE_NAME')


def handler(event, context):
    # Response will be a list of records.
    response = {
        "records": []
    }
    
    for record in event['Records']:

        payload = base64.b64decode(record.get('kinesis').get('data')).decode('utf-8')
        print(payload)
        payload_values = payload.split(",")
        
        try:
            dynamodb.put_item(TableName=output_table,
                    Item={
                        'timestamp': {
                            'S': str(datetime.now())
                        },
                        'location': {
                            'S': payload_values[0].replace('"', '')
                        },
                        'max_wind_speed': {
                            'S': payload_values[1]
                        }
                    })
            response['records'].append({
                'recordId': record.get('recordId'),
                'result': 'Ok'
            })
        except :
            response['records'].append({
                'recordId': record.get('recordId'),
                'result': 'DeliveryFailed'
            })

    return response
```
# Step 7: Verifying Data in DynamoDB

Finally, from the Wind Turbine Data Simulator, I started the DIY dataset. I checked to verify that the DIY DynamoDB table was populated with maximum wind speed data for the city wind farm.

![5](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/86662e31-f80a-41c7-93e0-dde87dd22e7f)

