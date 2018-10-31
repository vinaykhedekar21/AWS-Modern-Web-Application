# Module 3 - Adding a Data Tier with Amazon DynamoDB

![Architecture](/images/module-3/architecture-module-3.png)

### Overview

** Now that you have a service deployed and a working CI/CD pipeline to deliver changes to that service automatically whenever you update your code repository, you can quickly move new application features from conception to available for your Mythical Mysfits customers.  
** In this module you will create a table in [Amazon DynamoDB](https://aws.amazon.com/dynamodb/), a managed and scalable NoSQL database service on AWS with super fast performance.  

### Adding a NoSQL Database

#### Create a DynamoDB Table

To create the table using the AWS CLI, execute the following command in the Cloud9 terminal:

```
aws dynamodb create-table --cli-input-json file://~/environment/aws-modern-application-workshop/module-3/aws-cli/dynamodb-table.json
```

After the command runs, you can view the details of your newly created table by executing the following AWS CLI command in the terminal:

```
aws dynamodb describe-table --table-name MysfitsTable
```

If we execute the following command to retrieve all of the items stored in the table, you'll see that the table is empty:

```
aws dynamodb scan --table-name MysfitsTable
```

```
{
    "Count": 0,
    "Items": [],
    "ScannedCount": 0,
    "ConsumedCapacity": null
}
```

#### Add Items to the DynamoDB Table

```
aws dynamodb batch-write-item --request-items file://~/environment/aws-modern-application-workshop/module-3/aws-cli/populate-dynamodb.json
```

Now, if you run the same command to scan all of the table contents, you'll find the items have been loaded into the table:

```
aws dynamodb scan --table-name MysfitsTable
```

### Committing The First *Real* Code change

#### Push the Updated Code into the CI/CD Pipeline

Now, we need to check in these code changes to CodeCommit using the git command line client.  Run the following commands to check in the new code changes and kick of your CI/CD pipeline:

```
cd ~/environment/MythicalMysfitsService-Repository
```

```
git add .
```

```
git commit -m "Add new integration to DynamoDB."
```

```
git push
```

Now, in just 5-10 minutes you'll see your code changes make it through your full CI/CD pipeline in CodePipeline and out to your deployed Flask service to AWS Fargate on Amazon ECS.  

#### Update The Website Content in S3

```
aws s3 cp --recursive ~/environment/aws-modern-application-workshop/module-3/web/ s3://your_bucket_name_here/
```

Re-visit your Mythical Mysfits website to see the new population of Mysfits loading from your DynamoDB table and how the Filter functionality is working!

