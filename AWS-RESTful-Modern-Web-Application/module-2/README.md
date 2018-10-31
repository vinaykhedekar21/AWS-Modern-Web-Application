# Module 2: Creating a Service with AWS Fargate

![Architecture](/images/module-2/architecture-module-2.png)


**Services used:**
* [AWS CloudFormation](https://aws.amazon.com/cloudformation/)
* [AWS Identity and Access Management (IAM)](https://aws.amazon.com/iam/)
* [Amazon Virtual Private Cloud (VPC)](https://aws.amazon.com/vpc/)
* [Amazon Elastic Load Balancing](https://aws.amazon.com/elasticloadbalancing/)
* [Amazon Elastic Container Service (ECS)](https://aws.amazon.com/ecs/)
* [AWS Fargate](https://aws.amazon.com/fargate/)
* [AWS Elastic Container Registry (ECR)](https://aws.amazon.com/ecr/)
* [Amazon DynamoDB](https://aws.amazon.com/dynamodb/)
* [Amazon Cognito](http://aws.amazon.com/cognito/)
* [Amazon API Gateway](https://aws.amazon.com/api-gateway/)
* [Amazon Simple Storage Service (S3)](https://aws.amazon.com/s3/)
* [AWS Kinesis Data Firehose](https://aws.amazon.com/kinesis/data-firehose/)
* [AWS Lambda](https://aws.amazon.com/lambda/)
* [AWS CodeCommit](https://aws.amazon.com/codecommit/)
* [AWS CodePipeline](https://aws.amazon.com/codepipeline/)
* [AWS CodeDeploy](https://aws.amazon.com/codedeploy/)
* [AWS CodeBuild](https://aws.amazon.com/codebuild/)


### Overview

In Module 2, we will create a new microservice hosted using [AWS Fargate](https://aws.amazon.com/fargate/) on [Amazon Elastic Container Service](https://aws.amazon.com/ecs/) so that your Mythical Mysfits website can have a application backend to integrate with. 
** we will use Python and create a Flask app in a Docker container behind a Network Load Balancer. 
** These will form the microservice backend for the frontend website to integrate with.

### Create the Core Infrastructure using AWS CloudFormation

* [**An Amazon VPC**](https://aws.amazon.com/vpc/) 
* [**Two NAT Gateways**](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html) (one for each public subnet, also panning multiple AZs) 
* [**A DynamoDB VPC Endpoint**](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/vpc-endpoints-dynamodb.html) - our microservice backend will eventually integrate with [Amazon DynamoDB](https://aws.amazon.com/dynamodb/) for persistence (as part of module 3).
* [**A Security Group**](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html) - Allows your docker containers to receive traffic on port 8080 from the Internet through the Network Load Balancer.
* [**IAM Roles**](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html) - Identity and Access Management Roles are created. These will be used throughout the workshop to give AWS services or resources you create access to other AWS services like DynamoDB, S3, and more.

## Module 2a: Deploying a Service with AWS Fargate

### Creating a Flask Service Container

#### Building A Docker Image

docker build . -t REPLACE_ME_ACCOUNT_ID.dkr.ecr.REPLACE_ME_REGION.amazonaws.com/mythicalmysfits/service:latest
```
#### Pushing the Docker Image to Amazon ECR

aws ecr create-repository --repository-name mythicalmysfits/service
```
docker push REPLACE_ME_WITH_DOCKER_IMAGE_TAG
```

#### Create an ECS Cluster

aws ecs create-cluster --cluster-name MythicalMysfits-Cluster
```

#### Create an AWS CloudWatch Logs Group

```
aws logs create-log-group --log-group-name mythicalmysfits-logs
```

#### Register an ECS Task Definition

```
aws ecs register-task-definition --cli-input-json file://~/environment/aws-modern-application-workshop/module-2/aws-cli/task-definition.json
```

### Enabling a Load Balanced Fargate Service

#### Create a Network Load Balancer

To provision a new NLB, execute the following CLI command in the Cloud9 terminal (retrieve the subnetIds from the CloudFormation output you saved):

```
aws elbv2 create-load-balancer --name mysfits-nlb --scheme internet-facing --type network --subnets REPLACE_ME_PUBLIC_SUBNET_ONE REPLACE_ME_PUBLIC_SUBNET_TWO > ~/environment/nlb-output.json
```

#### Create a Load Balancer Target Group

```
aws elbv2 create-target-group --name MythicalMysfits-TargetGroup --port 8080 --protocol TCP --target-type ip --vpc-id REPLACE_ME_VPC_ID --health-check-interval-seconds 10 --health-check-path / --health-check-protocol HTTP --healthy-threshold-count 3 --unhealthy-threshold-count 3 > ~/environment/target-group-output.json
```

#### Create a Load Balancer Listener

```
aws elbv2 create-listener --default-actions TargetGroupArn=REPLACE_ME_NLB_TARGET_GROUP_ARN,Type=forward --load-balancer-arn REPLACE_ME_NLB_ARN --port 80 --protocol TCP
```

### Creating a Service with Fargate

#### Creating a Service Linked Role for ECS

```
aws iam create-service-linked-role --aws-service-name ecs.amazonaws.com
```

#### Create the Service

```
aws ecs create-service --cli-input-json file://~/environment/aws-modern-application-workshop/module-2/aws-cli/service-definition.json
```

After your service is created, ECS will provision a new task that's running the container you've pushed to ECR, and register it to the created NLB.  

#### Test the Service

#### Upload to S3
To upload this file to your S3 hosted website, use the bucket name again that was created during Module 1, and run the following command:


## Module 2b: Automating Deployments using AWS Code Services

![Architecture](/images/module-2/architecture-module-2b.png)


### Creating the CI/CD Pipeline

#### Create a S3 Bucket for Pipeline Artifacts


```
aws s3 mb s3://REPLACE_ME_CHOOSE_ARTIFACTS_BUCKET_NAME
```

Next, this bucket needs a bucket policy to define permissions for the data stored within it. 
```
aws s3api put-bucket-policy --bucket REPLACE_ME_ARTIFACTS_BUCKET_NAME --policy file://~/environment/aws-modern-application-workshop/module-2/aws-cli/artifacts-bucket-policy.json
```

#### Create a CodeCommit Repository

You'll need a place to push and store your code in. Create an [**AWS CodeCommit Repository**](https://aws.amazon.com/codecommit/) using the CLI for this purpose:

```
aws codecommit create-repository --repository-name MythicalMysfitsService-Repository
```

#### Create a CodeBuild Project

With a repository to store our code in, and an S3 bucket that will be used for our CI/CD artifacts, lets add to the CI/CD stack with a way for a service build to occur.  This will be accomplished by creating an [**AWS CodeBuild Project**](https://aws.amazon.com/codebuild/).  Any time a build execution is triggered, AWS CodeBuild will automatically provision a build server to our configuration and execute the steps required to build our docker image and push a new version of it to the ECR repository we created (and then spin the server down when the build is completed).  
```
aws codebuild create-project --cli-input-json file://~/environment/aws-modern-application-workshop/module-2/aws-cli/code-build-project.json
```

#### Create a CodePipeline Pipeline

Finally, we need a way to *continuously integrate* our CodeCommit repository with our CodeBuild project so that builds will automatically occur whenever a code change is pushed to the repository.  Then, we need a way to *continuously deliver* those newly built artifacts to our service in ECS.  [**AWS CodePipeline**](https://aws.amazon.com/codepipeline/) is the service that glues these actions together in a **pipeline** you will create next.  

```
aws codepipeline create-pipeline --cli-input-json file://~/environment/aws-modern-application-workshop/module-2/aws-cli/code-pipeline.json
```

#### Enable Automated Access to ECR Image Repository

We have one final step before our CI/CD pipeline can execute end-to-end successfully. With a CI/CD pipeline in place, you won't be manually pushing container images into ECR anymore.  CodeBuild will be pushing new images now. We need to give CodeBuild permission to perform actions on your image repository with an **ECR repository policy***. 

```
aws ecr set-repository-policy --repository-name mythicalmysfits/service --policy-text file://~/environment/aws-modern-application-workshop/module-2/aws-cli/ecr-policy.json
```

### Test the CI/CD Pipeline

#### Using Git with AWS CodeCommit
To test out the new pipeline, we need to configure git within your Cloud9 IDE and integrate it with your CodeCommit repository.

AWS CodeCommit provides a credential helper for git that we will use to make integration easy.  Run the following commands in sequence the terminal to configure git to be used with AWS CodeCommit (neither will report any response if successful):

```
git config --global user.name "REPLACE_ME_WITH_YOUR_NAME"
```

```
git config --global user.email REPLACE_ME_WITH_YOUR_EMAIL@example.com
```

```
git config --global credential.helper '!aws codecommit credential-helper $@'
```

```
git config --global credential.UseHttpPath true
```

Next change directories in your IDE to the environment directory using the terminal:

```
cd ~/environment/
```

Now, we are ready to clone our repository using the following terminal command:

```
git clone https://git-codecommit.REPLACE_REGION.amazonaws.com/v1/repos/MythicalMysfitsService-Repository
```

This will tell us that our repository is empty!  Let's fix that by copying the application files into our repository directory using the following command:

```
cp -r ~/environment/aws-modern-application-workshop/module-2/app/* ~/environment/MythicalMysfitsService-Repository/
```

#### Pushing a Code Change

Now the completed service code that we used to create our Fargate service in Module 2 is stored in the local repository that we just cloned from AWS CodeCommit.  

```
cd ~/environment/MythicalMysfitsService-Repository/
```

Then, run the following git commands to push in your code changes.  

```
git add .
git commit -m "I changed the age of one of the mysfits."
git push
```

After the change is pushed into the repository, you can open the CodePipeline service in the AWS Console to view your changes as they progress through the CI/CD pipeline. 