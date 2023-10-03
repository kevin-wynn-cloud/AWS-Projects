# Event-Driven Serverless ETL

In this lab, we will implement an Event-Driven Serverless ETL (Extract, Transform, Load) process using AWS services to provide real-time toll plaza data to a customer. The objective is to efficiently collect data from an Amazon S3 bucket, process it using AWS Glue, and store it in an Amazon Redshift database for further analysis.

# Architecture

![1](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/9a77b0f0-3c20-40df-b447-c6621f8fcc0f)

# Step 1: Configuring Amazon S3 Event Notifications
To kickstart the ETL process, we first set up an Amazon S3 event notification. We created an event called s3Events with a .json suffix and set it to trigger when new objects are created in the S3 landing bucket. These events are routed to invoke the start_workflow_function Lambda function, which initiates our ETL pipeline.

![Picture1](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/90bbe831-10a8-4b44-a11b-bf9f64171957)

# Step 2: Creating Amazon Redshift Table
In Amazon Redshift, we connected to the toll_db database as an admin user using the query editor. We ran SQL to create a toll_table with the necessary columns to store the toll plaza data.

```sql
Copy code
CREATE TABLE toll_table (
  transaction_id bigint,
  transaction_date character varying NOT null,
  toll_booth character varying NOT null,
  vehicle_make character varying NOT null,
  vehicle_category character varying NOT null,
  transaction_amount REAL
);
```

![picture2](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/76e243ab-9776-40da-8a94-fd9e9911a78a)

# Step 3: Setting Up AWS Secrets Manager and AWS Glue Connection
We utilized AWS Secrets Manager to securely retrieve our secret values, which are required for connecting to the Amazon Redshift database. We then configured an AWS Glue data connection by specifying the JDBC URL, LabVPC settings, database subnet, and Redshift security group.

![picture3](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/e613e4ff-76a3-4964-b6f7-0e2c4a89b886)

# Step 4: Running AWS Glue Crawler
With the data connection in place, we created an AWS Glue crawler, selecting JDBC as the data source and the redshift_conn connection. This crawler was associated with our AWS Glue IAM role, and we executed it to discover the schema and metadata of our data in Amazon Redshift.

![picture4](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/ea31d89e-22a5-4426-994b-10a4a63c041b)

# Step 5: Creating ETL Job
We proceeded to create an ETL job within AWS Glue. This ETL job extracts data from the S3 landing bucket, transforms it as needed, and loads it into Amazon Redshift. In the visual editor, we specified the source (S3 and Data Catalog) and target (Redshift and staging bucket) configurations.

![picture5](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/b807dbc7-7992-4142-8ae9-bff99fb39543)

# Step 6: Defining ETL Workflow and Trigger
To orchestrate our ETL process, we established an ETL workflow named redshift_workflow. Within this workflow, we introduced an On-Demand trigger named redshift-workflows-start. We added our S3 crawler as one of the job nodes, creating a seamless data flow.

![picture6](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/6122474a-9f50-4fec-b457-4fce3a43480c)

# Step 7: Testing and Invoking the Lambda Function
Finally, we tested and invoked our Lambda function, which generates fake toll plaza data and uploads it to the specified S3 bucket. This function is designed to simulate data ingestion and initiation of the ETL process. Proper logging and error handling were implemented to ensure smooth execution.

