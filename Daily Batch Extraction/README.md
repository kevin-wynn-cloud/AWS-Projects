# Daily Batch Extraction

In this lab, I tackled a pressing issue faced by a major supermarket franchise: the strain on their transactional database servers due to extensive sales data queries. To address this challenge, we explore the concept of data lakes, allowing the customer to run queries without impacting their database servers. Our ideal solution involves using AWS Glue to automate scheduled data extraction tasks, storing the data in a data lake, and leveraging Amazon Athena for in-depth analysis.

# Lab Architecture

![1](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/c37d1e79-928a-4f3d-9981-617ec1398729)

# Step 1: RDS Configuration

In this initial step, I navigated to Secrets Manager and retrieved the 'Secret value' from a created secret. Next, I accessed my RDS databases, collecting vital information about the database VPC and subnet. Finally, I enhanced security by adding an inbound rule to my RDS security group, permitting all TCP traffic from my Database Security Group (a self-referencing rule).

![2](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/e2cd643b-623c-4fbb-8220-ae84df323dc0)

# Step 2: AWS Glue Database and Connection

Continuing on, I proceeded to AWS Glue. Here, I created a 'batch-db' database and established a connection named 'batch-conn' as an Amazon RDS connection, utilizing the Amazon Aurora DB engine. I employed the 'sales-database-instance-writer' DB instance and credentials from Secret Manager. Afterward, I configured 'batch-conn' to use my lab's VPC, database subnet, and security group.

![3](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/a1f5509c-0787-456c-a770-dc73a38f1e7c)

# Step 3: Table Creation with AWS Glue Crawler
In this step, I added a table in AWS Glue using a crawler, naming it 'batch-db-table.' I specified the JDBC datastream as the source, linking it to 'batch-conn.' My path selection, 'sales/%,' ensured all tables in the sales database were included. Additionally, I configured the GlueRole to enable the crawler to assume the necessary role and ran the new crawler.

![4](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/767fd3bf-898e-4f07-9339-64d5a5b4c4ca)

# Step 4: ETL Job Creation in AWS Glue
Verifying the table definitions in my Data Catalog, I navigated to ETL Jobs. Here, I created a new job, opting for a visual interface with a source and target. I selected AWS Glue Catalog as the source and an S3 bucket as the target. In the visual editor, I established the 'Customer Table' node, mirroring the 'batch-db' schema and 'sales_customer' table.

![5](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/6b0c41d5-f8e8-4296-93e6-b0e057a4bb06)

# Step 5: Configuring the S3 Target
Continuing in the visual editor, I selected the S3 bucket node and chose the Parquet format for efficient column-based data storage. Snappy compression was selected for high-speed data access. My raw data bucket became the S3 target location, with the schema remaining intact, representing 'batch-db' as the database and 'customers' as the table.

![6](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/de201abd-9979-48fa-9acf-79068eeb50d8)

# Step 6: Unveiling the Script Editor
A review of the script editor revealed generated code, offering customization options for ETL jobs if needed. This code streamlined the data transformation process.

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

# Script generated for node Customer Table
CustomerTable_node1 = glueContext.create_dynamic_frame.from_catalog(
    database="batch-db",
    table_name="sales_customers",
    transformation_ctx="CustomerTable_node1",
)

# Script generated for node S3 bucket
S3bucket_node2 = glueContext.getSink(
    path="s3://raw-bucket-189964473925-707",
    connection_type="s3",
    updateBehavior="LOG",
    partitionKeys=[],
    compression="snappy",
    enableUpdateCatalog=True,
    transformation_ctx="S3bucket_node2",
)
S3bucket_node2.setCatalogInfo(catalogDatabase="batch-db", catalogTableName="customers")
S3bucket_node2.setFormat("glueparquet")
S3bucket_node2.writeFrame(CustomerTable_node1)
job.commit()
```

# Step 7: Job Naming and Configuration
I named the job 'customers_job' and specified my GlueRole IAM role. My job configuration included three workers, one retry, and a 5-minute job timeout. Saving the configuration, I ran the job and ensured its successful completion.

![7](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/25378a42-b79e-4dac-a1ce-68e6cb4732c5)

# Step 8: Athena's Analytics
To analyze the data, I navigated to Amazon Athena. I managed settings, specifying my 'processed-bucket' as the destination for query results. I executed an SQL query and reviewed the results.

```sql
SELECT * FROM "batch-db"."customers" limit 10;
```

![8](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/a192db71-c0fb-409d-a00d-352e004ea999)

# Step 9: Final Endeavor
Concluding this lab walkthrough, I created another job in AWS Glue Studio. This job ingested transaction table data from the Data Catalog, created the transaction table, and facilitated data access via Athena.

![9](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/fad13044-0ae5-4aa1-9bfa-3933b9e09a82)

