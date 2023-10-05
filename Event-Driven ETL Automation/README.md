# Event-Driven ETL Automation

The objective of this lab is to design and implement an event-driven Extract, Transform, Load (ETL) automation solution for a customer looking to streamline their fuel purchase planning for their fleet of trucks. Currently, data is captured using an application that generates JSON files, which are stored in a central shared folder. The existing process involves manual steps, such as removing sensitive customer information and running SQL queries. This lab leverages AWS services to automate this process, providing a more efficient and scalable solution.

# Lab Architecture:

![picture0](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/6257f90d-6a5c-4781-8d7f-eba3893745d5)

# Step 1: AWS Glue Job for JSON to Parquet Conversion

In this step, we create an AWS Glue job to convert JSON data to Parquet format. The script includes the following key components:

```python
import sys
from awsglue.transforms import *
from awsglue.utils import getResolvedOptions
from pyspark.context import SparkContext
from awsglue.context import GlueContext
from awsglue.job import Job

args = getResolvedOptions(sys.argv, ["JOB_NAME"])
sc = SparkContext()
glueContext = GlueContext(sc)
spark = glueContext.spark_session
job = Job(glueContext)
job.init(args["JOB_NAME"], args)

# Script generated for node S3 bucket
S3bucket_node1 = glueContext.create_dynamic_frame.from_options(
    format_options={"multiline": False},
    connection_type="s3",
    format="json",
    connection_options={
        "paths": ["s3://landing-bucket-189964473925-us-east-1/"],
        "recurse": True,
    },
    transformation_ctx="S3bucket_node1",
)

# Script generated for node Apply Mapping
ApplyMapping_node = ApplyMapping.apply(
    frame=S3bucket_node1,
    mappings=[
        ("shipping_id", "int", "shipping_id", "int"),
        ("shipping_date", "string", "shipping_date", "string"),
        ("customer_name", "string", "customer_name", "string"),
        ("street_address", "string", "street_address", "string"),
        ("destination_city", "string", "destination_city", "string"),
        ("shipping_cost", "double", "shipping_cost", "double"),
        ("shipping_distance", "double", "shipping_distance", "double"),
        ("quantity", "double", "quantity", "double"),
        ("product_id", "int", "product_id", "int"),
    ],
    transformation_ctx="ApplyMapping_node",
)

# Script generated for node Amazon S3
AmazonS3_node = glueContext.write_dynamic_frame.from_options(
    frame=ApplyMapping_node,
    connection_type="s3",
    format="json",
    connection_options={
        "path": "s3://staging-bucket-189964473925-us-east-1/transformed_data/",
        "partitionKeys": [],
    },
    transformation_ctx="AmazonS3_node",
)

job.commit()
```
I did likewise for my data normalization job, adding my staging bucket information to the script:

```python
import sys
from awsglue.transforms import *
from awsglue.utils import getResolvedOptions
from pyspark.context import SparkContext
from awsglue.context import GlueContext
from awsglue.job import Job

args = getResolvedOptions(sys.argv, ["JOB_NAME"])
sc = SparkContext()
glueContext = GlueContext(sc)
spark = glueContext.spark_session
job = Job(glueContext)
job.init(args["JOB_NAME"], args)

# Script generated for node Amazon S3
AmazonS3_node = glueContext.create_dynamic_frame.from_catalog(
    database="shipping-db",
    table_name="transformed_data",
    transformation_ctx="AmazonS3_node",
)

# Script generated for node Drop Fields
DropFields_node = DropFields.apply(
    frame=AmazonS3_node,
    paths=[
        "customer_name",
    ],
    transformation_ctx="DropFields_node",
)

# Script generated for node Amazon S3
AmazonS3_node = glueContext.write_dynamic_frame.from_options(
    frame=DropFields_node,
    connection_type="s3",
    format="glueparquet",
    connection_options={
        "path": "s3://staging-bucket-189964473925-us-east-1/normalized",
        "partitionKeys": [],
    },
    format_options={"compression": "uncompressed"},
    transformation_ctx="AmazonS3_node",
)

job.commit()
```

# Step 2: Data Normalization Job

Similar to Step 1, we create another AWS Glue job for data normalization. The script is structured as follows:

```python
import sys
from awsglue.transforms import *
from awsglue.utils import getResolvedOptions
from pyspark.context import SparkContext
from awsglue.context import GlueContext
from awsglue.job import Job

args = getResolvedOptions(sys.argv, ["JOB_NAME"])
sc = SparkContext()
glueContext = GlueContext(sc)
spark = glueContext.spark_session
job = Job(glueContext)
job.init(args["JOB_NAME"], args)

# Script generated for node Amazon S3
AmazonS3_node = glueContext.create_dynamic_frame.from_catalog(
    database="shipping-db",
    table_name="transformed_data",
    transformation_ctx="AmazonS3_node",
)

# Script generated for node Drop Fields
DropFields_node = DropFields.apply(
    frame=AmazonS3_node,
    paths=[
        "customer_name",
    ],
    transformation_ctx="DropFields_node",
)

# Script generated for node Amazon S3
AmazonS3_node = glueContext.write_dynamic_frame.from_options(
    frame=DropFields_node,
    connection_type="s3",
    format="glueparquet",
    connection_options={
        "path": "s3://staging-bucket-189964473925-us-east-1/normalized",
        "partitionKeys": [],
    },
    format_options={"compression": "uncompressed"},
    transformation_ctx="AmazonS3_node",
)

job.commit()
```

