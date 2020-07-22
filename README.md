# Seamlessly streaming data from Confluent Cloud into Amazon Redshift

An Overview of this excersise is below:

[Short Introduction](https://github.com/jobinthompu/Confluent-Dev-Day#short-introduction)\
[Kafka Connect – Confluent Cloud Redshift Connector](https://github.com/jobinthompu/Confluent-Dev-Day#kafka-connect--confluent-cloud-redshift-connector)\
[Prerequisites](https://github.com/jobinthompu/Confluent-Dev-Day#prerequisites)\
[Creating AWS Resources required for Redshift Streaming Module](https://github.com/jobinthompu/Confluent-Dev-Day#creating-aws-resources-required-for-redshift-streaming-module)\
[Creating Confluent Cloud Resources](https://github.com/jobinthompu/Confluent-Dev-Day#creating-confluent-cloud-resources)
- [Creating a Kafka Cluster](https://github.com/jobinthompu/Confluent-Dev-Day#creating-a-kafka-cluster)
- [Creating a Kafka Topic](https://github.com/jobinthompu/Confluent-Dev-Day#creating-a-kafka-topic)
- [Creating Kafka API Keys](https://github.com/jobinthompu/Confluent-Dev-Day#creating-kafka-api-keys)
- [Enabling Schema Registry for Confluent cloud and Create API Keys](https://github.com/jobinthompu/Confluent-Dev-Day#enabling-schema-registry-for-confluent-cloud-and-create-api-keys)
- [Configuring Confluent Cloud Redshift Sink Connector](https://github.com/jobinthompu/Confluent-Dev-Day#configuring-confluent-cloud-redshift-sink-connector)
- [Launching Lambda Streaming function to ingest data into Kafka Topic](https://github.com/jobinthompu/Confluent-Dev-Day#launching-lambda-streaming-function-to-ingest-data-into-kafka-topic)
- [Preview Data Ingested into Redshift table from Query Editor console](https://github.com/jobinthompu/Confluent-Dev-Day#preview-data-ingested-into-redshift-table-from-console)

[Clean Up](https://github.com/jobinthompu/Confluent-Dev-Day#clean-up)\
[Conclusion](https://github.com/jobinthompu/Confluent-Dev-Day#conclusion)

 
## Short Introduction:

There has been tremendous adoption of Apache Kafka throughout the years, and increasingly developers are using Kafka as the foundation for their event streaming applications. For this reason, it is important for developers to have access to a fully managed Apache Kafka service that frees them from operational complexities, so they don’t need to be pros in order to use the technology. 

This is where [Confluent Cloud](https://www.confluent.io/confluent-cloud) comes in. Built as a cloud-native service, Confluent Cloud offers developers a serverless experience with elastic scaling and pricing that charges only for what you stream.

In this post, we provide an overview of Confluent Cloud and step-by-step instructions to show you how to seamlessly stream data from Confluent Cloud Kafka into a Redshift Table.

## Kafka Connect – Confluent Cloud Redshift Connector

The Kafka Connect Amazon Redshift Sink connector allows you to stream data from Apache Kafka® topics to Amazon Redshift. The connector polls data from Kafka and writes this data to an Amazon Redshift database. Polling data is based on subscribed topics. Auto-creation of tables and limited auto-evolution are supported. 

In this workshop you'll learn how to create a Kafka cluster in Confluent Cloud Data, stream data from a Lambda Streaming function and ingest data into Redshift via Connector provided by Confluent Cloud . 

![alt tag](https://github.com/jobinthompu/Confluent-Dev-Day/blob/master/Resources/images/Arch.jpg)

	Fig 1: Above Architecture diagram shows how data is streamed from Confluent cloud to Redshift 

## Prerequisites
-	This Blog assumes you already have an active Confluent Cloud account. If not sign-up here: [Sign-Up for a Confluent Cloud Account](https://confluent.cloud/signup)

-	This Blog assumes you already have an Active AWS account. If not [Create and activate a new AWS account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/)

## Creating AWS Resources required for Redshift Streaming Module

Below CloudFormation Script will deploy: 

- 	A new VPC 
-	A Redshift Cluster, configure it to be publicly accessible, 
-	A Lambda Function which will simulate streaming data, convert JSON records read from an s3 bucket to Avro format and publish into Kafka Cluster running on Confluent Cloud 

Click on below button to launch the CloudFormation template which will spin-up required resources for this workshop:

[![Foo](https://github.com/jobinthompu/Confluent-Dev-Day/blob/master/Resources/images/LaunchStack.png)](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/create/review?stackName=Confluent-Redshift-Connector&templateURL=https://cloudformation-template-repo.s3.amazonaws.com/ConfluentRedshift.json)

On the Quick Create Stack page, acknowledge the resource creations and click *‘Create Stack’*. It may take a few minutes to complete the stack creation. Stack name is already populated as ‘Confluent-Redshift-Connector’

Once created navigate to the [Stack Output](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/outputs?filteringText=&filteringStatus=active&viewNested=true&hideStacks=false&stackId=Confluent-Redshift-Connector) tab and note down Redshift End Point and Lambda Function Name. We will require it for future steps. 


![alt tag](https://github.com/jobinthompu/Confluent-Dev-Day/blob/master/Resources/images/CreateCluster.png)

## Creating Confluent Cloud Resources

### Creating a Kafka Cluster

Now let’s Launch Confluent Cloud Console from another tab on the browser: https://confluent.cloud/login

Once logged in, click on the Create Cluster Icon on the screen to create a new Kafka Cluster. If you already have a cluster, you can create a new topic to complete the exercise.

![alt tag](https://github.com/jobinthompu/Confluent-Dev-Day/blob/master/Resources/images/ClusterConf.png)


On the next page,  let’s provide a name for the Cluster, say Data2Redshift and Select Provider as Amazon Web Service, Region as us-east-1 and click continue. In a few mins your cluster will be ready to use. Once created, you can navigate to ‘Cluster Settings’ and note down the Bootstrap Server address, which you will need in the later part of this exercise.

### Creating a Kafka Topic  

Once the cluster is started, let's create a topic. Click on the cluster, click Topics on the left navigation bar. Give a name say ‘redtopic’ and select the number of partitions required. Default is 6 and click ‘Create with Defaults’.

![alt tag](https://github.com/jobinthompu/Confluent-Dev-Day/blob/master/Resources/images/CreateTopic.png)

### Creating Kafka API Keys  

Once the topic is created, click on Kafka API Keys left navigation bar. And click Create Key. 

![alt tag](https://github.com/jobinthompu/Confluent-Dev-Day/blob/master/Resources/images/CreateAPI0.png)

On the Create an API Key pop-up, make sure to note down the Key and Secret before clicking continue as you will not be able to view the secret once the pop-up closes. You will be needing it in the later steps.

![alt tag](https://github.com/jobinthompu/Confluent-Dev-Day/blob/master/Resources/images/CreateAPI1.png)

### Enabling Schema Registry for Confluent cloud and Create API Keys  

Click Schemas on the navigation bar, Choose Amazon Web Services (AWS) as cloud provider, region as US and click Enable Schema Registry. Note down the Schema Registry endpoint URL, you will need it in the later steps. Next Click on Manage API Keys as show below:

![alt tag](https://github.com/jobinthompu/Confluent-Dev-Day/blob/master/Resources/images/EnableSR0.png)

On the API Access tab, click ‘Create Key’ and On the Create a new Schema Registry API Key pop-up, make sure to note down the Key and Secret before clicking continue as you will not be able to view the secret once the pop-up closes. You will be needing it in the later steps. Click Continue and proceed 

![alt tag](https://github.com/jobinthompu/Confluent-Dev-Day/blob/master/Resources/images/EnableSR1.png)

### Configuring Confluent Cloud Redshift Sink Connector  

Navigate back to Cluster and click Connectors on the left navigation bar. From the options select Redshift Sink. 

![alt tag](https://github.com/jobinthompu/Confluent-Dev-Day/blob/master/Resources/images/AddSink.png)

On the Add Amazon Redshift Sink Connector page, enter below information. You can leave non mandatory fields empty, it will pick default values:

| Key	        		| Value         								| 
| ----------------------|:---------------------------------------------:| 
| Topics      			| redtopic										| 
| Name      			| RedshiftConnector      						|  
| Kafka API Key 		| [Kafka Key]									| 
| Kafka API Secret 		| [Kafka Secret]	      						| 
| Input message format 	| Avro      									| 
| AWS Redshift domain 	| [Redshift Host from CloudFormation output] 	| 
| AWS Redshift port 	| 5439      									| 
| Connection user 		| awsuser      									| 
| Connection password 	| Awsuser01     								| 
| Database name 		| streaming-data      							| 
| Database timezone 	| [Select your timezone]     					| 
| Auto create table 	| True      									| 
| Auto add columns 		| True		    								| 
| Tasks			 		| 1												| 

![alt tag](https://github.com/jobinthompu/Confluent-Dev-Day/blob/master/Resources/images/ConfigureSink.png)

Click Continue to proceed and on the test and verify page, click launch to deploy the redshift connector.

![alt tag](https://github.com/jobinthompu/Confluent-Dev-Day/blob/master/Resources/images/ConfigureSink1.png)

In a few minutes, the connector will be in running state and you are set to start streaming.

![alt tag](https://github.com/jobinthompu/Confluent-Dev-Day/blob/master/Resources/images/ConnectorRunning.png)

### Launching Lambda Streaming function to ingest data into Kafka Topic

Now let’s navigate back to the [CloudFormation Resources](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/resources?stackId=Confluent-Redshift-Connector) page and click on the ConfluentStreamingFunction Physical id link to launch Lambda console. 

![alt tag](https://github.com/jobinthompu/Confluent-Dev-Day/blob/master/Resources/images/CfOutPut.png)

While on the Lambda console, click on the Test button on the top right corner to simulate streaming data. 

![alt tag](https://github.com/jobinthompu/Confluent-Dev-Day/blob/master/Resources/images/StreamingFunc.png)

On the ‘configure test event’ pop-up provide below json record, which in below format with fields listed for function to run. Provide a name for the test event and click Save to create test event.

	{
  	"BOOTSTRAP_SERVERS": "<YOUR_KAFKA_BOOTSTRAP_SERVER>",
  	"API_KEY": "<YOUR_KAFKA_API_KEY>",
  	"API_SECRET": "<YOUR_KAFKA_API_SECRET>",
  	"SR_URL": " <YOUR_SCHEMA_REGISTRY_URL>",
  	"SR_AUTH_KEY": "<YOUR_SCHEMA_REGISTRY_KEY>",
  	"SR_AUTH_SECRET": "<YOUR_SCHEMA_REGISTRY_SECRET>",
  	"BUCKET": "streaming-data-repo",
  	"TOPIC": " redtopic"
	}

![alt tag](https://github.com/jobinthompu/Confluent-Dev-Day/blob/master/Resources/images/ConfigureTest.png)

Once saved, click test again to launch the function to stream data into the Kafka Topic provided in the configuration. Once clicked the function will run for 2minutes before it times out. [Timeout can be adjusted in the lambda configuration from console itself, its set to make sure streaming is stopped to avoid resource consumption]

### Preview Data Ingested into Redshift table from console

Now let’s navigate to [Redshift Query Editor](https://console.aws.amazon.com/redshiftv2/home?region=us-east-1#query-editor:) from AWS console to verify the data loaded into the Table. The table will be having the same name as the topic if you left the redshift connector configurations to default. 

Once On the Redshift Query Editor, select the cluster and login with redshift cluster credentials provided earlier. Select the ‘Public’ Schema and choose the table ‘redtopic’ and right click and choose option ‘Preview Data’. 

![alt tag](https://github.com/jobinthompu/Confluent-Dev-Day/blob/master/Resources/images/PreviewTable.png)

You should be able to view the data streamed by the redshift connector. You may query the data or connect to [Amazon QuickSight](https://aws.amazon.com/quicksight/) to start  visualizing it. With this we have come to end of this blog post.

## Clean Up

- Once you complete the exercise go ahead and delete the Topic/Cluster you created in confluent cloud not to incur any unwanted resource consumption
- When you finish also remember to clean up all AWS resources that you created using AWS CloudFormation. Use the [AWS CloudFormation console](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks) or AWS CLI to delete the stack named “Confluent-Redshift-Connector”.

## Conclusion

In this post, we learned about Confluent Cloud, its Architecture and how it can help you stream data from Kafka topics to Redshift with Kafka connect. Did hands-on steps to launch a Redshift Cluster, Created a Kafka Cluster in Confluent Cloud, setup Redshift connector and streamed data directly into Redshift Table.  
Confluent Cloud offers a serverless experience with elastic scaling and pricing that charges only for what you stream. If you would like to learn more about Confluent Cloud , [Sign-Up for a Confluent Cloud Account](https://confluent.cloud/signup)
