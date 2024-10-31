# LevelUp! Lab for Serverless

## Lab Overview And High-Level Design

Let's start with the High-Level Design.

![Serverless Micro Servie Design](https://github.com/user-attachments/assets/f17f8d0e-3fea-46f8-856f-77afbdcad36b)

An Amazon API Gateway is a collection of resources and methods. For this tutorial, you create one resource (DynamoDBManager) and define one method (POST) on it. The method is backed by a Lambda function (LambdaFunctionOverHttps). That is when you call the API through an HTTPS endpoint, Amazon API Gateway invokes the Lambda function.

The POST method on the DynamoDBManager resource supports the following DynamoDB operations:

Create, update, and delete an item.
Read an item.
Scan an item.
Other operations (echo, ping), not related to DynamoDB, that you can use for testing.
The request payload you send in the POST request identifies the DynamoDB operation and provides the necessary data. For example:

The following is a sample request payload for a DynamoDB create item operation:
