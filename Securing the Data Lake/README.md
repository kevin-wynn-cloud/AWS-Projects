# Securing the Data Lake

In this guide, we will walk through the process of securing a data lake for a customer who wishes to add security compliance measures. The objective is to incorporate data sources containing sensitive information about customers and employees while restricting access to certain aspects of the data, specifically certain table columns. To achieve this, we will leverage AWS Lake Formation, which allows for the implementation of granular controls and restrictions.

# Lab Architecture 

![1](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/627d6e64-a8c4-4764-95f6-681d8a14be2c)

# Step 1: Setting Up the Data Ingestion Process

To begin, we need to establish the foundation for data ingestion:

1.1. Creating a Crawler for Data Ingestion

Navigate to AWS Glue.
- Create a crawler named "OpsIngestionCrawler."
- Choose the customer's "operations-datalake" S3 bucket as the data source.
- Assign the "OpsIngestionCrawlerRole" role.
- Create a new database in the AWS Glue Catalog, naming it "ops_data_ingestion" and designate it as the target database.
- Execute the crawler to catalog the data.

![2](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/8acff6ba-5fe7-4fd6-be67-63e452f1a7da)

# Step 2: Implementing Granular Access Controls

Now, let's set up access controls to restrict certain aspects of the data:

- Go to AWS Lake Formation.
- Add yourself as the initial admin.
- Select the "openeration_datalake" table.
- Grant permissions to the "opsUser" IAM role.
- Create a new data filter named "ops_filter."
- Exclude Personally Identifiable Information (PII) columns using the row filter expression "true."

![3](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/02b256c5-e17f-41f6-b91f-7fceeebc6e5c)

# Step 3: Extending Access to opsDirector and OpsManager

Lastly, let's grant specific access to the opsDirector and OpsManager roles:

- Provide permissions to the opsDirector and OpsManager roles.
- Verification of Access in Amazon Athena
- Finally, ensure that the implemented permissions and filters are working as intended by logging into the IAM users.
- Log in as each IAM user (opsUser, opsDirector, and OpsManager).
- Verify their respective access levels in Amazon Athena.

![4](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/e20359ce-ca79-43f3-8351-8eaaec082510)

By following these steps, you have successfully secured the data lake using AWS Lake Formation, granting access to authorized users while safeguarding sensitive information.