```python
import json
import sys
import os
import boto3
from botocore.exceptions import ClientError
import logging
import urllib3
http = urllib3.PoolManager()
SUCCESS = "SUCCESS"
FAILED = "FAILED"

import random
import datetime
from faker import Faker
from faker.providers import bank, credit_card, date_time, profile,currency, user_agent, job
from faker_vehicle import VehicleProvider

# It is a good practice to use proper logging.
# Here we are using the logging module of python.
# https://docs.python.org/3/library/logging.html

logger = logging.getLogger()
logger.setLevel(logging.INFO)

s3 = boto3.client('s3')
bucket_name = os.environ['BUCKET_NAME']
filename_list = []
responseData = {'status': 'NONE'}


def lambda_handler(event, context):
    logger.info(f'event: {event}')
    try:
        if event['RequestType'] == 'Create' or event['RequestType'] == 'Update':
            try:
                generate_data(100)
                logger.info(f"Upload Complete")
                responseData['status'] = 'Upload Complete'
                send(event, context, SUCCESS, responseData ,physicalResourceId=event['LogicalResourceId'])
            except Exception as e:
                logger.info(f"Error: {e}")
                responseData['status'] = f'FAILED TO COPY. ERROR {e}'
                send(event, context, SUCCESS, responseData ,physicalResourceId=event['LogicalResourceId'])
        elif event['RequestType'] == 'Delete':
            responseData['status'] = f'DELETE IN PROGRESS'
            send(event, context, SUCCESS, responseData ,physicalResourceId=event['LogicalResourceId'])
    except Exception as e:
        logger.info(f"Error: {e}")
        try:
            generate_data(100)
            logger.info(f"Upload Complete")
        except Exception as e:
            logger.info(f"Error: {e}")
    


def generate_data(num_records):
    logger.info("Generating data")
    # Call the get_secrets() function to get data from Secrets manager
    faker = Faker()

    # list of fake toll plaza name
    toll_plaza_list = ["buckley_rd","82nd_st","royal_ave","savanna_west","golf_rd","york_rd"]
    # toll_booth_name = random.choice(toll_plaza_list)
    

    fake = Faker()
    fake.add_provider(bank)
    fake.add_provider(credit_card)
    fake.add_provider(profile)
    fake.add_provider(date_time)
    fake.add_provider(currency)
    fake.add_provider(user_agent)
    fake.add_provider(job)
    fake.add_provider(VehicleProvider)


    fake_data = {}
    for n in range(0,num_records):
        date_obj = datetime.datetime.now() - datetime.timedelta(days=random.randint(0, 30))
        transaction_date = date_obj.strftime("%Y/%m/%d")
        toll_price = faker.random_int(1, 1000) / 100.0
        toll_plaza_name = random.choice(toll_plaza_list)
        fake_data["transaction_id"] = fake.random_number(5)
        fake_data["transaction_date"] = transaction_date
        fake_data["toll_booth"] = toll_plaza_name
        fake_data["vehicle_make"] = fake.vehicle_make()
        fake_data["vehicle_category"] = fake.vehicle_category()
        fake_data["transaction_amount"] = toll_price

        with open('/tmp/sample_data.json', 'a') as f_object:
            f_object.write(f"{fake_data}\n")
 
    print("File has been created.")

    try:
        response = s3.upload_file(
            '/tmp/sample_data.json',
            Bucket=bucket_name,
            Key=f'sample_data-{datetime.datetime.now().strftime("%Y-%m-%d-%H-%M")}.json'
            )
        logger.info('File Uploaded Successfully')
    except ClientError as e:
        logging.error(e)
        logger.info('File Not Uploaded')
    else:
        logger.info('empty list')

def send(event, context, responseStatus, responseData, physicalResourceId=None, noEcho=False, error=None):
    responseUrl = event['ResponseURL']

    logger.info(responseUrl)

    responseBody = {}
    responseBody['Status'] = responseStatus
    if error is None: 
        responseBody['Reason'] = 'See the details in CloudWatch Log Stream: ' + context.log_stream_name + ' LogGroup: ' + context.log_group_name
    else:
        responseBody['Reason'] = error
    responseBody['PhysicalResourceId'] = physicalResourceId or context.log_stream_name
    responseBody['StackId'] = event['StackId']
    responseBody['RequestId'] = event['RequestId']
    responseBody['LogicalResourceId'] = event['LogicalResourceId']
    responseBody['NoEcho'] = noEcho
    responseBody['Data'] = responseData

    json_responseBody = json.dumps(responseBody)

    print("Response body:\n" + json_responseBody)

    headers = {
        'content-type' : '',
        'content-length' : str(len(json_responseBody))
    }
    try:
        response = http.request('PUT',responseUrl,body=json_responseBody.encode('utf-8'),headers=headers)
        print("Status code: " + response.reason)
    except Exception as e:
        print("send(..) failed executing requests.put(..): " + str(e))

```

![picture7](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/d995f8af-e5f9-46df-8ef4-f821e5a26cc1)

# Conclusion
This Event-Driven Serverless ETL architecture leverages AWS services, such as S3, Glue, and Redshift, to provide real-time toll plaza data to our customer. By automating data collection, transformation, and loading, we ensure that the most up-to-date toll data is available for analysis and reporting. The architecture can be further extended and optimized based on specific business requirements and data volume.
