# Populating the Data Catalog

The customer faces challenges in organizing their data lake. Leveraging the AWS Glue Data Catalog can provide a solution, even if you choose to keep your data in its current location. AWS Glue crawlers will populate the Data Catalog with metadata and table definitions. Once these tables are added to the Data Catalog, Amazon Athena can be employed to analyze S3 data using SQL. The provided lab architecture:

![0  Picture 0](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/043398a7-e3e2-4ee9-8920-d5d6099b1eca)

Here's the step-by-step process to complete this lab:

# Step 1: Preparing the Data

To begin, I navigated to my S3 buckets and confirmed the presence of the inventory.csv file. Next, I proceeded to my AWS Glue databases and created a new database called inventory-db. Afterward, I accessed the tables section and added a table using a crawler named inventory-crawler, specifying my raw data bucket as the data source. I utilized the existing aws_glue_role IAM role and linked it to my inventory-db database to finalize this step.

![1  Picture 1](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/a407b254-e6b2-4192-971d-7afb5078e08a)

# Step 2: Cataloging and Querying Data

Moving on, I executed the newly created crawler to ensure the creation of the raw_data table in my AWS Glue Catalog. I also reviewed the schemas of this table. Following this, I accessed the Amazon Athena query editor, adjusted the editor settings to store query results in my query-results bucket, and previewed the table schemas within the editor.

![2  Picture 2](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/2a199931-a094-41c8-85ab-6447fff7150a)

# Step 3: Data Generation with AWS Lambda

Next, I explored the AWS Lambda service and examined my labFunction-inventory-Management function. The Python script for this function was as follows:

``` python
from faker import Faker
import random
from faker.providers import currency, barcode

import logging
import boto3
from botocore.exceptions import ClientError
import os

import pandas as pd
from collections import defaultdict

# Need to update the environment variable in the Lambda function with your input S3 bucket 
inputBucket = os.environ['input_bucket']

def lambda_handler(event, context):
    generate_data()
    upload_file(file_name='/tmp/inventory.csv',bucket=inputBucket)

def generate_data():
    fake = Faker()
    fake.add_provider(currency)

# Use Faker to generate updated inventory data. Add two new columns (product city and product country)
    fake_data = defaultdict(list)
    for _ in range(1000):
        fake_data["Product_id"].append(random.randint(0, 10000))
        fake_data["Product_stock"].append(random.randint(1, 1000))
        fake_data["Product_barcode"].append(fake.ean(length=8))
        fake_data["Product_city"].append(fake.city())
        fake_data["Product_country"].append(fake.country())

    df_fake_data = pd.DataFrame(fake_data)

    print(df_fake_data.head())

# Remove the first column index data
    df_fake_data.to_csv("/tmp/inventory.csv",index=False)

def upload_file(file_name, bucket, object_name=None):
    """Upload a file to an S3 bucket

    :param file_name: File to upload
    :param bucket: Bucket to upload to
    :param object_name: S3 object name. If not specified then file_name is used
    :return: True if file was uploaded, else False
    """

    # If S3 object_name was not specified, use file_name
    if object_name is None:
        object_name = os.path.basename(file_name)

    # Upload the file
    s3_client = boto3.client('s3')
    try:
        response = s3_client.upload_file(file_name, bucket, object_name)
    except ClientError as e:
        logging.error(e)
        return False
    return True
```

# Step 4: Lambda Configuration and Testing

I updated the environmental variables for the labFunction-inventory-Management function, specifically pointing the input_bucket key to my raw-data bucket's value. Subsequently, I configured a test event for the function and conducted a test run. The test event was successful, and I observed that the Product_city and Product_country columns were added as expected.

![4  Picture 4](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/587e590a-20d0-48d6-86c3-5aa7c8e5ceb8)

# Step 5: Re-cataloging Data

Returning to my AWS Glue crawlers, I re-ran the inventory-crawler. The crawler detected changes in the existing data, updating my table definition accordingly. It automatically incorporated new tables and partitions, which in this case included the product_city and product_country columns in my table's schemas. I was able to review both versions from the console. This process also updated the table schema in Athena.

![5  Picture 5](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/7efd87b0-1bcf-44f4-983c-733990cc1abb)

# Step 6: Schema Modification

Lastly, I edited my table schema in AWS Glue, changing the data type of the "product id" field in the inventory table from "bigint" to string. I then validated the updated data type both in the Data Catalog and in Athena. This step successfully improved the organization of the customer's data lake.

![6  Picture 6](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/579ad5e9-5200-43da-bfca-b11cd81acc3d)

By following these steps, I accomplished the task of populating the Data Catalog and optimizing the data lake organization.