# Step 3: AWS Step Functions State Machine

In this step, we orchestrate the ETL process using AWS Step Functions. The state machine includes the following components:

- AWS Glue job to convert JSON to Parquet.
- Crawlers to create a raw data catalog and check the status of the first crawler.
- Choice Flow to determine if the first crawler is running.
- Wait state to pause execution if the first crawler is running.
- AWS Glue job for data normalization.
- StartCrawler Glue job to create the processed data catalog.
- GetCrawler job to check the status of the second crawler.
- Repeated Choice and Step 5 for data querying using Amazon Athena.

![picture 3](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/cd80ca0e-ae44-403f-b4b2-9e2ca58f5545)

# Step 4: Review and Finalization

Review the entire workflow graph and the Amazon State Language definition code. Configure the workflow to use the "AWSStepFunctionsRole-Lab" role and rename the state machine to "data-workflow" before finalizing its creation.

![picture6](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/3d52b7d4-1aee-4ac2-89d0-f6320a25e0a5)

# Step 5: Lambda Function Configuration

Configure Lambda functions that trigger the ETL process. Ensure that the Lambda functions are correctly set up to start the Step Functions state machine.

start_step_function function:

```python
import json
import urllib.parse
import boto3
import os

s3 = boto3.client('s3')
stf = boto3.client('stepfunctions')

state_machine = os.environ['STATE_MACHINE_ARN']

def lambda_handler(event, context):
    
    #print("Received event: " + json.dumps(event, indent=2))

    # Get the object from the event and show its content type
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = urllib.parse.unquote_plus(event['Records'][0]['s3']['object']['key'], encoding='utf-8')
    try:
        s3_response = s3.get_object(Bucket=bucket, Key=key)
        stf_response = stf.start_execution(
                            stateMachineArn=state_machine
                            )
        print('Processing new raw data file {}/{}'.format(bucket, key))
        print('Started state machine {} '.format(stf_response['executionArn']))

    except Exception as e:
        print(e)
        print('Error getting object {} from bucket {}. Make sure they exist and your bucket is in the same region as this function.'.format(key, bucket))
        raise e
```

The shipping_schedule_function function:

```python
"""
This lambda function Generates fake shipping data and uploads to the bucket.
"""

import os
import boto3
from botocore.exceptions import ClientError
import logging
import urllib3
import json

http = urllib3.PoolManager()
SUCCESS = "SUCCESS"
FAILED = "FAILED"

import random
import datetime
from faker import Faker
from faker.providers import address, person

fake = Faker()
fake.add_provider(address)
fake.add_provider(person)

# It is a good practice to use proper logging.
# Here we are using the logging module of python.
# https://docs.python.org/3/library/logging.html

logger = logging.getLogger()
logger.setLevel(logging.INFO)

s3 = boto3.client('s3')
bucket_name = os.environ['BUCKET_NAME']


def lambda_handler(event, context):
    logger.info(f'event: {event}')
    generate_data(10)


def generate_data(num_records):
    logger.info("Generating data")
    # Call the get_secrets() function to get data from Secrets manager
    faker = Faker()

    # list of fake toll plaza name
    shipping_destination_list = ["serverless_island", "island_city", "machine_learning_island", "architect_city", "data_analytic_hanger", "main_city"]


    fake = Faker()

    for n in range(0, num_records):
        fake_data = {}
        date_obj = datetime.datetime.now() + datetime.timedelta(days=random.randint(1, 1))
        delivery_date = date_obj.strftime("%Y/%m/%d")
        shipping_price = faker.random_int(1000, 10000) / 1.0
        load_details = faker.random_int(1000, 10000)
        shipping_distance = faker.random_int(10, 100)
        shipping_destination = random.choice(shipping_destination_list)
        fake_data["shipping_id"] = fake.random_number(5)
        fake_data["shipping_date"] = delivery_date
        fake_data["customer_name"] = fake.name()
        fake_data["street_address"] = fake.street_address()
        fake_data["destination_city"] = shipping_destination
        fake_data["shipping_cost"] = shipping_price
        fake_data["shipping_distance"] = shipping_distance
        fake_data["quantity"] = load_details
        fake_data = json.dumps(fake_data)
        with open('/tmp/shipping_data.json', 'a') as f_object:
            f_object.write(f"{fake_data}\n")
    print("File has been created.")

    try:
        response = s3.upload_file(
            '/tmp/shipping_data.json',
            Bucket=bucket_name,
            Key=f'shipping_data.json-{datetime.datetime.now().strftime("%Y-%m-%d-%H-%M")}.json'
        )
        logger.info('File Uploaded Successfully')
    except ClientError as e:
        logging.error(e)
        logger.info('File Not Uploaded')
    else:
        logger.info('empty list')
```

# Step 6: SNS Topic Integration

Integrate an SNS (Simple Notification Service) topic into the Step Functions state machine workflow. Configure notifications to publish to the created topic when loading shipping data. Ensure that you specify the topic's ARN in the Step Functions configuration.

![picture10](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/c35efde1-728d-4a97-bfc8-a0c206fd9d77)

# Conclusion:

This lab demonstrates the design and implementation of an event-driven ETL automation solution using AWS services. By automating the data transformation process, the customer can efficiently plan their fuel purchases for their truck fleet, reducing manual effort and improving data processing speed. This architecture is scalable and can adapt to changing data requirements.
