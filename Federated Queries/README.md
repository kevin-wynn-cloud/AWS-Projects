# Federated Queries

This project involved acquiring business insights from data within DynamoDB, particularly data derived from a customer request ticketing system, and performing queries on this dataset.

# Solutions

The first expored solution is to use an AWS Glue function (a crawler) to scan and create an AWS Glue Data Catalog. Then, utilize an Extract, Transform, and Load (ETL) job within AWS Glue. This ETL job takes the cataloged data and transforms it into a suitable format (in this case, Parquet) while creating a data lake in Amazon S3. Once the cataloged data resides in S3, SQL queries can be executed using Amazon Athena.

However, an alternative solution to explore involves using federated queries with Athena. This approach entails executing SQL queries against the data in DynamoDB through an AWS Lambda function, acting as an intermediary. In this lab, I explore both approaches.

# Provided Architecture

![1  Provided Architecture](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/bef49bf2-4661-4e55-b3af-44ef0b66ea14)

# Step 1

I navigated to the AWS Glue console and accessed the Crawler subsection. There, I initiated the process of creating a crawler named 'ticket-data-crawler' and configured the data source to my DynamoDB table, named 'DynamoDBTicketTable'.

![2  Crawler Source](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/adc65c91-12bc-40bb-93e2-c8ad0d142ffd)

# Step 2

I utilized my existing IAM role, 'AWSGlueServiceRole,' and targeted the database named 'glue_ticket_db.' Subsequently, I completed the creation of my crawler. I ran the crawler and patiently awaited confirmation of its completion. Finally, I verified that the crawler successfully added my 'DynamoDBTicketTable.'

![3  TicketTable](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/60b618b5-d4a3-49f0-a6d7-885624d90d33)

# Step 3

Next, I proceeded to the ETL job and selected the AWS Glue Catalog as my data source. I also designated an S3 bucket as my target destination. Within the visual editor, I selected the Data Catalog node, set the database to 'glue_ticket_db,' and the table to 'DynamoDBTicketTable.' Moving to the job editor, I chose the S3 bucket, selected Parquet as my desired format, and specified the location as my intended bucket ('data-lake-bucket-714823731171-us-east-1').

![4  Visual Editor](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/15a332e1-b68d-4b1f-b887-3647b56d702c)

# Step 4

In the Data Catalog update options, I selected to create a table in the Data Catalog and, on subsequent runs, update the schema and add new partitions. I chose 'glue_ticket_db' for the Database and named the table 'glue-etl-ticket-table.'

![5  Picture 5](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/a7f0b602-0481-49b1-a024-84d612994cba)

#Step 5

I then navigated to job details and named the job 'ticket-etl-job.' For the IAM Role, I utilized 'AWSGlueServiceRole-lab.' Following this, I saved the configuration and initiated the job.

![6  Running job](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/cd1375c3-da9d-4c2d-b4a9-023151e900fa)

# Step 6

Subsequently, I accessed Athena to query the 'glue_ticket_db' database. I adjusted my editor settings to store the query results in my 'athena-bucket.' I employed the CTAS query to create a new table populated with the results of a SELECT query.

![7  Picture 7](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/92f67207-a64f-45b5-b084-ccaee00a48f0)

# Step 7

For my next query, I executed:

``` SQL
SELECT RequestType, COUNT(*) 
FROM "requests"   
GROUP BY RequestType
ORDER BY 2 DESC;
```

Following this, I established a new Data Source within the Amazon Athena console, specifying DynamoDB and naming it 'athena-federated-dynamodb.' I then selected my 'athena_connector_function' (Lambda) to run federated queries, serving as the code to mediate between my target data source and Athena. I subsequently modified my Data source in the Athena editor from 'AwsDataCatalog' to 'athena-federated-dynamodb' and executed another query.

![8  Picture 8](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/74d543d3-84b4-4453-863b-5f9986bb6e7a)

Finally, I created an additional table named 'diytable' using the CTAS command. I specified the 'newtable' folder as the external location:

``` SQL
CREATE TABLE AwsDataCatalog.glue_ticket_db.DIYTable
WITH (
        external_location = 's3://data-lake-bucket-714823731171-us-east-1/newtable'
        )
AS SELECT *
FROM "glue-etl-ticket-table";
```

![Picture 9](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/24ea8e5a-ccf6-4926-ab41-80e837c5190d)
