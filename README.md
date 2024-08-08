# Automobile ECommerce (Backend)

## Table of Contents
1. Introduction
2. Overview of the functionality
3. Tech Stack
4. Setting up the development environment
5. Implementation Details
6. Build and Deployment

## Introduction <a name="introduction"></a>

The purpose of this project is to demonstrate hands on knowledge about different frontend and backend technologies. Though the functionalities implemented in this project can also be implemented using fully managed services which do not need any coding and require very little configuration, the effort is made to use individual services and make them interact with each other in order to gain hands on experience.

Automobile ECommerce project is about allowing users to browse different kinds of cars, provide information, and filter based on various criteria. In the first phase, only a single page website is designed which will allow users to browse different cars and can filter the search results based on various attributes. In the coming phases additional functionality will be added to make new features available such as user authentication, save favorites and more. The final goal is to mimic a full fleged ecommerce website to make it a one stop shop for automobile buyers.

### Overview of the functionality

The project aims to store car related information and provide mechanism to access that information as and when needed.

This repository is for the backend functionality of an end to end ecommerce project.
For information on frontend functionality, please follow the below URL.

[https://github.com/mangeshb2024/automobile-ecommerce-frontend/edit/main/README.md](https://github.com/mangeshb2024/automobile-ecommerce-frontend/blob/main/README.md)

Various aspectes of the functionality are described below.

#### Storage

The storage layer allows for storing car related information including different attributes such as manufacturer, model, engine, transmission, etc. and static data such as images from diferent angles.

#### Compute

The compute layer allows for handling the reuqests made by clients for accessing car data. The compute layer connects with storage layer for retrieving the required information, enrich the information if needed and pass the response back to clients.

#### API Management

APIs act as a front door for the clients providing a single place entry to backend services that provide access to business logic implemented in compute layer and data stored in storage layer.

## Tech Stack

AWS is used as a cloud service provider for implementing backend services as well as hosting frontend application. Below are the AWS services and technologies used for various layers in the tech stack.

Technology stack can be divided into two categories, frontend (client side) and backend (server side).

Following sections describe backend functionality. For information on frontend functionality, please follow below URL.

[https://github.com/mangeshb2024/automobile-ecommerce-frontend/edit/main/README.md](https://github.com/mangeshb2024/automobile-ecommerce-frontend/blob/main/README.md)

#### Storage layer

**Amazon Dynamodb** - Dynamodb is used a sererless NoSQL key-value data store for storing attributes related to cars as well as the filter criteria. NoSQL database gives flexibility to add or remove attributes dynamically because of the relaxed schema restrictions.

**Amazon S3** - Amazon Simple Storage service is used as an object store for storing the static assets needed for the project such as car images.

#### Compute layer

**AWS Lambda** - AWS Lambda is used as a serverless compute service for serving the requests from clients. Its serverless nature means the service can scale up and down as the traffic from clients increases or decreases. The service receives the requests from clients, fetches the data from Dynamodb table and returns the response to the clients.

#### Interface layer

**Amazon API Gateway** - Amazon API Gateway is used as a REST API management service for exposing the business logic implemented by the Lambda compute layer. API Gatway allows a single point entry for all the backend services. API endpoints exposed by API Gateway are used by client application to invoke the requests to the backend.

#### Other services

**Systems Manager Parameter Store** - For storing dynamodb table names which need to be referenced by Lambda for fetching the data.

**IAM** IAM Roles are used to grant permissions to other services.

### Provisioning and Hosting services

**AWS CloudFormation** - AWS CloudFormation is used as an Infrastructure as Code provisioning tool for deploying backend services. Use of this service avoids the manual efforts and mistakes when the services needed in a stack need to be deployed or updated.

### Version Control

**git and GitHub** - git is used for maintaing versions for the code base on the local system while a repository is created to push the code base to GitHub.

## Setting up the development environment

All the backend services are developed and deployed using AWS Cloud.

Set up an account in AWS. The account provides free tier benefits

For compute layer, AWS Lambda is used. All Lambda functions are written in Python language.

For editing Lambda functions and CloudFormation templates locally, Visual Studio Code is used as code editor.

## Implementation Details

### Storage layer

**Dynamodb**

Dynamodb is used to store car related data as well as filters which will be used by the frontend to select the cars to be displayed based on some criteria. The attributes which need to be stored are as below.

manufacturer
model
vehicletype
bodytype
engine
fuel
transmission
transmissiontype

**DB Table (Car Data): automobileDataTable**

This table will store car related attributes mentioned above. All attributes are of String type. 'manufacturer' is chosen as the partition/hash key for the base table whereas 'model' is used as the sort/range key. Since the combination of partition key and sort key needs to be unique and in case of car data, the combination may not be unique because a particular model from a manufacturer can have different variants, solution used is to append a random unique string at the end of sort key to make it unique.

This table will be populated using a Lambda function (loadCarsData.py) which will be triggered when the source data is uploaded in a source S3 bucket. For more information on Lambda function, refer Compute layer.

For facilitating efficient search, a number of global and local secondary indexes are created to retrieve data based on different filter criteria. If the filter criteria matches any of the index keys, the respective index will be used for fetching the data efficiently, otherwise base table will be queried. The logic to select the index is implemented in Lambda function. For more information on Lambda function, refer Compute layer.

CloudFormation Template Snippet

    dataTable: 
    Type: AWS::DynamoDB::Table
    Properties: 
      AttributeDefinitions: 
        - 
          AttributeName: "manufacturer"
          AttributeType: "S"
        - 
          AttributeName: "vehicletype"
          AttributeType: "S"
        - 
          AttributeName: "bodytype"
          AttributeType: "S"
        - 
          AttributeName: "model"
          AttributeType: "S"
        - 
          AttributeName: "engine"
          AttributeType: "S"
        - 
          AttributeName: "fuel"
          AttributeType: "S"
        - 
          AttributeName: "transmission"
          AttributeType: "S"
      KeySchema: 
        - 
          AttributeName: "manufacturer"
          KeyType: "HASH"
        - 
          AttributeName: "model"
          KeyType: "RANGE"
      ProvisionedThroughput: 
        ReadCapacityUnits: "1"
        WriteCapacityUnits: "1"
      TableName: !Ref dynamoDBTableName
      GlobalSecondaryIndexes: 
        - 
          IndexName: "gsi_bodytype_model"
          KeySchema: 
            - 
              AttributeName: "bodytype"
              KeyType: "HASH"
            -
              AttributeName: "model"
              KeyType: "RANGE"
          Projection: 
            ProjectionType: "ALL"
          ProvisionedThroughput: 
            ReadCapacityUnits: "1"
            WriteCapacityUnits: "1"
        - 
          IndexName: "gsi_bodytype_engine"
          KeySchema: 
            - 
              AttributeName: "bodytype"
              KeyType: "HASH"
            - 
              AttributeName: "engine"
              KeyType: "RANGE"
          Projection: 
            ProjectionType: "ALL"
          ProvisionedThroughput: 
            ReadCapacityUnits: "1"
            WriteCapacityUnits: "1"
        - 
          IndexName: "gsi_bodytype_fuel"
          KeySchema: 
            - 
              AttributeName: "bodytype"
              KeyType: "HASH"
            - 
              AttributeName: "fuel"
              KeyType: "RANGE"
          Projection: 
            ProjectionType: "ALL"
          ProvisionedThroughput: 
            ReadCapacityUnits: "1"
            WriteCapacityUnits: "1"
        - 
          IndexName: "gsi_bodytype_transmission"
          KeySchema: 
            - 
              AttributeName: "bodytype"
              KeyType: "HASH"
            - 
              AttributeName: "transmission"
              KeyType: "RANGE"
          Projection: 
            ProjectionType: "ALL"
          ProvisionedThroughput: 
            ReadCapacityUnits: "1"
            WriteCapacityUnits: "1"
        - 
          IndexName: "gsi_model_engine"
          KeySchema: 
            - 
              AttributeName: "model"
              KeyType: "HASH"
            - 
              AttributeName: "engine"
              KeyType: "RANGE"
          Projection: 
            ProjectionType: "ALL"
          ProvisionedThroughput: 
            ReadCapacityUnits: "1"
            WriteCapacityUnits: "1"
        - 
          IndexName: "gsi_model_fuel"
          KeySchema: 
            - 
              AttributeName: "model"
              KeyType: "HASH"
            - 
              AttributeName: "fuel"
              KeyType: "RANGE"
          Projection: 
            ProjectionType: "ALL"
          ProvisionedThroughput: 
            ReadCapacityUnits: "1"
            WriteCapacityUnits: "1"
        - 
          IndexName: "gsi_model_transmission"
          KeySchema: 
            - 
              AttributeName: "model"
              KeyType: "HASH"
            - 
              AttributeName: "transmission"
              KeyType: "RANGE"
          Projection: 
            ProjectionType: "ALL"
          ProvisionedThroughput: 
            ReadCapacityUnits: "1"
            WriteCapacityUnits: "1"
        - 
          IndexName: "gsi_engine_fuel"
          KeySchema: 
            - 
              AttributeName: "engine"
              KeyType: "HASH"
            - 
              AttributeName: "fuel"
              KeyType: "RANGE"
          Projection: 
            ProjectionType: "ALL"
          ProvisionedThroughput: 
            ReadCapacityUnits: "1"
            WriteCapacityUnits: "1"
        - 
          IndexName: "gsi_engine_transmission"
          KeySchema: 
            - 
              AttributeName: "engine"
              KeyType: "HASH"
            - 
              AttributeName: "transmission"
              KeyType: "RANGE"
          Projection: 
            ProjectionType: "ALL"
          ProvisionedThroughput: 
            ReadCapacityUnits: "1"
            WriteCapacityUnits: "1"
        - 
          IndexName: "gsi_fuel_transmission"
          KeySchema: 
            - 
              AttributeName: "fuel"
              KeyType: "HASH"
            - 
              AttributeName: "transmission"
              KeyType: "RANGE"
          Projection: 
            ProjectionType: "ALL"
          ProvisionedThroughput: 
            ReadCapacityUnits: "1"
            WriteCapacityUnits: "1"
        - 
          IndexName: "gsi_transmission"
          KeySchema: 
            - 
              AttributeName: "transmission"
              KeyType: "HASH"
          Projection: 
            ProjectionType: "ALL"
          ProvisionedThroughput: 
            ReadCapacityUnits: "1"
            WriteCapacityUnits: "1"
      LocalSecondaryIndexes: 
        - 
          IndexName: "lsi_bodytype"
          KeySchema: 
            - 
              AttributeName: "manufacturer"
              KeyType: "HASH"
            - 
              AttributeName: "bodytype"
              KeyType: "RANGE"
          Projection:
            ProjectionType: "ALL"
        - 
          IndexName: "lsi_vehicletype"
          KeySchema: 
            - 
              AttributeName: "manufacturer"
              KeyType: "HASH"
            - 
              AttributeName: "vehicletype"
              KeyType: "RANGE"
          Projection:
            ProjectionType: "ALL"
        - 
          IndexName: "lsi_engine"
          KeySchema: 
            - 
              AttributeName: "manufacturer"
              KeyType: "HASH"
            - 
              AttributeName: "engine"
              KeyType: "RANGE"
          Projection:
            ProjectionType: "ALL"
        - 
          IndexName: "lsi_fuel"
          KeySchema: 
            - 
              AttributeName: "manufacturer"
              KeyType: "HASH"
            - 
              AttributeName: "fuel"
              KeyType: "RANGE"
          Projection:
            ProjectionType: "ALL"
        - 
          IndexName: "lsi_transmission"
          KeySchema: 
            - 
              AttributeName: "manufacturer"
              KeyType: "HASH"
            - 
              AttributeName: "transmission"
              KeyType: "RANGE"
          Projection:
            ProjectionType: "ALL"

**DB Table (Filter Data): automobileFilterDataTable**

This table will store the filter criteria, which will be requested by frontend for rendering in UI giving filter options to users. There will be one item for each column attribute from the table "automobileDataTable" consisting only the unique values for a given attribute. This table will also be populated by the same Lambda function (loadCarsData.py) which populates the cars data table "automobileDataTable".

The will have only one key attribute (partition key) filterColumn which will map to different attributes in the data table automobileDataTable. Each item in this table will have another attribute which will contain the unique values for each column attribute in the table automobileDataTable.

CloudFormation Template Snippet

    filterTable: 
    Type: AWS::DynamoDB::Table
    Properties: 
      AttributeDefinitions: 
        - 
          AttributeName: "filterColumn"
          AttributeType: "S"
      KeySchema: 
        - 
          AttributeName: "filterColumn"
          KeyType: "HASH"
      ProvisionedThroughput: 
        ReadCapacityUnits: "1"
        WriteCapacityUnits: "1"
      TableName: !Ref dynamoDBFilterTableName

**S3**

**automobile-ecommerce-source-data**

This S3 bucket will be used to load the car related attributes data eventually into Dynamodb. When the input file is loaded into this S3 bucket, an event will be generated which will trigger the Lambda function "loadCarsData.py" to read the file and load the data into Dynamodb tables. The below CloudFormation snippet shows how the Lambda is triggered as part of notification configuration on S3 bucket.

    dataSourceS3Bucket:
    Type: AWS::S3::Bucket
    DependsOn: S3LambdaInvokePermission
    Properties:
      BucketName: !Ref sourceDataS3Bucket
      NotificationConfiguration:
        LambdaConfigurations:
          - Event: s3:ObjectCreated:Put
            Function: !GetAtt [ loadDataFunction, Arn]

For triggering the Lambda function, S3 will need the required permissions. These will be provided by below CloudFormation snippet.

    S3LambdaInvokePermission:
    Type: AWS::Lambda::Permission
    DependsOn: loadDataFunction
    Properties:
      FunctionName:
        Fn::GetAtt:
          - loadDataFunction
          - Arn
      Action: lambda:InvokeFunction
      Principal: s3.amazonaws.com
      SourceArn:
        Fn::Sub: arn:aws:s3:::${sourceDataS3Bucket}

**automobile-ecommerce-static-data**

This S3 bucket will be used to store the static data i.e. images of the cars. When the request is made from the frontend, the Lambda function "loadCarsData.py" fetches the data from Dynamodb table and the response is enriched with reference to static data images stored in this S3 bucket. When the data is rendered in UI, the frontend will retrieve the images from this S3 bucket.

Currently, this bucket is manually created and static data is manually loaded before the application is deployed. This will be automated in near future.

### Compute layer

**Lambda**

AWS Lambda fits the requirement for this project from computing layer prespective as the business logic needs to be executed only in response to an event triggered from frontend. So no need of long running computing infrastructure such as EC2. Also, since Lambda is a serverless service, there is no need to manage underlying infrastructure as well as automatic scaling handles scaling up and scaling down in response to the traffic.

**loadCarsData.py**

This function is used for loading the cars data received in a csv file into the Dynamodb table "automobileDataTable" as well "automobileFilterDataTable".

The function is triggered in response to an S3 event i.e. when an input CSV file containing the cars data is uploaded into an S3 bucket, an event will be generated which will trigger this Lambda function. This function will then read the file from S3, parse its contents and load the data into Dynamodb table "automobileDataTable". The function will also check if the record being loaded already exists in the table or not. If it exists already, then the record is skipped. 

While loading the data in "automobileDataTable", the function also loads the unique values for each attribute from "automobileDataTable" into "automobileFilterDataTable".

The function needs the permissions to access S3 bucket and Dynamodb tables. These permissions are provided by IAM Role which Lambda assumes to access these services.

    loadDataLambdaExecutionRole:
        Type: AWS::IAM::Role
        Properties:
        AssumeRolePolicyDocument: 
            Version: "2012-10-17"
            Statement:
            - Effect: Allow
                Principal:
                Service:
                    - lambda.amazonaws.com
                Action:
                - 'sts:AssumeRole'
        Description: IAM Execution Role for loadData Lambda function
        ManagedPolicyArns: 
            - arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
            - arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
            - arn:aws:iam::aws:policy/AmazonDynamoDBFullAccess
            - arn:aws:iam::aws:policy/AmazonSSMReadOnlyAccess
        RoleName: loadDataLambdaExecRole

**fetchCarsData.py**

This function will be used for fetching the car data which is stored in Dynamodb table "automobileDataTable". The function will receive input as "multiValueQueryStringParameters". Based on the parameters received, either query or scan operation will be performed on Dynamodb table.

The input parameters will be analysed and if any index (global or local) is found to be fitting the input criteria, then the query operation will be used. The parameters that fall outside the index keys (partition and sort) will be pushed into filter expression of the query operation. If none of the indexes fit the input parameters, then scan operation will be used. All the parameters will then be pushed into filter expression of the scan operation.

Once the data is retrieved from dynamodb table, the response will be enriched with image data before returning to the client. For each record retrieved from the table, S3 bucket will be looked up for any images corresponding to the car model. If found, the object URLs will be added to the record and the response is returned to the client.

The function will need permissions to access dynamodb table as well as S3 bucket. This will be achived by assuming an IAM role which will grant the required permissions to the function.

    fetchDataLambdaExecutionRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument: 
        Version: "2012-10-17"
        Statement:
          - Effect: Allow
            Principal:
              Service:
                - lambda.amazonaws.com
            Action:
              - 'sts:AssumeRole'
      Description: IAM Execution Role for fetchCarsData Lambda function
      ManagedPolicyArns: 
        - arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
        - arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
        - arn:aws:iam::aws:policy/AmazonDynamoDBReadOnlyAccess
        - arn:aws:iam::aws:policy/AmazonSSMReadOnlyAccess
      RoleName: fetchDataLambdaExecRole

**FetchFilters.py**

This function is similar to FetchCarsData function but is simpler in terms of implementation. All the data in the table automobileFilterDataTable is retrieved and returned to the client. The function only needs permissions to access Dynamodb table. This is achieved by assuming an IAM role which has the required permissions.

    fetchFiltersLambdaExecutionRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument: 
        Version: "2012-10-17"
        Statement:
          - Effect: Allow
            Principal:
              Service:
                - lambda.amazonaws.com
            Action:
              - 'sts:AssumeRole'
      Description: IAM Execution Role for fetchFilters Lambda function
      ManagedPolicyArns: 
        - arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
        - arn:aws:iam::aws:policy/AmazonDynamoDBReadOnlyAccess
        - arn:aws:iam::aws:policy/AmazonSSMReadOnlyAccess
      RoleName: fetchFiltersLambdaExecRole

### Interface layer

**API Gateway**

API Gateway acts as a front door to applications to access data, business logic and functionality from the backend services. Any request for accessing backend services goes via API Gateway. The REST APIs are created which expose endpoints for various resources and methods to perform action on those resources.

**automobileAPI**

This API is created with regional endpoint to provide access to resources maintained in the backend such as cars and filters data. 

    apiGateway:
    Type: AWS::ApiGateway::RestApi
    Properties:
      Description: API to fetch data from backend DynamodB table
      EndpointConfiguration: 
        Types:
          - REGIONAL
      Name: !Ref apiGatewayName

**Resources**

Below resources are created under the API.

**Cars Resource**

    apiGatewayCarsResource:
    Type: AWS::ApiGateway::Resource
    Properties:
      RestApiId: !Ref apiGateway
      ParentId: !GetAtt
        - apiGateway
        - RootResourceId
      PathPart: cars

**Filters Resource**

    apiGatewayFiltersResource:
    Type: AWS::ApiGateway::Resource
    Properties:
      RestApiId: !Ref apiGateway
      ParentId: !GetAtt
        - apiGateway
        - RootResourceId
      PathPart: filters

**Methods**

Below methods are created for each resource for accessing the data.

**GET Method for cars Resource**

This method is created to fetch the cars data from Dynamodb table automobileDataTable. This method is integrated with fetchCarsData Lambda function using proxy integration type. The request received by the API Gateway will be passed as it is to the Lambda function without any processing.

    apiGatewayCarsGETMethod:
    Type: AWS::ApiGateway::Method
    Properties:
      AuthorizationType: NONE
      HttpMethod: GET
      Integration:
        IntegrationHttpMethod: POST
        Type: AWS_PROXY
        Uri: !Sub
          - arn:aws:apigateway:${AWS::Region}:lambda:path/2015-03-31/functions/${lambdaArn}/invocations
          - lambdaArn: !GetAtt fetchDataFunction.Arn
      ResourceId: !Ref apiGatewayCarsResource
      RestApiId: !Ref apiGateway

**GET Method for filters Resource**

This method is created to fetch filters data from Dynamodb table automobileFilterDataTable. This is again a proxy integration with fetchFilters Lambda function.

    apiGatewayFiltersGETMethod:
    Type: AWS::ApiGateway::Method
    Properties:
      AuthorizationType: NONE
      HttpMethod: GET
      Integration:
        IntegrationHttpMethod: POST
        Type: AWS_PROXY
        Uri: !Sub
          - arn:aws:apigateway:${AWS::Region}:lambda:path/2015-03-31/functions/${lambdaArn}/invocations
          - lambdaArn: !GetAtt fetchFiltersFunction.Arn
      ResourceId: !Ref apiGatewayFiltersResource
      RestApiId: !Ref apiGateway

**Lambda Invoke Permissions**


API Gateway will need permission to invoke the Lambda function as part of proxy integration. These are provided by below CloudFormation snippets. This creates resource policy on Lambda functions and allows access to API Gateway service.

**fetchCarsData invoke permission**

    apiGatewayFetchDataLambdaInvokePermission:
        Type: AWS::Lambda::Permission
        Properties:
        Action: lambda:InvokeFunction
        FunctionName: !GetAtt fetchDataFunction.Arn
        Principal: apigateway.amazonaws.com
        SourceArn: !Sub arn:aws:execute-api:${AWS::Region}:${AWS::AccountId}:${apiGateway}/${apiGatewayStageName}/GET/cars

**fetchFilters invoke permission**

    apiGatewayFetchFiltersLambdaInvokePermission:
    Type: AWS::Lambda::Permission
    Properties:
      Action: lambda:InvokeFunction
      FunctionName: !GetAtt fetchFiltersFunction.Arn
      Principal: apigateway.amazonaws.com
      SourceArn: !Sub arn:aws:execute-api:${AWS::Region}:${AWS::AccountId}:${apiGateway}/${apiGatewayStageName}/GET/filters

**Deployment**

APIs can not be accessed unless they are deployed into stage. This is achieved by below CloudFormation snippet where deployment stage prod will be created.

    apiGatewayDeployment:
    Type: AWS::ApiGateway::Deployment
    DependsOn:
      - apiGatewayCarsGETMethod
      - apiGatewayFiltersGETMethod
    Properties:
      RestApiId: !Ref apiGateway
      StageName: !Ref apiGatewayStageName

### Provisioning and Deployment

#### CloudFormation, CodePipeline

### Version Control

**git and GitHub** - git is used for maintaing versions for the code base on the local system while a repository is created to push the code base to GitHub.

### Set up the project on local machine

Create a directory on local system call 'automobile-ecommerce-backend'.
Create Lambda function python files and CloudFormation yml templates locally in the above folder and edit using Visual Studio Code editor.
Once created, set up the git repository as per below instructions

### Setup the git repository

Signup or Login to GitHub (https://github.com/) and create a new repository for the project.

Download and install git on your local machine. (https://git-scm.com/download)

Initialize local git repository by executing below commands.

    cd automobile-ecommerce-backend
    echo "# automobile-ecommerce-backend" >> README.md
    git init
    git add .

If you get a permission error for .config directory while executing git add command, then it can be resolved by granting below permission in sudo mode.

    sudo chown -R $(whoami) /Users/<user>/.config

Once the error is resolved, proceed with remaining commands.

    git commit -m "first commit"
    git branch -M main
    git remote add origin https://github.com/<github account name>/automobile-ecommerce-backend.git
    git push -u origin main

## Build, Provisioning, and Deployment

### Build

Backend services such as Dynamodb, AWS Lambda and API Gateway do not need building as such. These services can be created using management console and can be unit tested. Once tested successfully, these can be provisioned using CloudFormation.

### Provisioning

CloudFormation is used for provisioning of resources. The template 'Infrastructure.yaml' will be used by CloudFormation service to provision all the resources explained in the implementation section above.

### Deployment

The provisioining and deployment process is further automated by using CodePipeline. The flow of operations is as below.

A CodePipeline is created for deployment of CloudFormation template as well as loading of initial data into the DynamoDB tables as well static data into S3 Bucket.

**Source**

For Source provider, select GitHub (Version 2) 

Connect to this GitHub account by creating a new connection.

For repository name, select automobile-ecommerce-backend.

For Default branch, select main.

For Trigger, select below values.

Trigger type: Specify filter
Event type: Push
Filter type: Branch
Branches:
    Include: main
File paths (optional):
    Include: Infrastructure.yaml

**Build**

For Build, select Build provider as AWS CodeBuild

Create a project with default settings, except that for Buildspec select "Use a buildspec file".

buildspec.yml



