# Templating a Serverless Application

The customer is currently facing challenges in managing coffee orders for a group of coders participating in a contest to create the best scheduling app. In response to this need, the CEO has decided to develop a serverless application architecture using Amazon Web Services (AWS) components, specifically Amazon API Gateway, AWS Lambda, and Amazon DynamoDB. To streamline the development process, we will utilize the AWS Serverless Application Model (AWS SAM) for its shorthand syntax, which simplifies the creation of functions, APIs, databases, and event source mappings.

# Lab Architecture:

![0](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/26790762-77b0-4646-ab16-366e5afd2230)

# Step 1 - Setting up the Cloud9 Environment:

Opened AWS Cloud9 and created a new environment named "lab-Cloud9-IDE." I selected a t2.micro instance running Amazon Linux 2 with SSH connection type. To run this environment within Subnet 1 of my Lab Virtual Private Cloud (VPC).

![1](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/1fc09d5f-8417-4165-ad39-e6a95e91b566)

# Step 2 - Creating a Lambda SAM Application:

- Added the "us-east-1" region via the AWS tab in Cloud9.
- Expanded the selected "us-east-1" region and chose to create a Lambda SAM Application.
- Selected Python 3.7 as the runtime and used the AWS SAM Hello World template, naming the application "appfn."

# Step 3 - Adding Dependencies and Deploying:

- Added the following text to the "requirements.txt" file: urllib3==1.26.15
- Returned to the AWS tab, navigated to Lambda, and deployed the SAM application.
- Deployed the "appfn/template.yaml" template and deployed to my code bucket, naming the deployed stack "appfn."

![3](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/e34e9aff-ea5b-47d0-9949-163760966120)

# Step 4 - Writing Lambda Function Code:

In the "app.py" window of the Cloud9 IDE, I added the following Python code for the Lambda function. The commented lines would be changed as directed for the later portion of the lab.

```python
# Runtime: Python 3.6
# This Lambda function displays informational messages about the state of the application.
# Please review the comments for each code block to help you understand the execution of this Lambda function.

import os
import json
import boto3
import logging
from datetime import datetime
from botocore.exceptions import ClientError

# AWS Lambda Function Logging in Python - More info: https://docs.aws.amazon.com/lambda/latest/dg/python-logging.html
logger = logging.getLogger()
logger.setLevel(logging.INFO)


session = boto3.Session()
# Boto3 - DynamoDB Client - Mode Info: https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/dynamodb.html
dynamodb = session.resource("dynamodb")

# You can use SAM (template.yaml) to automatically create a table in DynamoDB 
# (AWS :: Serverless :: SimpleTable), to choose the name remember to use the TableName parameter
# https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-resource-simpletable.html
# Table name to fetch a message
table_name = 'app_table'

app_table = dynamodb.Table(table_name)

# The Lambda handler is the first function that executes in your code.
def lambda_handler(event, context):

    # context – AWS Lambda uses this parameter to provide runtime information to your handler.
    # event – AWS Lambda uses this parameter to pass in event data to the handler. This parameter is usually of the Python dict type.
    # AWS Lambda function Python handler - https://docs.aws.amazon.com/lambda/latest/dg/python-handler.html
    
    # Event received from API Gateway Lambda Proxy Integration
    #
    #   In Lambda proxy integration, when a client submits an API request, API Gateway passes 
    # to the integrated Lambda function the raw request as-is in event parameter
    # Input Format https://docs.aws.amazon.com/apigateway/latest/developerguide/set-up-lambda-proxy-integrations.html#api-gateway-simple-proxy-for-lambda-input-format

    print('Received event: ' + json.dumps(event, indent=2))
    
    # variable with informational message - This message will be printed if there is no Query String
    msg = 'Template message created' 

    # On the World Wide Web, a query string is a part of a uniform resource locator (URL) that assigns values to specified parameters.
    # https://en.wikipedia.org/wiki/Query_string    
    queryStringParameters = event.get('queryStringParameters')
    if queryStringParameters is not None:
        # Found the QueryStringParameter, searching for the id key, for example, Query String => id=1
        id = queryStringParameters.get('id')
        if id is not None:
            # Found the id, usually you use the query string parameter to filter, paginate 
            # or search the database for information, in this lab we will only display a message.
            
            # Uncomment the two lines below if you configured DynamoDB in the SAM template
            # str_id = str(id) # convert id to string to match with DynamoDB partition key type
            # msg = 'Template with message {}'.format(get_item(str_id))  
            
            # Comment the line below if you going to use DynamoDB
            msg = 'Template with message id = {}'.format(id)
    
    
    #   In Lambda proxy integration, API Gateway requires the backend Lambda function to return output
    # to API Gateway, then API Gateway sends the return to the client
    # Output format: https://docs.aws.amazon.com/apigateway/latest/developerguide/set-up-lambda-proxy-integrations.html#api-gateway-simple-proxy-for-lambda-output-format
    response = {
        'statusCode': '200',
        'body': msg,
        'headers': {
            'Content-Type': 'application/json',
        }
    }
    
    return response
```

# Step 5 - Redeploying the SAM Application:

After saving the code, I redeployed the SAM Application, naming it "appfn-new."

![4](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/8fb4ae59-b1d1-4fc1-86d8-2cc56b4f2bca)

# Step 6 - Invoking the Lambda Function:

I navigated to the "appfn-new" Lambda application in Cloud9 and selected the function to invoke on AWS.

![5](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/1f458c26-66e5-4dc8-b44b-471dffe67421)

# Step 7 - Testing the API Gateway:

I navigated to the API Gateway subdirectory in the "us-east-1" region and chose to invoke the "appfn-new" selection. I selected the "/hello" resource and the "GET" method, then clicked "Invoke." I tested with empty query strings and "id=1."

# Step 8 - Creating an AWS CodeCommit Repository:

I opened a new terminal and ran commands to create and configure an AWS CodeCommit repository:

```bash
cd appfn

aws codecommit create-repository --repository-name my-app

git config --global credential.helper '!aws codecommit credential-helper $@'

git config --global credential.UseHttpPath true

git config --global init.defaultBranch main

git init

git add .

git commit -m "first commit"

git remote add origin https://git-codecommit.us-east-1.amazonaws.com/v1/repos/my-app

git push -u origin main
```

# Step 9 - Modifying the SAM Template:

I edited the "template.yaml" file in my "appfn" folder and ensured my DynamoDB table was named "app_table." I also made the necessary changes in the Lambda code covered in the commented lines in the Lambda function.

```yaml
# AWS SAM Template

# ...

Resources:
  HelloWorldFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: hello_world/
      Handler: app.lambda_handler
      Runtime: python3.7
      Events:
        HelloWorld:
          Type: Api
          Properties:
            Path: /hello
            Method: get

  AppTable:
    Type: AWS::DynamoDB::Table
    Properties:
      TableName: app_table
      AttributeDefinitions:
        - AttributeName: id
          AttributeType: S
      KeySchema:
        - AttributeName: id
          KeyType: HASH
      ProvisionedThroughput:
        ReadCapacityUnits: 5
        WriteCapacityUnits: 5
```
