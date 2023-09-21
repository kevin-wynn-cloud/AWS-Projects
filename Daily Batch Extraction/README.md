# Daily Batch Extraction

In this lab, I tackled a pressing issue faced by a major supermarket franchise: the strain on their transactional database servers due to extensive sales data queries. To address this challenge, we explore the concept of data lakes, allowing the customer to run queries without impacting their database servers. Our ideal solution involves using AWS Glue to automate scheduled data extraction tasks, storing the data in a data lake, and leveraging Amazon Athena for in-depth analysis.

# Lab Architecture

![0  Picture 0](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/f074ce28-1251-460a-a600-61ec3dea80d9)

# Step 1: RDS Configuration

In this initial step, I navigated to Secrets Manager and retrieved the 'Secret value' from a created secret. Next, I accessed my RDS databases, collecting vital information about the database VPC and subnet. Finally, I enhanced security by adding an inbound rule to my RDS security group, permitting all TCP traffic from my Database Security Group (a self-referencing rule).

![1  Picture 1](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/9c41bdad-30b0-4965-9d29-21ff063abc6b)

# Step 2: AWS Glue Database and Connection

Continuing on, I proceeded to AWS Glue. Here, I created a 'batch-db' database and established a connection named 'batch-conn' as an Amazon RDS connection, utilizing the Amazon Aurora DB engine. I employed the 'sales-database-instance-writer' DB instance and credentials from Secret Manager. Afterward, I configured 'batch-conn' to use my lab's VPC, database subnet, and security group.

![2  Picture 2](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/9647134a-8143-4bf4-bff6-1cb6834287e5)

# Step 3: Table Creation with AWS Glue Crawler
In this step, I added a table in AWS Glue using a crawler, naming it 'batch-db-table.' I specified the JDBC datastream as the source, linking it to 'batch-conn.' My path selection, 'sales/%,' ensured all tables in the sales database were included. Additionally, I configured the GlueRole to enable the crawler to assume the necessary role and ran the new crawler.

![3  Picture 3](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/e761ee48-1e2f-43f9-933a-68e70dc1a923)

# Step 4: ETL Job Creation in AWS Glue
Verifying the table definitions in my Data Catalog, I navigated to ETL Jobs. Here, I created a new job, opting for a visual interface with a source and target. I selected AWS Glue Catalog as the source and an S3 bucket as the target. In the visual editor, I established the 'Customer Table' node, mirroring the 'batch-db' schema and 'sales_customer' table.

![4  Picture 4](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/35fce22c-c0e7-46cc-868e-8d45ee66a4ed)

# Step 5: Configuring the S3 Target
Continuing in the visual editor, I selected the S3 bucket node and chose the Parquet format for efficient column-based data storage. Snappy compression was selected for high-speed data access. My raw data bucket became the S3 target location, with the schema remaining intact, representing 'batch-db' as the database and 'customers' as the table.

![5  Picture 5](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/fe6198a8-1054-4f20-9316-1dd149277639)

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

![7  Picture 7](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/0a08cdb9-8a4b-4437-8d08-1756f4c2a5eb)

# Step 8: Athena's Analytics
To analyze the data, I navigated to Amazon Athena. I managed settings, specifying my 'processed-bucket' as the destination for query results. I executed an SQL query and reviewed the results.

```sql
SELECT * FROM "batch-db"."customers" limit 10;
```

![8  Picture 8](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/70c664f0-6832-4882-8c6b-b5746ededd52)

# Step 9: Final Endeavor
Concluding this lab walkthrough, I created another job in AWS Glue Studio. This job ingested transaction table data from the Data Catalog, created the transaction table, and facilitated data access via Athena.

![9  Picture 9](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/a532641c-9dea-425c-8057-57925f01a05c)

