# Streaming Ingestion

In this Cloud Quest lab, our customer is a logistics manager for a gas station franchise who needs assistance in optimizing fuel deliveries. Currently, they manually query data every morning and then request gas trucks to be dispatched to their stations when fuel levels are low. By implementing Amazon Kinesis, their data can be collected, processed, and analyzed in real-time. Additionally, Amazon Athena can automate their querying from once per day to once per minute, enabling faster response times. Athena can also use an AWS Lambda function to queue refueling requests in an Amazon SQS delivery queue.

# Lab Architecture

![0  Picture 0](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/1e50e891-dc4c-4d27-b2a9-2da26d47bda9)

# Step 1: Setting Up the Kinesis Data Firehose Delivery Stream

First, I navigated to Amazon Kinesis and created a Kinesis Data Firehose delivery stream. I used Direct PUT as a source and Amazon S3 as a destination, enabling record format conversion and selecting Apache Parquet as an output format (to convert stream data from JSON to Apache Parquet before storing data in my bucket, which enables faster queries). I selected my AWS Glue DB named 'conversion' and the AWS Glue table named 'conversion_table'. I selected my 'consumption-bucket' as my S3 bucket destination.

![1  Picture 1](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/6eca1bfb-5e1a-4ea3-9f0d-2d7eeb5127b5)

# Step 2: Configuring Kinesis Data Firehose

Next, I enabled dynamic partitioning, new line delimiter, and inline parsing for JSON before creating my stream. Then I named keys for 'station_id', 'year', 'month', 'day', 'hour', and 'minute', with corresponding JQ expressions of .station_id and '.event_timestamp| strftime("%Y")' for all other keys. I applied the dynamic partitioning keys for the S3 bucket prefix, resulting in:

# Step 3: Using Existing IAM Role

Next, I used my existing IAM Kinesis Firehose role before finally creating my delivery stream.

![3  Picture 3](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/47be61ea-6a69-4425-892a-97971cc20d38)

# Step 4: Configuring AWS Lambda (GasStationApp)

After that, I went to AWS Lambda and selected my 'GasStationApp'. I configured the environmental variable to point my 'delivery_stream' key toward my 'SI-Firehose' value. Thereafter, I tested the Lambda code, which is included below:

```python
import os
import json
import random
import boto3
import time

firehose_client = boto3.client('firehose')

timestamp = time.time()


def lambda_handler(event, context):
    
    DELIVERY_STREAM = os.environ['delivery_stream']
    
    station_list = []
    
    for _ in range(10):
        
        id = _ + 1
        # guarantee a low fuel tank for one station
        if _ == 0:
            json_data = {
            'station_id': str(id),
            'fuel_tank1_level': str(random.randint(100,999)),
            'fuel_tank2_level': str(random.randint(100,999)),
            'fuel_tank3_level': str(random.randint(100,999)),
            'fuel_tank4_level': str(random.randint(100,999)),
            'fuel_tank5_level': str(random.randint(100,199)),
            'event_timestamp': int(timestamp)
            }
        else:
            json_data = {
            'station_id': str(id),
            'fuel_tank1_level': str(random.randint(100,999)),
            'fuel_tank2_level': str(random.randint(100,999)),
            'fuel_tank3_level': str(random.randint(100,999)),
            'fuel_tank4_level': str(random.randint(100,999)),
            'fuel_tank5_level': str(random.randint(100,999)),
            'event_timestamp': int(timestamp)
            }
        
        response = firehose_client.put_record(
            DeliveryStreamName=DELIVERY_STREAM,
            Record={
                    'Data': json.dumps(json_data)
            }
        )
        
        station_list.append(json_data)
        
    return station_list
```

![4  Picture 4](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/9cee869e-8195-4005-be3b-e41b0b3f8d90)

# Step 5: Verifying Data in S3 and Testing FuelPlanningApp

After, I verified that the data was placed successfully in my 'consumption-bucket' in S3. Then I returned to Lambda to test the 'FuelPlanningApp'. The lambda code for this app:

