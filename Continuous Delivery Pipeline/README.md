# Continuous Delivery Pipeline

The customer's requirement is to scale their development rapidly, and they are seeking a tool to build, test, and deploy software whenever a code change is made. To accomplish this, we will use AWS CodePipeline to orchestrate each step in the release process. We will configure CodeCommit as the pipeline source to push code changes immediately. Next, we will launch CodeBuild to build the application. Finally, we will use AWS CloudFormation to deploy the build artifacts using infrastructure as code.

# Lab Architecture

![0](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/7efc05dd-507a-40cb-9f7b-ce2bf784f06f)

# Step 1: Setup Repository
First, I navigated to CodeCommit and created a repository. Then, I went to Cloud9 and opened my Continuous Delivery Pipeline Lab environment. I cloned my repository using its HTTPS link.

![1](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/3c6f99df-9dda-49bd-8aeb-297c38dd802c)

# Step 2: Prepare AWS Resources
I ran "sam init" and chose the AWS Quick Start Template, selecting the Python 3.8 runtime. I reviewed the app.py and template.yaml files before running "aws s3 mb s3://kevinwynn02102017" to create a new bucket.

![2](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/3dc65e05-47e5-4822-979a-53488aaf6735)

# Step 3: Define Build Specification
I created a new file in my repository called buildspec.yaml and added and saved the following code:

```yaml
version: 0.2

phases:
  install:
    runtime-versions:
       python: 3.8
  build:
    commands:
      - pip install aws-sam-cli
      - sam build -t sam-app/template.yaml
      - sam package --template-file sam-app/template.yaml --output-template-file outputtemplate.yaml --s3-bucket kevinwynn02102017
artifacts:
  files: 
    - outputtemplate.yaml
```

# Step 4: Commit Changes
I added changes in the working directory to the staging area, created a commit in my local repository, and synchronized my local and remote repositories with the following commands:

```bash
- git add .
- git commit -a -m "first commit"
- git push
```

# Step 5: Configure CodeBuild
I navigated to CodeBuild in the console and created a build project using the Ubuntu OS and the Linux EC2 environment type.

![5](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/81c1b6d1-6ed1-49e9-a8b4-78fe9a4f425c)

# Step 6: Set Up CodePipeline
Next, I went to CodePipeline and created a pipeline specifying AWS CodeCommit as the source provider, AWS CodeBuild as the build provider, and AWS CloudFormation as the deployment provider.

![6](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/073dbcbb-a39a-4d03-8636-1adf9243faca)

# Step 7: Add Manual Approval
I edited my pipeline to add an additional stage for manual approval. I then selected "release change" on my pipeline to execute the new effect.

![7](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/173cbfe6-22ca-4704-adfe-5fa6cf7cf719)

# Step 8: Test the Pipeline
I then tested my pipeline by changing the "Hello World" text in my app.py code and repeated the earlier steps:

```bash
- git add .
- git commit -a -m "first commit"
- git push
```

![8](https://github.com/kevin-wynn-cloud/AWS-Projects/assets/144941082/6cf4556b-a855-48f7-9994-6654f9965354)
