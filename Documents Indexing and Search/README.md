# Documents Indexing and Search with AWS Glue and Amazon OpenSearch Service

In this lab, we will explore how to enhance an online banking application to meet the customer's requirements, which include extending the transaction history from the last six months to the last five years, enabling text-based searching across all fields in bank statements, and ensuring cost-effectiveness, speed, and scalability. To achieve this, we will leverage the capabilities of Amazon OpenSearch Service, which seamlessly integrates with Amazon S3 and can directly ingest records from the banking mainframe.

# Architecture

![picture0](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/1546c043-4dc3-42eb-b674-b7ade9a28115)

# Step 1: OpenSearch Domain Setup

- Navigated to Amazon OpenSearch Service and created a public access domain named "bank-transactions" in a single Availability Zone (AZ).
- Utilized version 7.1 of OpenSearch and provisioned an m5.large.search instance.
- Created a master user named "lab-user."

![picture1](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/90934daf-efbe-4a93-9d05-f8a4aa932f0f)

# Step 2: Glue ETL Job Configuration

- In Glue Studio, created an ETL job named "bank-transactions-ingestion-job."
- Developed a Spark script within the job and specified the path to the dependent JAR located in the S3 ingestion bucket.
- Added job parameters including endpoint, user, password, and input bucket details.
- Executed the ETL job.

```python
import sys
from awsglue.transforms import *
from awsglue.utils import getResolvedOptions
from pyspark.context import SparkContext
from awsglue.context import GlueContext
from awsglue.job import Job

# The Jobs parameters are received here
args = getResolvedOptions(sys.argv, ["JOB_NAME","es_user","es_pass","es_endpoint","input_bucket"])

sc = SparkContext()
glueContext = GlueContext(sc)
spark = glueContext.spark_session
job = Job(glueContext)
job.init(args["JOB_NAME"], args)


# This section of the code create a DynamicFrame from 
# the the CSV files found in the input bucket.
# For more information on DynamicFrame see:
# https://docs.aws.amazon.com/glue/latest/dg/glue-etl-scala-apis-glue-dynamicframe-class.html 

TransactionsDF = glueContext.create_dynamic_frame.from_options(
    format_options={
        "quoteChar": '"',
        "withHeader": True,
        "separator": ",",
        "optimizePerformance": False,
    },
    connection_type="s3",
    compression="gzip", 
    format="csv",
    connection_options={
        "paths": [args['input_bucket']],
        "recurse": True,
    },
    transformation_ctx="TransactionsDF",
)

#  This sections creates the mappings for the transformation of the source DataFrame,
#  list of mapping tuples, each consisting of: (source column, source type, target column, target type)
#  with the mappings you can change the column name and its type from source to destination
#  To remove a unnecessary column that you don't want to index, remove one of the tuples (ex: line 53)

ApplyMapping  = ApplyMapping.apply(
    frame=TransactionsDF,
    mappings=[
        ("sequence_number", "long", "sequence_number", "long"),
        ("account_id", "long", "account_id", "long"),
        ("date", "string", "date", "string"),
        ("year", "long", "year", "long"),
        ("type", "string", "type", "string"),
        ("operation", "string", "operation", "string"),
        ("amount", "choice", "amount", "choice"),
        ("balance", "choice", "balance", "choice"),
        ("k_symbol", "string", "k_symbol", "string"),
    ],
    transformation_ctx="ApplyMapping",
)

DataFrame = ApplyMapping.toDF()

#  The following section write the records from the dataframe into the opensearch 
#  clustewr on the especified es_endpoint bellow 
#  using the elasticsearch-hadoop connector. 
# 
#  You can specify any name for your index, or create multiple indexex with different 
#  data. The driver will auto create the index (see line 75). 
#  The index format is "name/type". For more information about ElasticSearch indexes and type see:
#  https://www.elastic.co/blog/index-vs-type
#  For more information on the usage of the elasticsearch-hadoop connector see:
#  https://www.elastic.co/guide/en/elasticsearch/hadoop/current/configuration.html 
#  

es_index = "main-index/transactions"

DataFrame.write.mode("overwrite").format("org.elasticsearch.spark.sql").\
        option("es.resource", "index/type").\
        option("es.net.http.auth.user",args['es_user']).\
        option("es.net.http.auth.pass",args['es_pass']).\
        option("es.nodes", args['es_endpoint']).\
        option("es.port", 443).\
        option("es.nodes.wan.only", True).\
        option("es.index.auto.create", True).\
        option("es.resource", es_index).\
        option("es.mapping.id", "sequence_number").\
        option("es.write.operation", "upsert").\
        save()

print("Moved records: ", DataFrame.count())
print("Total records: ", DataFrame.count())

job.commit()
```

![picture 2](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/a1d53b02-5be5-4880-867e-215c8ea55e18)

![picture2](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/62fc7df2-9c08-4afd-a404-074c1016ad6a)

# Step 3: Application Testing and Kibana Exploration

- Accessed the online banking application via the EC2 instance's public IP address.
- Tested the application to ensure it allows searching across the last five years of transactions and performs text-based searches.
- Accessed the Kibana URL associated with the OpenSearch domain.
- Entered the credentials and interacted with Elasticsearch API as a private tenant.

![picture3](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/cbf6f61e-60ec-4124-aa85-b41c9b381e49)

![picture4](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/3b3c6364-3616-4fa4-bcb8-c202df7adea3)

