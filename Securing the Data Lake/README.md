# Securing the Data Lake

In this guide, we will walk through the process of securing a data lake for a customer who wishes to add security compliance measures. The objective is to incorporate data sources containing sensitive information about customers and employees while restricting access to certain aspects of the data, specifically certain table columns. To achieve this, we will leverage AWS Lake Formation, which allows for the implementation of granular controls and restrictions.

# Lab Architecture 
![0  Picture 0](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/26386c18-1f02-45ef-925c-2d30201d4cf5)

# Step 1: Setting Up the Data Ingestion Process

To begin, we need to establish the foundation for data ingestion:

1.1. Creating a Crawler for Data Ingestion

Navigate to AWS Glue.
- Create a crawler named "OpsIngestionCrawler."
- Choose the customer's "operations-datalake" S3 bucket as the data source.
- Assign the "OpsIngestionCrawlerRole" role.
- Create a new database in the AWS Glue Catalog, naming it "ops_data_ingestion" and designate it as the target database.
- Execute the crawler to catalog the data.

![1  Picture 1](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/8b567dc9-5bae-49ca-bc21-6507630de17f)

# Step 2: Implementing Granular Access Controls

Now, let's set up access controls to restrict certain aspects of the data:

- Go to AWS Lake Formation.
- Add yourself as the initial admin.
- Select the "openeration_datalake" table.
- Grant permissions to the "opsUser" IAM role.
- Create a new data filter named "ops_filter."
- Exclude Personally Identifiable Information (PII) columns using the row filter expression "true."

![2  Picture 2](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/c1c4264d-9b47-479f-9d36-0e3a1e3c4d0c)

# Step 3: Extending Access to opsDirector and OpsManager

Now, let's grant specific access to the opsDirector and OpsManager roles:

- Provide permissions to the opsDirector and OpsManager roles.
- Verification of Access in Amazon Athena
- Finally, ensure that the implemented permissions and filters are working as intended by logging into the IAM users.
- Log in as each IAM user (opsUser, opsDirector, and OpsManager).
- Verify their respective access levels in Amazon Athena.

![3  Picture 3](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/db4019e3-b20a-4bf0-9e5f-a93c2160b107)

By following these steps, you have successfully secured the data lake using AWS Lake Formation, granting access to authorized users while safeguarding sensitive information.
