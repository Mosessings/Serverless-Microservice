# Lab for Serverless

## Lab Overview And High-Level Design

Let's start with the High-Level Design.

![Serverless Micro Servie Design](https://github.com/user-attachments/assets/f17f8d0e-3fea-46f8-856f-77afbdcad36b)

An Amazon API Gateway is a collection of resources and methods. For this tutorial, you create one resource (DynamoDBManager) and define one method (POST) on it. The method is backed by a Lambda function (LambdaFunctionOverHttps). That is when you call the API through an HTTPS endpoint, Amazon API Gateway invokes the Lambda function.

The POST method on the DynamoDBManager resource supports the following DynamoDB operations:

* Create, update, and delete an item.
* Read an item.
* Scan an item.
* Other operations (echo, ping), not related to DynamoDB, that you can use for testing.
  
The request payload you send in the POST request identifies the DynamoDB operation and provides the necessary data. For example:

The following is a sample request payload for a DynamoDB create item operation:
```json
{
    "operation": "create",
    "tableName": "lambda-apigateway",
    "payload": {
        "Item": {
            "id": "1",
            "name": "Bob"
        }
    }
}
```
The following is a sample request payload for a DynamoDB read item operation:
```json
{
    "operation": "read",
    "tableName": "lambda-apigateway",
    "payload": {
        "Key": {
            "id": "1"
        }
    }
}
```

## Setup

### Create Lambda IAM Role 
Create the execution role that gives your function permission to access AWS resources.

To create an execution role

1. Open the roles page in the IAM console.
2. Choose Create role.
3. Create a role with the following properties.
    * Trusted entity – Lambda.
    * Role name – **lambda-apigateway-role**.
    * Permissions – Custom policy with permission to DynamoDB and CloudWatch Logs. This custom policy has the permissions that  the function needs to write data to DynamoDB and upload logs.
    ```json
    {
    "Version": "2012-10-17",
    "Statement": [
    {
      "Sid": "Stmt1428341300017",
      "Action": [
        "dynamodb:DeleteItem",
        "dynamodb:GetItem",
        "dynamodb:PutItem",
        "dynamodb:Query",
        "dynamodb:Scan",
        "dynamodb:UpdateItem"
      ],
      "Effect": "Allow",
      "Resource": "*"
    },
    {
      "Sid": "",
      "Resource": "*",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Effect": "Allow"
    }
    ]
    }
    ```
### Create Lambda Function

**To create the function**
1. Click "Create function" in AWS Lambda Console

![Create function](https://github.com/user-attachments/assets/6b0b1aba-6c52-4ff7-98d6-cf35c73652fd)

2. Select "Author from scratch". Use name **LambdaFunctionOverHttps** , select **Python 3.7** as Runtime. Under Permissions, select "Use an existing role", and select **lambda-apigateway-role** that we created, from the drop down

3. Click "Create function"

![create ff](https://github.com/user-attachments/assets/f0121d57-7296-4aca-be67-6e1980e5f910)

4. Replace the boilerplate coding with the following code snippet and click "Save"

**Example Python Code**
```python
from __future__ import print_function

import boto3
import json

print('Loading function')


def lambda_handler(event, context):
    '''Provide an event that contains the following keys:

      - operation: one of the operations in the operations dict below
      - tableName: required for operations that interact with DynamoDB
      - payload: a parameter to pass to the operation being performed
    '''
    #print("Received event: " + json.dumps(event, indent=2))

    operation = event['operation']

    if 'tableName' in event:
        dynamo = boto3.resource('dynamodb').Table(event['tableName'])

    operations = {
        'create': lambda x: dynamo.put_item(**x),
        'read': lambda x: dynamo.get_item(**x),
        'update': lambda x: dynamo.update_item(**x),
        'delete': lambda x: dynamo.delete_item(**x),
        'list': lambda x: dynamo.scan(**x),
        'echo': lambda x: x,
        'ping': lambda x: 'pong'
    }

    if operation in operations:
        return operations[operation](event.get('payload'))
    else:
        raise ValueError('Unrecognized operation "{}"'.format(operation))
```

![LFOH](https://github.com/user-attachments/assets/0865a921-63b4-4cf4-ba24-e5063bd48e94)

### Test Lambda Function

Let's test our newly created function. We haven't created DynamoDB and the API yet, so we'll do a sample echo operation. The function should output whatever input we pass.
1. Click the arrow on "Select a test event" and click "Configure test events"

![LFOH2](https://github.com/user-attachments/assets/5d0c9bd3-765b-4351-9dce-cfe238c2074c)

2. Paste the following JSON into the event. The field "operation" dictates what the lambda function will perform. In this case, it'd simply return the payload from input event as output. Click "Create" to save
```json
{
    "operation": "echo",
    "payload": {
        "somekey1": "somevalue1",
        "somekey2": "somevalue2"
    }
}
```

![LFOH3](https://github.com/user-attachments/assets/9bff5130-9952-4f74-b2ed-467757eb8dd8)

3. Click "Test", and it will execute the test event. You should see the output in the console

![ttt](https://github.com/user-attachments/assets/6e8ebd76-7b74-4789-a136-00acb48bdab2)

We're all set to create DynamoDB table and an API using our lambda as backend!

### Create DynamoDB Table

Create the DynamoDB table that the Lambda function uses.

**To create a DynamoDB table**

1. Open the DynamoDB console.
2. Choose Create table.
3. Create a table with the following settings.
   * Table name – lambda-apigateway
   * Primary key – id (string)
4. Choose Create.

![dddd](https://github.com/user-attachments/assets/cdb05994-9f8d-4a48-96ef-16f1110d7b83)

### Create API

**To create the API**
1. Go to API Gateway console
2. Click Create API

![22222](https://github.com/user-attachments/assets/f71428ed-d51f-4f83-b147-5643da8a97d2)

3. Scroll down and select "Build" for REST API

![222222](https://github.com/user-attachments/assets/4e20ac10-f8af-4d4e-811f-bf904263698d)

4. Give the API name as "DynamoDBOperations", keep everything as is, click "Create API"

![2222222](https://github.com/user-attachments/assets/b32c562f-8f84-404d-950c-bea52bc34fa1)

5. Each API is collection of resources and methods that are integrated with backend HTTP endpoints, Lambda functions, or other AWS services. Typically, API resources are organized in a resource tree according to the application logic. At this time you only have the root resource, but let's add a resource next 

Click "Actions", then click "Create Resource"

![create resources](https://github.com/user-attachments/assets/66f959d7-33dc-4605-a85f-9bee5d51ae5a)

6. Input "DynamoDBManager" in the Resource Name, Resource Path will get populated. Click "Create Resource"

![create resources 2](https://github.com/user-attachments/assets/6892cd58-707f-4be2-8da7-08c019fd7abd)

7. Let's create a POST Method for our API. With the "/dynamodbmanager" resource selected, Click "Actions" again and click "Create Method".

![Create resources 3](https://github.com/user-attachments/assets/c7ee8701-4b09-4d76-81c7-51cec818a18a)

8. Select "POST" from drop down , then click checkmark

![create resources 4](https://github.com/user-attachments/assets/af96b011-b609-44c1-a4be-31c7f037622f)

9. The integration will come up automatically with "Lambda Function" option selected. Select "LambdaFunctionOverHttps" function that we created earlier. As you start typing the name, your function name will show up.Select and click "Save". A popup window will come up to add resource policy to the lambda to be invoked by this API. Click "Ok"

![create resources 5](https://github.com/user-attachments/assets/ed4dc0ed-f70b-4e11-8465-32d5d0269eac)

Our API-Lambda integration is done!

### Deploy the API

In this step, you deploy the API that you created to a stage called prod.

1. Click "Actions", select "Deploy API"

![0](https://github.com/user-attachments/assets/0c07a03d-8c4e-4dce-945b-a7ec5273c1c1)

2. Now it is going to ask you about a stage. Select "[New Stage]" for "Deployment stage". Give "Prod" as "Stage name". Click "Deploy"

![00](https://github.com/user-attachments/assets/d3500995-d2f2-427a-a4c3-44601a543783)

3. We're all set to run our solution! To invoke our API endpoint, we need the endpoint url. In the "Stages" screen, expand the stage "Prod", select "POST" method, and copy the "Invoke URL" from screen

![000](https://github.com/user-attachments/assets/93e7b4e0-6c62-4da4-8a86-fe7b7f253434)

### Running our solution

1. The Lambda function supports using the create operation to create an item in your DynamoDB table. To request this operation, use the following JSON:

```json
{
    "operation": "create",
    "tableName": "lambda-apigateway",
    "payload": {
        "Item": {
            "id": "1234ABCD",
            "number": 5
        }
    }
}
```
2. To execute our API from local machine, we are going to use Postman and Curl command. You can choose either method based on your convenience and familiarity.
   
    * To run this from Postman, select "POST" , paste the API invoke url. Then under "Body" select "raw" and paste the above JSON. Click "Send". API should execute and return "HTTPStatusCode" 200.
  
![uu](https://github.com/user-attachments/assets/5aa2d11b-4e28-4c0b-a9fa-8749bb62ba9e)

   * To run this from the terminal using Curl, run the below

    ```
    $ curl -X POST -d "{\"operation\":\"create\",\"tableName\":\"lambda-apigateway\",\"payload\":{\"Item\":{\"id\":\"1\",\"name\":\"Bob\"}}}" https://$API.execute-api.$REGION.amazonaws.com/prod/DynamoDBManager
    ```   
3. To validate that the item is indeed inserted into DynamoDB table, go to Dynamo console, select "lambda-apigateway" table, select "Items" tab, and the newly inserted item should be displayed.

![z](https://github.com/user-attachments/assets/22d86f05-8fa8-4aed-98b1-1fe9bdaf45b8)

4. To get all the inserted items from the table, we can use the "list" operation of Lambda using the same API. Pass the following JSON to the API, and it will return all the items from the Dynamo table

```json
{
    "operation": "list",
    "tableName": "lambda-apigateway",
    "payload": {
    }
}
```

![zz](https://github.com/user-attachments/assets/a41239f9-5e0b-4ec6-92eb-151098af6cd9)

We have successfully created a serverless API using API Gateway, Lambda, and DynamoDB!

## Cleanup

Let's clean up the resources we have created for this lab.


### Cleaning up DynamoDB

To delete the table, from DynamoDB console, select the table "lambda-apigateway", and click "Delete table"

![v](https://github.com/user-attachments/assets/b49ed195-f0cd-48b9-81c0-d9e8c27bb583)

To delete the Lambda, from the Lambda console, select lambda "LambdaFunctionOverHttps", click "Actions", then click Delete 

![vv](https://github.com/user-attachments/assets/4fc371a4-0427-41a5-bdab-a0cc47638241)

To delete the API we created, in API gateway console, under APIs, select "DynamoDBOperations" API, click "Actions", then "Delete"

![vvv](https://github.com/user-attachments/assets/caa55575-3cb3-4dda-a53d-0301a73da821)
