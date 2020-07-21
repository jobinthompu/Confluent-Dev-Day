# Seamlessly streaming data from Confluent Cloud into Amazon Redshift

## Short Description:

There has been tremendous adoption of Apache Kafka throughout the years, and increasingly developers are using Kafka as the foundation for their event streaming applications. For this reason, it is important for developers to have access to a fully managed Apache Kafka service that frees them from operational complexities, so they don’t need to be pros in order to use the technology. 

This is where Confluent Cloud comes in. Built as a cloud-native service, Confluent Cloud offers developers a serverless experience with elastic scaling and pricing that charges only for what you stream.

In this post, we provide an overview of Confluent Cloud and step-by-step instructions to show you how to seamlessly stream data from Confluent Cloud Kafka into a Redshift Table.

## Kafka Connect – Confluent Cloud Redshift Connector

The Kafka Connect Amazon Redshift Sink connector allows you to stream data from Apache Kafka® topics to Amazon Redshift. The connector polls data from Kafka and writes this data to an Amazon Redshift database. Polling data is based on subscribed topics. Auto-creation of tables and limited auto-evolution are supported. 

In this workshop you'll learn how to create a Kafka cluster in Confluent Cloud Data, stream data from a Lambda Streaming function and ingest data into Redshift via Connector provided by Confluent Cloud . 

![alt tag](https://github.com/jobinthompu/Confluent-Dev-Day/blob/master/Resources/images/Arch.jpg)

	Fig 1: Above Architecture diagram shows how data is streamed from Confluent cloud to Redshift 

## Prerequisites
-	This Blog assumes you already have an active Confluent Cloud account. If not sign-up here: Sign-Up for a Confluent Cloud Account

-	This Blog assumes you already have an Active AWS account. If not Create and activate a new AWS account.

## Creating AWS Resources required for Redshift Streaming Module

Below CloudFormation Script will deploy: 

- 	A new VPC 
-	A Redshift Cluster, configure it to be publicly accessible, 
-	A Lambda Function which will simulate streaming data, convert JSON records read from an s3 bucket to Avro format and publish into Kafka Cluster running on Confluent Cloud 

Click on below button to launch the CloudFormation template which will spin-up required resources for this workshop:

[!Launch Stack (https://github.com/jobinthompu/Confluent-Dev-Day/blob/master/Resources/images/LaunchStack.png)](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/create/review?stackName=Confluent-Redshift-Connector&templateURL=https://cloudformation-template-repo.s3.amazonaws.com/ConfluentRedshift.json)

On the Quick Create Stack page, acknowledge the resource creations and click ‘Create Stack’. It may take a few minutes to complete the stack creation. Stack name is already populated as ‘Confluent-Redshift-Connector’

Once created navigate to the Stack Output tab and note down Redshift End Point and Lambda Function Name. We will require it for future steps. 

![alt tag](https://github.com/jobinthompu/Confluent-Dev-Day/blob/master/Resources/images/CFT-Output.png)

## Creating Confluent Cloud Resources

### Creating a Kafka Cluster

Now let’s Launch Confluent Cloud Console from another tab on the browser: https://confluent.cloud/login

Once logged in, click on the Create Cluster Icon on the screen to create a new Kafka Cluster. If you already have a cluster, you can create a new topic to complete the exercise.

