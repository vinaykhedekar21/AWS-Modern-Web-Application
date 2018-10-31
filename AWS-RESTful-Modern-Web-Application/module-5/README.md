# Module 5: Capturing User Behavior

![Architecture](/images/module-5/architecture-module-5.png)

**Services used:**
* [AWS CloudFormation](https://aws.amazon.com/cloudformation/)
* [AWS Kinesis Data Firehose](https://aws.amazon.com/kinesis/data-firehose/)
* [Amazon S3](https://aws.amazon.com/s3/)
* [Amazon API Gateway](https://aws.amazon.com/api-gateway/)
* [AWS Lambda](https://aws.amazon.com/lambda/)
* [AWS CodeCommit](https://aws.amazon.com/codecommit/)
* [AWS Serverless Appliation Model (AWS SAM)](https://github.com/awslabs/serverless-application-model)
* [AWS SAM Command Line Interface (SAM CLI)](https://github.com/awslabs/aws-sam-cli)


Modern application design principles prefer focused, decoupled, and modular services.  So rather than add additional methods and capabilities within the existing Mysfits service that you have been working with so far, we will create a new and decoupled service for the purpose of receiving user click events from the Mysfits website.  This full stack has been represented using a provided CloudFormation template.

* An [**AWS Kinesis Data Firehose delivery stream**](https://aws.amazon.com/kinesis/data-firehose/): 
* An [**Amazon S3 bucket**](https://aws.amazon.com/s3/): 
* An [**AWS Lambda function**](https://aws.amazon.com/lambda/): 
* An [**Amazon API Gateway REST API**](https://aws.amazon.com/api-gateway/): 
* [**IAM Roles**](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html): 

#### Create a new CodeCommit Repository

```
aws codecommit create-repository --repository-name MythicalMysfitsStreamingService-Repository
```

### Update the Lambda Function Package and Code

#### Use pip to Intall Lambda Function Dependencies

```
pip install requests -t .
```

#### Push Your Code into CodeCommit
Let's commit our code changes to the new repository so that they're saved in CodeCommit:

```
git add .
```

```
git commit -m "New stream processing service."
```

```
git push
```

### Creating the Streaming Service Stack


#### Create an S3 Bucket for Lambda Function Code Packages

```
aws s3 mb s3://REPLACE_ME_YOUR_BUCKET_NAME/
```

#### Use the SAM CLI to Package your Code for Lambda

```
sam package --template-file ./real-time-streaming.yml --output-template-file ./transformed-streaming.yml --s3-bucket REPLACE_ME_YOUR_BUCKET_NAME
```

#### Deploy the Stack using AWS CloudFormation

```
aws cloudformation deploy --template-file /home/ec2-user/environment/MythicalMysfitsStreamingService-Repository/transformed-streaming.yml --stack-name MythicalMysfitsStreamingStack --capabilities CAPABILITY_IAM
```

### Sending Mysfit Profile Clicks to the Service

Perform the following command for the new streaming stack to retrieve the new API Gateway endpoint for your stream processing service:

```
aws cloudformation describe-stacks --stack-name MythicalMysfitsStreamingStack
```

#### Push the New Site Version to S3

```
aws s3 cp ~/environment/aws-modern-application-workshop/module-5/web/index.html s3://YOUR-S3-BUCKET/
```
