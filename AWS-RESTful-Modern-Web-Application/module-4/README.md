# Module 4: Adding User and API features with Amazon API Gateway and AWS Cognito

![Architecture](/images/module-4/architecture-module-4.png)

### Overview

** We need to first have users register on the website.  To enable registration and authentication of website users, we will create a [**User Pool**](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-identity-pools.html) in [**AWS Cognito**](http://aws.amazon.com/cognito/) 
** we will deploy an REST API with [**Amazon API Gateway**](https://aws.amazon.com/api-gateway/) to sit in front of our NLB. 
### Adding a User Pool for Website Users

#### Create the Cognito User Pool

```
aws cognito-idp create-user-pool --pool-name MysfitsUserPool --auto-verified-attributes email
```

#### Create a Cognito User Pool Client

```
aws cognito-idp create-user-pool-client --user-pool-id REPLACE_ME --client-name MysfitsUserPoolClient
```

### Adding a new REST API with Amazon API Gateway

#### Create an API Gateway VPC Link

```
aws apigateway create-vpc-link --name MysfitsApiVpcLink --target-arns REPLACE_ME_NLB_ARN > ~/environment/api-gateway-link-output.json
```

```
{
    "status": "PENDING",
    "targetArns": [
        "YOUR_ARN_HERE"
    ],
    "id": "abcdef1",
    "name": "MysfitsApiVpcLink"
}
```

With the VPC link creating, we can move on to create the actual REST API using Amazon API Gateway.  

#### Create the REST API using Swagger

Your MythicalMysfits REST API is defined using **Swagger**, a popular open-source framework for describing APIs via JSON.  

```
aws apigateway import-rest-api --parameters endpointConfigurationTypes=REGIONAL --body file://~/environment/aws-modern-application-workshop/module-4/aws-cli/api-swagger.json --fail-on-warnings
```

```
{
    "name": "MysfitsApi",
    "endpointConfiguration": {
        "types": [
            "REGIONAL"
        ]
    },
    "id": "abcde12345",
    "createdDate": 1529613528
}
```

#### Deploy the API

Now, our API has been created, but it's yet to be deployed anywhere. 

```
aws apigateway create-deployment --rest-api-id REPLACE_ME_WITH_API_ID --stage-name prod
```
```
https://REPLACE_ME_WITH_API_ID.execute-api.REPLACE_ME_WITH_REGION.amazonaws.com/prod
```
