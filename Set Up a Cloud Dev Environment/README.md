# Setting Up a Cloud Development Environment

An educational company aims to provide students with an immersive cloud development environment for coding. Setting up multiple local development environments with various plugins and stack configurations can be cumbersome. To streamline this process, the company has decided to use AWS Cloud9. AWS Cloud9 offers immediate access to a fully configured Integrated Development Environment (IDE) with pre-installed runtimes, package managers, debugging tools, and support for over 40 programming languages. This lab write-up documents the steps taken to create a cloud development environment and complete a Python function for uploading files to Amazon S3.

# Lab Architecture:

![picture0](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/3a4faaf3-358c-4029-b7e5-47d2e57f2e6a)

# Step 1: Creating a Cloud9 Environment

- Navigated to AWS Cloud9.
- Created a new environment using the Amazon Linux 2 platform, SSH connection, and a t2.micro instance type.
- Created a new Python file template and added a print statement.

![picture1](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/ac40c97e-bd3c-4d57-baaa-5954d737f320)

# Step 2: Cloning Code from a Git Repository

- Cloned code from a Git repository using the following command in the bash terminal:

```bash
git clone https://git-codecommit.us-east-1.amazonaws.com/v1/repos/lab
``` 

```python
"""
This is an example of a program for your first day at Coders Campus.
It starts by defining a message (msg) and a file name (filename), then generates a banner with the message, prints the banner and saves it the file in the local directory.
The program defines a bucket (bucket_name) and uploads the file generated in the filesystem location to that bucket on Amazon S3.
"""

import pyfiglet
import logging
import boto3
from botocore.exceptions import ClientError


def upload_file(file_name, bucket, object_name=None):
    """
    Upload a file to an S3 bucket

    :param file_name: File to upload
    :param bucket: Bucket to upload to
    :param object_name: S3 object name. If not specified then file_name is used
    :return: True if file was uploaded, else False
    """
    
    
       
    # YOUR CODE HERE
    # Reference: https://boto3.amazonaws.com/v1/documentation/api/latest/guide/s3-uploading-files.html
    
    return False
    

def save_file(content, filename):
    """
    Saves file in the local filesystem
    
    :param content: the content to be saved inside the file
    :param filename: the name of the file in the local filesystem
    """
    f = open(filename, 'w')
    f.write(content)
    f.close()

def generate_banner(msg):
    """"
    Generates a banner from the message
    
    :param msg: message to be formatted as a banner
    :return: the message formatted as a banner
    """
    ascii_banner = pyfiglet.figlet_format(msg)
    return ascii_banner


def main():
    """"
    The main entry point when the python is executed
    """
    msg = "Hello World!"
    filename = "hello.txt"
    
    # Generates a banner from the msg
    banner = generate_banner(msg)
    
    # Prints the banner in the screen
    print(banner)
    
    # Saves the content of the banner in a file in local filesystem with the filename
    save_file(banner, filename)
    
    # Replace YOUR_BUCKET_NAME with the bucket name you created
    bucket_name = 'YOUR_BUCKET_NAME'
    
    # Uploads the filename into S3 Bucket name
    if upload_file(filename,bucket_name): 
      print("Upload successful")
    else:
      print("Upload Not Implemented yet")


if __name__ == "__main__":
    main()
```

![picture3](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/1c5cbb16-cb7a-469f-b998-72553c872eb9)

# Step 3: Creating an S3 Bucket

- Created an S3 bucket using the AWS CLI with the following command:

```bash
aws s3 mb s3://mylabbucket198912

s3://mylabbucket198912
```

![picture4](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/cd49d814-e899-45d1-9da0-70094cd11aee)

# Step 4: Version Control and Commit

- Changed the working directory to the lab directory.
- Checked the Git status.
- Added the hello.py script to the staging area.
- Committed the code to the Git repository with a descriptive comment.
- Pushed the changes to the remote repository using the following commands:

```bash
git add hello.py
git commit -m "Improved upload message"
git push

```

![picture5](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/b1192c3f-ee7a-4452-89b0-53439f19df8a)

# Step 5: Modifying the Upload Function

The following section of code in the hello.py script was modified:

```python
def upload_file(file_name, bucket, object_name=None):
    """
    Upload a file to an S3 bucket

    :param file_name: File to upload
    :param bucket: Bucket to upload to
    :param object_name: S3 object name. If not specified, file_name is used
    :return: True if file was uploaded, else False
    """

    # If S3 object_name was not specified, use file_name
    if object_name is None:
        object_name = os.path.basename(file_name)

    # Upload the file
    s3_client = boto3.client('s3')

    try:
        s3_client.upload_file(
            file_name,
            bucket,
            object_name,
            Callback=ProgressPercentage(file_name)
        )
    except ClientError as e:
        logging.error(f"Error uploading file to {bucket}/{object_name}: {e}")
        return False
    return True
```

# Conclusion:

In this lab, we successfully set up a cloud development environment using AWS Cloud9, cloned code from a Git repository, configured an S3 bucket, and made improvements to the Python script for uploading files to Amazon S3. This environment will facilitate coding exercises and projects for students in a hassle-free manner.