```python
import os
import time
import boto3

# Athena constant
DATABASE = os.environ['glue_database']
TABLE = os.environ['glue_table']

# S3 constant
S3_OUTPUT = os.environ['output_bucket']

# SQS constant
QUEUE_URL = os.environ['queue_url']

def lambda_handler(event, context):

    # Created query
    query = """SELECT * 
               FROM %s.%s 
               WHERE fuel_tank1_level < '200'
               OR fuel_tank2_level < '200'
               OR fuel_tank3_level < '200'
               OR fuel_tank4_level < '200'
               OR fuel_tank5_level < '200';""" % (DATABASE, TABLE)

    # Athena client
    client = boto3.client('athena')

    
    # Execution
    queryStart = client.start_query_execution(
        QueryString=query,
        QueryExecutionContext={
            'Database': DATABASE
        },
        ResultConfiguration={
            'OutputLocation': S3_OUTPUT,
        }
    )

    queryId = queryStart['QueryExecutionId']
    time.sleep(3)
    
    stationids = []
    queryResults = client.get_query_results(QueryExecutionId = queryId)

    rowheaders = queryResults['ResultSet']['Rows'][0]['Data'] #The first item in ['ResultSet']['Rows'] contains a list of the column names
    rowindex = 0
    for r in queryResults['ResultSet']['Rows']:
        row_dict = {}
        if rowindex > 0: #Skip column names
            columnindex = 0
            for columnvalue in r['Data']:
                row_dict[rowheaders[columnindex]['VarCharValue']] = columnvalue['VarCharValue']
                columnindex += 1
            stationids.append("Gas station " + row_dict['station_id'] + " is low on fuel. Alerting fuel trucks.")
            stationID = row_dict['station_id']
                
            send_sqs_message(stationID)
                
        rowindex += 1

    return stationids
    
def send_sqs_message(stationID):
    
    # SQS client
    client = boto3.client('sqs')
    
    sendMessage = client.send_message(
        QueueUrl=QUEUE_URL,
        MessageBody=stationID
    )
    
    print(sendMessage['MessageId'])
```

![5  Picture 5](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/96576fe3-11b5-40d7-8175-6642bd994fd7)

# Step 6: Reviewing SQS Queue and Testing FuelTruckApp

Afterward, I navigated to Amazon SQS and reviewed my 'Fuel_Planning_Queue'. Then I returned back to Lambda to test my 'FuelTruckApp' that polls the 'Fuel_Planning_Queue' in Amazon SQS for messages that contain the IDs of gas stations with low fuel. The Lambda code:

```python
import os
import boto3

def lambda_handler(event, context):
    
    QUEUE_URL = os.environ['queue_url']
    
    # sqs client
    client = boto3.client('sqs')
    
    receiveMessage = client.receive_message(
        QueueUrl=QUEUE_URL,
        MaxNumberOfMessages=10,
        WaitTimeSeconds=5
    )
    
    for m in receiveMessage.get('Messages', []):

        print("Fuel truck has been dispatched to gas station " + m['Body'] + ".")
        
        receipt_handle = m['ReceiptHandle']
        client.delete_message(
            QueueUrl=QUEUE_URL,
            ReceiptHandle=receipt_handle
        )
    
    processed_messages = len(receiveMessage.get('Messages', []))
    
    if processed_messages == 0:
        message = 'No messages found in queue. Messages processed: ' + str(processed_messages)
    else:
        message = 'Messages processed: ' + str(processed_messages)
        
    return message
```

![6  Picture 6](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/80043ff3-cea0-455b-af69-4c00cfe5a2e9)

# Step 7: Reducing SQS Queue Messages

By repeating the invocation of the 'FuelTruckApp', I was able to reduce the SQS queue messages to zero.

![7  Picture 7](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/1a405d78-d9f7-4cfb-b06c-ab25f4b09c9f)

![8  Picture 8](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/0c9218e6-531f-45da-bf4e-05b558159b9b)





