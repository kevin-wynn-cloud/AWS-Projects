# Deploying APIs Gradually

The customer is planning to migrate to a serverless architecture, and our goal is to recommend the use of AWS Lambda functions to expose services as APIs through Amazon API Gateway. We also aim to ensure fault tolerance and high availability. Additionally, the customer needs the ability to deploy APIs gradually, which will be achieved using AWS CodeDeploy, as it's integrated with Lambda.

# Lab Architecture

![0](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/d7b49729-3dab-42e2-8f05-a51074ef0213)


# Step 1: Checking Environment

- Navigated to the 'Deploying API Gradually' Cloud9 environment.
- Verified that Python 3.8.16 and SAM CLI version 1.72.0 are installed for later steps.

# Step 2: Initializing the SAM Template

- Executed the command sam init.
- Selected the AWS Quick Start Template and the Hello World Example application template.
- Named the project 'MyAPI'.

![2](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/0d5af660-bb1c-43f3-a49b-5633f897620f)

# Step 3: Reviewing the Lambda Function Code

- Navigated to the 'app.py' function and reviewed the code provided:

```python
import json

def lambda_handler(event, context):
    return {
        "statusCode": 200,
        "body": json.dumps({
            "message": "hello world",
        }),
    }
```

# Step 4: Building the Serverless Application

- Reviewed the 'template.yaml' file for configuration details.
- Executed the command sam build to build the serverless application.

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: >
  MyAPI

  Sample SAM Template for MyAPI

# More info about Globals: https://github.com/awslabs/serverless-application-model/blob/master/docs/globals.rst
Globals:
  Function:
    Timeout: 3
    MemorySize: 128

Resources:
  HelloWorldFunction:
    Type: AWS::Serverless::Function # More info about Function Resource: https://github.com/awslabs/serverless-application-model/blob/master/versions/2016-10-31.md#awsserverlessfunction
    Properties:
      CodeUri: hello_world/
      Handler: app.lambda_handler
      Runtime: python3.8
      Architectures:
        - x86_64
      Events:
        HelloWorld:
          Type: Api # More info about API Event Source: https://github.com/awslabs/serverless-application-model/blob/master/versions/2016-10-31.md#api
          Properties:
            Path: /hello
            Method: get

Outputs:
  # ServerlessRestApi is an implicit API created out of Events key under Serverless::Function
  # Find out more about other implicit resources you can reference within SAM
  # https://github.com/awslabs/serverless-application-model/blob/master/docs/internals/generated_resources.rst#api
  HelloWorldApi:
    Description: "API Gateway endpoint URL for Prod stage for Hello World function"
    Value: !Sub "https://${ServerlessRestApi}.execute-api.${AWS::Region}.amazonaws.com/Prod/hello/"
  HelloWorldFunction:
    Description: "Hello World Lambda Function ARN"
    Value: !GetAtt HelloWorldFunction.Arn
  HelloWorldFunctionIamRole:
    Description: "Implicit IAM Role created for Hello World function"
    Value: !GetAtt HelloWorldFunctionRole.Arn
```

# Step 5: Deploying the Application

- Ran the command sam deploy --guided to enter interactive mode.
- Named the stack 'MyAPIStack'.
- Selected the 'us-east-1' region and provided necessary configuration options (prompted).

# Step 6: Adjusting Deployment Preferences

- Edited the 'template.yaml' file to change the deployment preference to 'Linear10PercentEvery1Minute'.

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: >
  MyAPI

  Sample SAM Template for MyAPI

# More info about Globals: https://github.com/awslabs/serverless-application-model/blob/master/docs/globals.rst
Globals:
  Function:
    Timeout: 3
    MemorySize: 128

Resources:
  HelloWorldFunction:
    Type: AWS::Serverless::Function # More info about Function Resource: https://github.com/awslabs/serverless-application-model/blob/master/versions/2016-10-31.md#awsserverlessfunction
    Properties:
      CodeUri: hello_world/
      Handler: app.lambda_handler
      Runtime: python3.
      AutoPublishAlias: live
      DeploymentPreference:
        Type: Linear10PercentEvery1Minute
      Architectures:
        - x86_64
      Events:
        HelloWorld:
          Type: Api # More info about API Event Source: https://github.com/awslabs/serverless-application-model/blob/master/versions/2016-10-31.md#api
          Properties:
            Path: /hello
            Method: get

Outputs:
  # ServerlessRestApi is an implicit API created out of Events key under Serverless::Function
  # Find out more about other implicit resources you can reference within SAM
  # https://github.com/awslabs/serverless-application-model/blob/master/docs/internals/generated_resources.rst#api
  HelloWorldApi:
    Description: "API Gateway endpoint URL for Prod stage for Hello World function"
    Value: !Sub "https://${ServerlessRestApi}.execute-api.${AWS::Region}.amazonaws.com/Prod/hello/"
  HelloWorldFunction:
    Description: "Hello World Lambda Function ARN"
    Value: !GetAtt HelloWorldFunction.Arn
  HelloWorldFunctionIamRole:
    Description: "Implicit IAM Role created for Hello World function"
    Value: !GetAtt HelloWorldFunctionRole.Arn
```
  
# Step 7: Validating and Redeploying the SAM Template

- Ran sam validate to validate the modified SAM template.
- Executed sam build to rebuild the SAM.
- Deployed the updated SAM with sam deploy.

![7](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/dd5f5adc-cc40-40e5-bb3c-688714e43b9d)

# Step 8: Monitoring the Deployment and Function Aliases

- Navigated to AWS CodeDeploy in the console to monitor the deployment using the graphical user interface.
- Accessed AWS Lambda in the console to review function aliases.

![8](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/47852863-8710-4fa0-8a01-0019f4467a44)

![9](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/80a31f30-e866-4f44-b2f1-ed2978d5bc9a)

# Conclusion: 

The migration to a serverless architecture using AWS Lambda and Amazon API Gateway, with gradual deployment facilitated by AWS CodeDeploy, is on track, with fault tolerance and high availability built into the system.
