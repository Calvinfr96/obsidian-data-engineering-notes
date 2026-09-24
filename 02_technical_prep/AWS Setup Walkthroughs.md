---
aliases:
  - "aws_notes"
tags:
  - de-study
source_notes:
  - "/Users/Apple/dea-workspace/dea-general-learning/AWS/aws_notes.md"
imported: 2026-09-24
---

# AWS Setup Walkthroughs

Review: [[02_technical_prep/AWS Services Review|AWS Services Review]] · [[02_technical_prep/AWS Interview Prep|AWS Interview Prep]]

These are the original course setup walkthroughs. For service behavior and interview tradeoffs, use the linked reviews.

## Contents

- [[#Identity and Access Management (IAM) — setup]]
- [[#Simple Service Storage (S3) — setup]]
- [[#Lambda — setup]]
- [[#Secrets Manager — setup]]
- [[#Simple Notification Service (SNS) — setup]]
- [[#Simple Queue Service (SQS) — setup]]
- [[#Step Functions — setup]]
- [[#Kinesis — setup]]
- [[#API Gateway — setup]]

## Identity and Access Management (IAM) — setup

- IAM Users:
  - When a user is created, it is assigned a user name and password that will be used to sign into the AWS Management Console under a specific account ID. The actions they can perform in the console are determined by the permissions assigned to the user.
  - When a user is created, permissions can be assigned to the user in one of three ways:
    - Permissions can be inherited from a group by assigning the user to a group.
    - Permissions can be copied from an existing user.
    - Permissions can be manually assigned to a user by attaching policies to the user entity.

- IAM Groups:
  - When a group is created, you have the option to add users to the group and attach permissions to the group that all users will inherit. Users and permissions can be modified after creation.

- IAM Roles:
  - When you create a role, you will be asked to select a trusted entity, which will typically be an AWS account or service.
  - Once you select a trusted entity, you will be asked to add permissions by selecting AWS or customer-managed policies.
  - Once permissions are added, you will be asked to specify a name and description for the role and add tags.


## Simple Service Storage (S3) — setup

- S3 Buckets:
  - When you create an S3 bucket, you will first be asked to choose the bucket type, which will either be general purpose or directory. General purpose buckets are used for general-purpose object storage and offer high durability and availability. Directory buckets are used in scenarios that require high-performance, low-latency, and consistent performance requirements, such as data-intensive applications. In most use cases, you will select general purpose.
  - After choosing the bucket type, you will specify the bucket name, which must be _globally_ unique.
  - When choosing settings for your bucket, you will be given the opportunity to copy settings from an existing bucket.
  - Ownership of objects within a bucket fall into two categories:
    - ACLs disabled (recommended): All objects within the bucket are owned by the associated AWS account and access to the bucket and objects within it are controlled by using IAM policies.
    - ACLs enabled: Objects in the bucket can be owned by other AWS accounts and access to the bucket and its objects are controlled using one or more Access Control Lists (ACLs).
  - After specifying ownership settings, you will be asked to specify public access settings. "Block all public access" is selected by default and recommended.
  - After specifying public access settings, you will be asked to enable or disable (default) bucket versioning. Bucket versioning is a feature that assigns a unique version ID to every object modification, allowing for protection against accidental overwrites or deletions by allowing you to retrieve variants of an object using the version ID.
  - After specifying bucket versioning, you will be given the option to add tags and select the type of encryption. The default encryption type is server-side encryption using S3-managed keys.

- S3 Objects:
  - Once a bucket is created, objects can be added to it by manually uploading files to the bucket.
  - Folders can be created within a bucket to categorize objects as needed.
  - Objects and folders can be copied and moved to different s3 destinations as needed. Objects can also be deleted as needed.

## Lambda — setup

- Creating a Lambda Function:
  - When creating a function, you can either start from scratch, using a blueprint provided by AWS, or a container image.
  - After choosing how you want to create your function, you will give it a name, select a runtime programming language, architecture (x86_64 or arm64), and customize permissions by selecting an execution role. You can allow AWS to create default execution role (allowing lambda to upload logs to CloudWatch), select an existing IAM role, or create a new role from AWS policy templates.

## Secrets Manager — setup

- Create and Use Secrets:
  - When you need to connect to an external service or entity, such as Snowflake, in your python code, you either hard code the necessary credentials directly into the code, or retrieve them programmatically from secrets manager. Retrieving credentials from secrets manager is safer because it prevents anyone with access to the code from retrieving the credentials.
  - When you create a secret in secrets manager, you will be asked to choose the type of secret, which can vary from credentials for an AWS service to credentials for a third-party service or database or other types of secrets, such as API keys.
    - The first two types of secrets (AWS or third-party credentials) will require you to specify a username and password.
    - The third type secret (other) requires you to specify key-value pairs.
  - After specifying your credentials or key-value pairs, you can select the type of encryption key you want to use, which will default to an encryption key managed by secrets manager.
  - After specifying the type of encryption key, you will be asked to provide an name and description, as well as add tags if needed. You can also choose to apply a permissions resource policy to the secret and replicate the secret to other AWS regions.
  - Next, you will be asked configure automatic rotation (optional) and provide a rotation schedule, if needed, and specify a lambda rotation function to perform the rotation.


## Simple Notification Service (SNS) — setup

- Creating a Topic:
  -  When you create a topic, you will be asked to provide a name and choose the type. The two types of topics in SNS are first-in-first-out (FIFO) and standard.
    - FIFO topics strictly preserve message ordering, send messages exactly once, support high throughput, and allow users to subscribe from AWS SQS.
    - Standard topics utilize "best-effort" message ordering, send messages at least once, support high throughput, and allow users to subscribe from AWS SQS, AWS Lambda, HTTP, SMS, email, and mobile application endpoints.
  - After specifying the type of topic you want to create, you will be given the option to specify details such as encryption, access policies, data protection policies, delivery policies, and logging.
  - Once a topic is created, subscribers can be added to the topic by creating subscriptions. When creating a subscription, you must specify the type of protocol (SQS, Lambda, HTTP, etc.), the name of the endpoint, as well as optional subscriber filter policies and re-drive policies.
  - When a subscription is created in a topic, it must be confirmed before it can receive messages. AWS automatically sends a link to the subscriber, via the provided endpoint, to perform this confirmation.

- Publishing a Message:
  - Once a topic is created and subscribers are added and confirmed, messages can be sent to subscribers by publishing a message to the topic.
  - When you publish a message, you will be asked to provide an optional subject, an optional TTL, and a message body. You can also specify a message attribute, which consist of a name, value, and type.
  - When a message is successfully published, it is automatically sent to subscribers based on the policies associated with the topic.
  - In order for a publisher, such as S3, to publish messages to an SNS topic, that publisher must be given permission to do so by updating the access policy of the topic accordingly. In S3, you would then create an event notification in an S3 bucket that would be sent to the SNS topic, then to the associated subscribers.


## Simple Queue Service (SQS) — setup

- Creating a Queue:
  - When creating an SQS queue, you will first be asked to select the type (Standard or FIFO) and provide a name.
  - After selecting the type and providing a name, you will be asked to specify configuration details, such as visibility timeout, delivery delay, receive message wait time, message retention period, and maximum message size.
  - After specifying configuration details, you will be asked to select an encryption type. You can choose to enable or disable server-side encryption and you can choose between an SQS encryption key and an AWS Key Management Service encryption key.
  - After specifying encryption details, you will be asked to provide an access policy that determines the entities that can access the queue and what actions they can perform on the queue.
  - After providing an access policy, you will be given the option to specify a re-drive policy (determines which source queues can use this queue as a dead-letter queue) and whether you want to make the queue a dead-letter queue (one that receives undeliverable messages). You will also be given the option to specify tags for the queue.

## Step Functions — setup

- Step Function Example (Order State Machine):
  - The following step function will consist of various lambda functions which perform different tasks in the online ordering process. One lambda function calls a purchase handler, another lambda function calls a refund handler, and the other lambda function calls a result handler.
  - The state machine starts with a choice between "refund" and "purchase", which decides whether the purchase-handler or refund-handler lambda is called. Once either of these lambda functions is completed, they will call the result handler.

- Creating a State Machine:
  - When you create a state machine in step functions, you will be given the choice to choose among various templates or create your own custom state machine.
  - When creating a custom state machine, you will do so in a workflow editor that allows you to add elements to the state machine. Elements can either be actions, flows, or patterns. Actions are AWS services. Flows consist of various types elements, including choice, parallel, map, pass, wait, succeed, and fail.
    - Choice adds branching logic by comparing input data against defined rules (e.g., if/else) to determine which state to transition to next.
    - Parallel executes multiple branches of the workflow simultaneously. The state waits until all branches finish before moving to the next step.
    - Map dynamically iterates over an input array and runs a set of steps for each item. It supports Inline (for smaller datasets) and Distributed (for high-concurrency processing of millions of items) modes.
    - Pass passes its input to its output without performing work. It is often used to transform JSON data or inject fixed data for debugging.
    - Wait pauses the execution for a specific duration or until a specific timestamp is reached.
    - Succeed is a terminal state that stops the execution successfully.
    - Fail is a terminal state that stops the execution and marks it as a failure, allowing you to specify an error name and cause. 
  - After building the workflow in the editor, you will need to provide other details about the state machine, such as the name and permissions. By default, step functions will create an IAM role with all required permissions to execute the various services being used in the workflow. However, you can also choose your own IAM role.


## Kinesis — setup

- Creating and Using a Data Stream:
  - When creating a data stream, you will first be asked to provide a name and choose between on-demand and provisioned data stream capacity.
  - Once a data stream is created, producers can write data to the stream and consumers can read data from the stream.
  - Python scripts can act as a producer when they are written to send data to a data stream.

- Creating and Using Data Firehose:
  - A data firehose allows you to send data ingested by a data stream to another AWS service for storage or analytics.
  - When creating a data firehose, you will be asked to select a source, such as a kinesis data stream and a destination, such as an S3 bucket.
  - After choosing a source and a destination, you will be asked to specify the source settings, such as the specific data stream from which you want to ingest data.
  - After specifying the source settings, you will be given the option to transform source records with AWS Lambda, convert the record format, and decompress source records.
  - After choosing whether you want to transform source records, you will be asked to specify destination settings, such as the specific S3 bucket in which you want to store the ingested, transformed data.
  - Firehose writes data to the source based on the buffer size and interval. The buffer size specifies how much data must be ingested before it is sent to the source (recommended is 5MB) and the interval specifies how often data should be sent to the source. Firehose will send data to the source based on whichever (size or interval) requirement is met first.


## API Gateway — setup

- API Setup:
  - When you create an API, you will first be asked to choose between several API types, including HTTP, Web Socket, and REST APIs.
  - After selecting the API type, you will be asked to provide API details, including whether the API is a new API, import API, or clone of an existing API, as well as the API name, description, and endpoint type (Regional by default).
  - After providing API details, you will be asked to provide resources (paths) that will be apart of the invocation URL. When a resource name will be dynamic, such as the name of an S3 bucket, it should be placed in curly braces (i.e. `{bucket_name}`).
  - After creating resources, you need to specify methods (`GET`, `POST`, `PUT`, `PATCH`, `DELETE`) for those resources. Once the method is chosen, you need to specify which service the API will integrate with. If you select an AWS service as the type of integration, you will need to specify the service, region, HTTP method, action type (action name or path override), and execution role (Allows API gateway to perform the necessary actions, like uploading a file to S3).
    - Path override should be used when the filepath depends on the specific resource (bucket and filename) being acted on.
