# AWS-SAP-C02-Notes
These notes are my attempt at helping myself and possibly others to pass the AWS Certified Solutions Architect Professional Exam. I strongly recommend taking Solutions Architect Associate exam before diving into this. These notes are just key points to remember before attempting the exam and not a complete guide. I'd recommend taking Stephane Maarek's course [Ultimate AWS Certified Solutions Architect Professional 2023](https://www.udemy.com/course/aws-solutions-architect-professional/) and taking a few practice exams from Tutorials Dojo [AWS Certified Solutions Architect Professional Practice Exam](https://tutorialsdojo.com/courses/aws-certified-solutions-architect-professional-practice-exams/) and finally AWS Whitepapers and Docs for concepts you want to know indepth. That's all I had used to pass the exam.

Note: The author makes no promises or guarantees on this guide as this is as stated, a guide used by myself and nothing more.


## S3

IOPS - How fast can R/W happen

Throughput - How much data can be moved

Max size - 5TB

Max size in single put - 5GB

MultiPart uploads recommended for size over 100MB

Safeguard against accidental deletion - enable MFA and Versioning on Bucket

Data can be protected by enabling cross region replication

S3 Standard: This is the default storage class and provides high durability, availability, and performance for frequently accessed data. It is suitable for storing data that requires low latency and high throughput.

S3 Intelligent-Tiering: This storage class uses machine learning to automatically move objects between two access tiers (frequent and infrequent) based on changing access patterns. This helps optimize costs by automatically moving objects to the most cost-effective tier.

S3 Standard-Infrequent Access (S3 Standard-IA): This storage class is designed for data that is accessed less frequently but still requires high durability, availability, and low latency access. It has a lower storage cost but higher retrieval cost compared to S3 Standard.

S3 One Zone-Infrequent Access (S3 One Zone-IA): This is a cost-effective storage class that stores data in a single availability zone. It is suitable for storing data that can be recreated easily, such as non-critical backup copies.

S3 Glacier: This storage class is designed for long-term archival and data retention. It provides very low storage costs but has a longer retrieval time (minutes to hours) compared to other storage classes.

S3 Glacier Deep Archive: This is the lowest-cost storage class for long-term data archival and retention. It provides the lowest storage cost but has a longer retrieval time (12 hours) compared to other storage classes.

Lifecycle management can destroy objects, Intelligent tiering can move objects between tiers.

S3 supports encryption at rest using AWS key, Customer key, KMS Key or you can use local encryption process Encrypt on client side and then upload.

SSE-S3 key is managed and rotated by aws

SSE-KMS key AWS manage the data key but you manage the customer master key (CMK) in AWS KMS

SSE-C requires that you manage the encryption key

S3 allows you to apply a policy to enforce https connections only

For customer managed S3 SSE-C encryption,
For Amazon S3 REST API calls, you have to include the following HTTP Request Headers:
* x-amz-server-side-encryption-customer-algorithm
* x-amz-server-side-encryption-customer-key
* x-amz-server-side-encryption-customer-key-MD5 
 
For presigned URLs, you should specify the algorithm using the x-amz-server-side-encryption-customer-algorithm request header.

### Tricks in S3

Transfer Acceleration: Speed up data uploads using cloudfront in reverse

Requester Pays: The requester can pay for files they'll download instead of bucket owner

Tags: Can assign tags to objects for use in billing, security, etc

Events: Trigger notifs to SNS, SQS or Lambda on upload/delete/update/restore

Static Web Hosting

Bittorrent Protocol: Use Bittorrent protocol to retrieve publically available object by automatically generating .torrent files. Share objects in S3 over bittorrent protocol

S3 sync feature is available in AWS CLI using which you can sync data to S3, without manually uploading objects to S3

Remember that the sync feature of S3 only uploads the “delta” or in other words, the “difference” in the subset, it is better for migration tasks with time constraint. Sync can be done days before and then consecutive sync tasks will take fraction of time.

S3 bucket events can directly trigger a lambda function without eventbridge

Origin Access Identity is mainly used to restrict access to objects in S3 bucket and only allow access from cloudfront

When Versioning feature is enabled in S3, it causes all of the existing files to have a Version ID of null

### Glacier

Glacier is used by AWS Storage Gateway Virtual Tape Library

Glacier Has Faster retrieval speeds options if you pay more, not too fast to stream but fast retrieval of data

You don't necessarily need S3 to access glacier. Glacier Vault exists which can be given access to by IAM roles.

Glacier Vault is an archive container

Glacier Vault Lock can be used for access control like enforcing MFA or immutable objects.

Once a glacier vault lock is attached to glacier vault, it is permanent

Bucket policies are resource based policies, if bucket explictly allows access to a user/resource then the policy in IAM role makes no difference.

aws:SecureTransport condition can be used in bucket policies to enforce https connections

## EBS

EBS are tied to single AZ and can only be used with EC2

You can use EBS Multi-Attach to attach an EBS volume to multiple instances

Use Snapshots to migrate EBS to different AZ

EBS Snapshots can be used to create encrypted volume from unencrypted volume.

They are used to share EBS data with other accounts

Snapshots work as incremental backup, consecutive snapshots only record change from previous snapshot and does not record entire volume.

Use Amazon Data Lifecycle manager to schedule snapshots

## Instance Stores

Instance Stores are locked to an EC2 instance, they are Ephermal and can only be used for cache, buffers, etc.

Data is lost once instance stopped or terminated.

Instance Stores perform better than EBS because they are directly attached to instance and are not on network like EBS

## EFS

EFS, We only pay for what we use (Not in case of EBS)

EFS has multi AZ Support and we can configure mount points from multiple AZ

It also supports on prem but make sure we have direct connect for stability but it doesn't take care of security.

Amazon DataSync is recommended to sync data between on prem and AWS EFS

EFS is 3x more expensive than EBS and 20x more expensive than S3

## Storage Gateway

Storage Gateway is a VM which you can download and run onsite with VMWare or HyperV or on EC2. Provides local storage resources backed by AWS S3 and Glacier.

You can spin up a Storage Gateway in your datacenter, mount volumes on it and then sync it to S3

It is often used as a disaster recovery preparedness to sync data to AWS. Also useful for cloud migrations.

Storage Gateway has different modes
* File Gateway(File level): NFS/SMB Shares synced to S3

* Volume Gateway Stored Mode(Block level): Async replication of data to S3 via iSCSI

* Volume Gateway Cached Mode(Block level): Primary data is stored to S3 and frequently access data is cached locally, it uses iSCSI

* Tape Gateway: Virtual Tape Library to be used with existing backup software on prem, uses iSCSI
 
Used for cloud migration. Setup volume gateway stored mode to sync data to S3 and then switch over to cached mode because most of data is stored to S3.

## RDS

In occassions where database isn't supported by RDS, you can run a database on EC2.

RDS is best suited for structured, relational data store needs. Automated backups and patching in customer defined maintenance windows.

Push button scaling, replication and redundancy

RDS has Multi-AZ Synchronous replication. Use Read Replicas across different regions, replication using read replica across regions is async.

In multi AZ replication, if master DB fails, standby automatically becomes master. Read Replicas are not automatically promoted to master and it is a multistep process.

In case of region failure, Read Replicas need to be manually promoted to master db in single AZ and then we need to reconfigure it to multi AZ.

MariaDB is open-source fork of MySQL

RDS requires manual intervention to promote read replicas to master, however RDS can have automatic failover to standby databases within a region. 

Amazon Aurora automatically handles failovers to Read Replicas.

Oracle RAC is not supported by RDS

RDS provides Transparent Data Encryption (TDE) for Oracle and SQL Server

Reserved instances are recommended cost-saving to RDS instances that will be running continuously for years

Amazon RDS Proxy sits between your application and your relational database to efficiently manage connections to the database and improve the scalability of the application. Amazon RDS Proxy can be enabled for most applications with no code changes

## Types of Data

Lots of Large Binary Objects (BLOBs) --> use S3

Automated Scalability --> DynamoDB

Name/Value Pairs --> DynamoDB

Data is not predictable or not well structured --> DynamoDB

Unsupported Database platforms like IBM DB2 or SAP HANA --> EC2

Need complete control over database --> EC2

## DynamoDB

DynamoDB is managed multi-AZ noSQL data Store with cross region replication option

Reads are eventual consistent but you can request strongly consistent read via SDK parameters

Priced on throughput rather than compute. Provision read and write capacity in anticipation of need

Auto Scale capacity adjusts per configured min/max levels. It works by reading consistent throughput, triggers cloudwatch alarm which in turn scales the DB

On the downside of AutoScaling capacity, dynamoDB doesn't know when to scale down if there's no traffic as at assumes that it might get hammered with traffic anytime again

If you truly don't know the load, OnDemand Capacity is preferred for flexible capacity at small premium cost

Can achieve ACID Compliance with DynamoDB Transactions

For time series data in DynamoDB, AWS recommends one table per period.

General design principles in Amazon DynamoDB recommend that you keep the number of tables you use to a minimum. For most applications, a single table is all you need.

As a DynamoDB table grows, it will spit into more partitions. If the RCU and WCU remain constant, they are divided equally across the partitions. When the allocation of that partition is used up, you risk throttling. AWS will permit burst RCU and WCU at times but it is not assured. You could increase the RCU and WCU but this would increase cost. Therefore, we can archive off as much data as possible but also need to shrink the partitions down to something more reasonable. We can do this by backing up and recreating/restoring the table

DynamoDB Streams captures a time-ordered sequence of item-level modifications in any DynamoDB table and stores this information in a log for up to 24 hours

DynamoDB can also send the item level change data to Kinesis data streams to bypass DynamoDB streams' data retention limit of <24 hours. Kinesis data streams has limit of 365 days

Max Object size for DynamoDB can be 400KB, for bigger files, store them to S3 and then store a reference to it in DynamoDB. Writes on S3 can trigger a lambda function and lambda function can write object metedata to DynamoDB

Apache Cassandra can easily migrate to DynamoDB

ACID Support, multiple transactions across multiple tables

Can have Standard and IA table classes

* Local Secondary Index: Same primary key as parent table but can have alternate sort key. LSG must be defined at table creation time
* Global Secondary Index: Create a view with different primary key and sort key. Can be defined after table creation

Main difference between DynamoDB and traditional RDS is that you can only query by PK + Sort key (optional) on main table and indexes. If you want to query something on DynamoDB, you need to make sure that index for it is created in advance

* Global Tables: Cross Region replicated tables, active-active type of replication. Must enable DynamoDB Streams first

DAX (DynamoDB Accelerator) is a seamless cache for DynamoDB and requires no application rewrites. Writes also go through DAX to DynamoDB. DAX cache has 5 mins TTL by default. Supports upto 10 nodes in a cluster and is Multi AZ (Minimum 3 nodes recommended). Supports encryption

Upsert into DynamoDB is an insert command but if row exists, it will update it. Insert into DynamoDB would create duplicate rows in this case.

DynamoDB does not store data in json format, use documentDB

## Amazon Timestream

Amazon Timestream Database is built for storing and analyzing time series data. Can be used as an alternative to DynamoDB or RedShift and includes some builtin analytics like interpolation or smoothing
* Use Cases: Sensor Networks, Industrial Machinary, Equipment Telemetry

## DocumentDB

DocumentDB is MongoDB compatable Database. It emulates MongoDB API so it acts like MongoDB to existing drivers and clients.

Backup to S3, Multi-AZ, Integrates KMS, Scalable

DocumentDB is good at handling realtime big data stored in form of documents and is built to scale like amazon aurora

## VPC

For a VPC, the existing CIDR cannot be modified.

The VPC IPv4 CIDR block can be from /16 to /28.

The VPC IPv6 CIDR block size is fixed at /56. The subnet CIDR block size is fixed at /64

In a VPC with CIDR ..0.0/24 , the following five IP addresses are reserved:
* 0.0: Network address.
* 0.1: Reserved by AWS for the VPC router.
* 0.2: Reserved by AWS. The IP address of the DNS server is the base of the VPC network range plus two. ...
* 0.3: Reserved by AWS for future use.
* 0.255: Network broadcast address.

There can only be one Virtual GateWay attached to a VPC at a given time

## NAT Gateway

NAT instance has lower cost than NAT Gateway but comes with overhead of instance maintenance. AWS recommends NAT Gateway

## Application Load Balancer (ALB)

An Application Load Balancer cannot be assigned an Elastic IP address (static IP address). Use Global Accelerator to assign static IP to ALB

However, a Network Load Balancer can be assigned one Elastic IP address for each Availability Zone it uses.

## IAM

To setup cross account sharing:
1. Define permissions that the cross account will have when they assume a role you want to share
2. Create a trust policy defining the principals that can grant access to the given role
3. The trusted account must grant assumerole permissions to the user/group/service

Don't confuse permission sets with trust policies

When you create a role which you want to share across accounts, Permission sets are defined set of allowed actions when someone assumes a role

Trust policies are "Exactly who can access the created role and from which accounts"

Most common use cases for cross account sharing/assume roles
1. Allow external or temporary users limited access to your aws account
2. Cross account serverless to trigger events in secondary account
3. Application permissions for workloads running across accounts

AWS STS is used in SSO for user authentication. Amazon Cognito is more preferred for mobile applications, it has SDK with security built into it.

Cognito users are seperate from IAM users.

For mobile apps, AWS recommends using the Cognito SDK for managing temporary creds

Use Amazon Cognito Identity Pools for mobile apps

OAuth 2.0 is the industry-standard protocol for authorization.

When assuming a role to access a resource in account B, the user gives up all access privileges of its role in account A. If question asks user to use multiple resources in both accounts, it is better to opt for resource based policy instead

IAM Permission Boundaries is an advanced feature to use a managed policy to set the maximum permissions an IAM entity can get

Used to allow developers to self-assign policies and manage their own permissions while making sure they can't escalate their privileges

useful to restrict one specific user instead of whole account using organizations and SCP

IAM Access Analyser is used to find out which resources are accessed externally. Works on S3, IAM Roles, KMS Keys, Lambda, Layers, SQS, Secrets Manager Secrets

You define a zone of trust and anything outside the zone of trust is IAM Access Analyser findings

IAM Access Analyzer can generate policy based on access activity, it reviews cloudtrail logs for upto 90 days and then writes access policies for calls made.

Conditions in IAM policies can be used for conditional access

e.g only let users with MFA access a specific resource

e.g only let users access a particular tagged resource

External ID is important thing while sharing resources with third parties. External ID is a secret between you and third party which you need to define. It requires third party to have an aws account ID.

Amazon cognito replaces token vending machine (TVM)

IAM can securely encrypt your private keys and stores the encrypted version in IAM SSL certificate storage

## AWS Organizations

Set up AWS Organizations by sending an invitation to all member accounts of the company from the master account of your organization

Create an OrganizationAccountAccessRole IAM role in the member account and grant permission to the master account to assume the role

A publishing account is an account that is used to host standardized versions of resources, such as golden AMIs or a corporate ECR with docker images. These resources can then be shared with other accounts in the organization.

Here are some of the benefits of using a publishing account:

* Centralized management of resources: All of the standardized resources are stored in one place, making it easier to manage them.
* Improved security: The publishing account can be configured with strict security policies to protect the resources.
* Increased efficiency: By sharing resources, you can reduce the amount of time and effort that is required to create and maintain them.

If you are looking for a way to centralize the management of resources and improve security, then you should consider using a publishing account.

Here are some of the steps that you can follow to set up a publishing account:

* Create a new account in AWS.
* In the new account, create the standardized resources that you want to share.
* Configure the security policies for the publishing account.
* In the other accounts in the organization, grant access to the publishing account.

Once you have set up a publishing account, you can easily share standardized resources with other accounts in the organization. This can help you to improve the efficiency and security of your AWS environment.

Take note that the NotAction/NotResource is an advanced policy element that explicitly matches everything except the list of action/resources that you specified.

Consolidated billing for instances is tagged per AZ and not per account. Instances running in same AZ but different accounts in same organization will benifit from the purchase.

Organization account access role is automatically created when a management account in OU creates a member account using aws Organizations API. That role is assumed everytime the management account performs any tasks on new account.

Consolidated Billing feature
* Consolidated billing across all accounts
All Features
* Ability to apply SCP so members cannot leave orgs
* Can't go back to consolidated billing once opted for it

SCP does not apply to (First account/root OU) management account, it is a master account with full access.

## Lambda@Edge

Lambda@Edge is an extension of AWS Lambda which lets you execute functions that customize the content that CloudFront delivers.

Lambda@Edge can only change request and response at edge locations, it doesn't cache

Use Cases of Lambda@Edge:
* Manipulate HTTP requests and response
* Implement request filtering before reaching your application
* User authentication and authorization
* Generate HTTP responses at edge
* A/B Testing
* Bot mitigation at edge

## Cloudfront

Note that you can’t use a self-signed certificate for HTTPS communication between CloudFront and your origin.

There is no default SSL certificate in ELB, unlike what we have in CloudFront (*.cloudfront.net)

Field-level encryption in cloudfront allows you to securely upload user-submitted sensitive information to your web servers

S3 Cross Region Replication is great for serving dynamic content and Cloudfront is great for caching Static content at edge locations 

Resources serving as origin for cloudfront should be PUBLIC

To prevent users from directly accessing exposed cloudfront origin, use custom HTTP headers and only let the services respond to requests with header sent by cloudfront

You can also use security groups to only allow requests from cloudfront to prevent public access

Cloudfront has origin groups for origin failover (primary/secondary) which can be cross region

Cloudfront can also check response from origin and accordingly cache and display error web pages

## Cloudtrail

You can configure CloudTrail to deliver log files from multiple regions to a single S3 bucket for a single account.

Cloudtrail supports centralized logging from multiple regions

Cloudtrail events are stored for 90 days by default. Log them to S3 and use athena to analyse older logs

Cloudtrail insights is a paid service which analyses unusual activity in your account

Don't confuse with guardduty. This is for unusual API activity within aws accounts where as guardduty checks for compromised services/instances by checking vpc flow logs, dns logs, etc

Cloudtrail may take upto 15 minutes to deliver an event

Cloudtrail cannot track queries made to rds

## Application Migration Service (MGN) or Server Migration Service (SMS)

Server migration connector is downloaded as a virtual appliance into on-prem vSphere or Hyper-V setup

## Database Migration Service (DMS) and Schema Conversion Tool (SCT)
	
Schema Conversion Tool (SCT) Does not support No SQL Databases.

A homogeneous migration is when we migrate between the same type of databases.

## Amazon MQ

Amazon MQ provides two managed broker deployment connection options: public brokers and private brokers.

Public brokers receive internet-accessible IP addresses, while private brokers receive only private IP addresses from the corresponding CIDR range in their VPC subnet. 

Under the hood, the private IP address option is effectively acting as a VPC endpoint. Connectivity over the private IP never leaves the Amazon network.

Amazon MQ runs on servers and can run in Multi-AZ with failover

Migrating on prem queue service like Rabbit MQ, Apache ActiveMQ, etc to Amazon MQ would be re-platform

## Kinesis Data Streams

Kinesis Data Stream - Process data on the fly for ML Models or Analytics (Near Real-Time)

Kinesis Data Streams can store record from 24 hours (default) to 365 days. It is a transient data store, don't treat it as persistent.

Kinesis Data Streams and Firehose can only write data upto 1 MB/s or 1000 messages/s per shard

Once data is inserted in kinesis, it can't be changed (immutable)

Kinesis Agent can write to both data streams and firehose.

Kinesis Producer Library (KPL) does batch, compression, retries, C++, Java

You can use Amazon Kinesis API to build your Amazon Kinesis Application. However, AWS recommends using Amazon Kinesis Client Library (KCL) if applicable because it performs heavy-lifting tasks associated with distributed stream processing, making it more productive to develop applications.

Put into Kinesis Data Streams - Producer (Kinesis Producer Library)

Pull from Kinesis Data Streams - Consumer (Kinesis Consumer Library)

Kinesis Producers:
* 1MB/s or 1000 messages/s write per shard
* ProvisionedThroughputException otherwise
* Solution: Increase Shards
Kinesis Consumers:
* 2MB/s read per shard across all consumers
* 5 API calls per second per shard
Kinesis Consumer Enhanced Fan-Out Model:
* 2MB/s read Per Shard, per enhanced consumer. (No API Calls needed, push based service)

Default limit is 500 shards, can be increased to unlimited. Shard capacity can be on-demand (expensive) or pre provisioned

## Kinesis Data Firehose

Kinesis Data Firehose - Store data in S3, ElasticSearch, RedShift, Splunk, HTTP endpoints

Kinesis Data Firehose can also transform data by calling a lambda function before writing it to S3

Kinesis firehose has a buffer and if buffer time/size (min buffer time is 60 secs) is reached, the buffer is flushed to the target destination. Hence it is not real-time, it is near real-time!

If real-time flush from Kinesis Data Streams to S3 is needed, firehose is not a good option!

Use Data Streams --> Lambda --> Target, it will cost a bit more than firehose but will also be faster

## Simple Workflow Service (SWF)

SWF is strictly a status tracking system which determines status of your task.

Tracks state of your workflow when you interact with it via API.

Best suited for human enabled workflows like order fulfilment or procedural requests

Activity Worker is a program that interacts with SWF service to get tasks, process them and return results.

## AWS Batch

AWS Batch can boot up an instance, run jobs on it and terminate it when they're done.

AWS Batch in the back end, runs jobs using ECS, EKS, or Fargate (Batch tasks need to have correct IAM Role)

* Batch Managed Compute Environment: You can select an option of running jobs on On-Demand Instances or spot instances to reduce costs. - in VPC

AWS Batch Multi Node Mode is good for large scale HPC

Leverage multiple EC2 instances at same time, good for tightly coupled workflow where multiple instances need to be active at same time

1 main node and many child node, doesn't work with spot instances for now. Works even better with EC2 cluster placement group

## AWS Auto Scaling

Auto Scaling cooldown defaults to 5 minutes and larger cooldown periods will result in slower scale in/out as the system waits for cooldown period to expire to determine to scale further.

Step Scaling policies don't have a cooldown period and have an instance warm up period where you can specify how long to wait for instance to come up so that another step scaling can be triggered.

Use detailed cloudwatch monitoring in auto scaling and EC2 instances to monitor load minute-by-minute. Basic monitoring has granularity of 5 minutes.

## Amazon Redshift

Redshift does not support multi-AZ deployment. Best bet is to use multi-node cluster which supports data replication and node recovery

Redshift (must provision servers) can be configured to automatically copy snapshots of a cluster to another region

Snapshot copy grant needs to be setup between regions for redshift clusters to seamlessly copy encrypted snapshots

Redshift spectrum (serverless) enables to query data from S3 without loading it. Must have a redshift cluster available to start query

Reshift Workload Management (WLM) enables you to flexibly manage queries' priorities within workloads
	* Automatic WLM: Managed by Redshift
	* Manual WLM: Managed by User

Redshift concurrency scaling (Concurrency Scaling Clusters) enables you to provide consistently fast performance with virtually unlimited concurrent users and queries

## AWS CloudFormation

Change Sets can allow you to preview changes to a CloudFormation stack before they are applied. Make use of change sets after updating cloudformation template to identify potential trouble spots

Use Stack Policies to explicitly protect sensitive portions/resources of your stack

Stack Policy is deny by default on all resources and requires an explicit allow for resources to be updated by cloudformation template

Once Stack Policy is applied, we cannot remove it. It can only be updated using the CLI

AWS:CloudFormation::CustomResource can be used to call out to fetch a value from a third-party API

When you associate a Lambda function with a custom resource, the function is invoked whenever the custom resource is created, updated, or deleted. AWS CloudFormation calls a Lambda API to invoke the function and to pass all the request data (such as the request type and resource properties) to the function

You want to create a snapshot of the volume when the resource is deleted by CloudFormation. What is the easiest method for you to take?

The easiest method is using the DeletePolicy attribute in the CloudFormation template

The "Snapshot" value ensures that a snapshot is created before the CloudFormation stack is deleted

The "Retain" value is incorrect as it keeps the volume rather than creates a snapshot

Cloudformation StackSets can assume execution role in member accounts and deploy stacks and identical rules across all accounts and regions

## AWS OpsWorks

AWS OpsWorks is AWS managed instance for chef and puppet. Provides configuration management to deploy code, automate tasks, configure instances, perform upgrades, etc

OpsWorks Stacks is an AWS creation and uses embedded chef client installed on EC2 instances to run chef recipies. It also supports on prem servers with agent.

EC2 instances, RDS instances, ELBs are examples of layers in OpsWorks Stacks

Stacks can be cloned within same region

OpsWorks is a global service but when you create a stack, you must specify a region

You can't manage cross region opsworks stacks, resource created by stack needs to be in same region as the resource managing it

OpsWorks Stacks offers three types of scaling: 
	* 24/7 for instances that remain on all the time; 
	* time-based for instances that can be scheduled for a certain time of day and on certain days of the week; 
	* load-based scaling which will add instances based on metrics. 

All this can be configured from within the OpsWorks Stack console

## AWS Config

AWS Config rules can check resources for certain desired conditions and if violations are found, the resource is flagged as noncompliant. Works well with eventbridge

Examples of config rules:
* Is backup enabled on RDS?
* Is CloudTrail enabled on AWS account?
* Are EBS volumes encrypted?

Enforcing standardized tagging can be done via aws config rules and custom scripts. e.g EC2 instances not properly tagged are stopped or terminated

Most resources can have upto 50 tags

Resource Groups are grouping of AWS resources defined by tags

AWS Config must be enabled for security hub to work

## AWS System Manager

* Inventory: Collect metadata about instances
* State Manager: Creates a state to represent a certain configuration is applied to instances. e.g keep track of instances that have been updated to the current stable version of Apache HTTP Server
* Parameter Store: Shared secure storage for config data, connection strings, passwords, etc
* Insight Dashboards: Account level view of CloudTrail, Config, Trusted Advisor
* Resource Groups: Group resource through tagging for organization
* Maintenance Windows: Define schedules for instances to patch, update, run scripts, etc for other services in aws like patch manager.
* Automation: Automating routine maintenance tasks and scripts
* Run Command: Execute a script or powershell command without having to rdp into instance. No need for SSH access, just need to install ssm agent (Which comes installed on aws AMIs). e.g run a shell script on 53 different instances at the same time
* Patch Manager: Automates process of patching instances for updates. Acts as a pre-approval gatekeeper on which patches are auto approved. Patch Manager supports linux, windows and MacOS. Use tags for grouping.
	Patch Baseline (Multiple if you have multiple environments)
	Patch Group: Group based on tags
	Define Maintenance Windows: Schedule, duration, registered tasks, etc
	Add AWS-RunPatchBaseline Run Command as part of registered tasks of the maintenance window (works cross platform)
	Rate control: concurrency and error control
	Monitor Patch Compliance using SSM Inventory
* SSM Documents: They're json or yaml files. Defines actions that system manager performs, includes different steps and parameters that you specify and they're stored as versions and can be shared across different accounts.
* Session Manager: Allows you to start a secure shell on your instances. Does not need SSH access, bastion hosts, or SSH Keys. All data running through session manager can be logged.
* OpsCenter: Resolves Operational issues. Aggregates information to resolve issues on each OpsItems like CloudTrail Logs, AWS Config changes and relationships, CloudWatch Alarms, CLoudFormation Stack Information

Parameter policies are only for advanced parameter store and they allow parameters to have an expiration date

There's no way to share secret from secrets manager using resource access manager. Instead use resource based policy on secret

## Amazon Workspaces

Amazon Workspaces and AppStream are similar but different. Workspaces are desktop as a service. AppStream is Desktop Application hosting on web UI and streaming service, clients can access AppStream applications via web browser and work on them.

e.g give virtual desktop or hosted applications for seasonal workers to access your internal services and prevent data theft

e.g AppStream can be used for product demo by preconfiguring everything and access application via web browser

## AWS Connect

AWS Connect is fully managed contact center solution. It has configurable call handling, inbound/outbound telephony, it can be integrated with lex for interactive voice response, chatbot and analytics.

Can integrate with other enterprise customer relationship management (CRM) systems.

## Other AWS Services

Amazon Chime is online meeting and video conferencing service. Like skype, zoom, etc

WorkDocs is online document storage and collaboration platform. Google Drive or Dropbox for corporations. Supports version management, sharing, etc

Amazon WorkMail is fully managed email and calender as a service. Compatable with Outlook, Android, IOS, IMAP and other mail clients

Amazon WorkLink is secured reverse proxy. Provide secure access to internal web applications for mobile devices. It is one step below a full vpn connection

Alexa for Business is deploying Alexa functionality and features internally in your enterprise. There are hooks and SDKs available for it. e.g use alexa devices to access corporate data. (is conference room booked? tell me about net sales last month?)

## Amazon Sagemaker

Amazon Sagemaker is managed Jupyter Notebook deployment. It also provides ways to train and validate ML models using docker containers. You can use AWS' builtin algorithms or bring your own algorithms and build own container. Sagemaker can help with deploying ML models at scale by using endpoints.

## Blue/Green Deployment

A blue/green deployment is a deployment strategy in which you create two separate, but identical environments. One environment (blue) is running the current application version and one environment (green) is running the new application version. 

Using a blue/green deployment strategy increases application availability and reduces deployment risk by simplifying the rollback process if a deployment fails. 

Once testing has been completed on the green environment, live application traffic is directed to the green environment and the blue environment is deprecated.

Blue/Green is not just for nimble web apps in the cloud, most huge major upgrades can use blue/green deployments

## Route 53

Route 53 weight is an integer going from 0 to 255

AWS recommends a minimum of at least 10 secs of TTL because some DNS providers don't honor 0 TTLs

For geolocation routing, you need to be sure you have a default record in the case that the location cannot be determined

Public hosted zones in Route 53 contain records that specify how you want to route traffic on the internet.

Private hosted zones in Route 53 contain records that specify how you want to route traffic in an Amazon VPC

Private Hosted Zones provide DNS services to VPCs but cannot be accessed from the internet

You must enable enableDNSHostnames and enableDNSSupport in VPC settings for private hosted zones to work

They need to be associated with VPCs for them to work. They can be associated with VPCs either by the console, CLI or programmatically via SDK

Route 53 is DNSSEC compliant

DNSSEC works only with public hosted zones

Route 53 has Inbound endpoints and outbound endpoints for Hybrid DNS. Forwarding Resolver rules are mapping of domain name to IP for query forwarding. System rules override forwarding rules.
    * Inbound Endpoint – provides DNS resolution of AWS resources, such as EC2 instances, for your corporate network
    * Outbound Endpoint – provides resolution of specific DNS names that you configure using forwarding rules to your VPC

Resolver Rules work seamlessly with RAM for cross account resource sharing

Route 53 can do health checks on cloudwatch alarms

Route 53 can combine health checks from multiple instances into one healthcheck. Supports upto 256 entries

Route 53 is outside vpc and cannot access private endpoints. Workaround can be to use cloudwatch alarms

If you want to associate VPCs that you created by using one account with a private hosted zone that you created by using a different account, you first must authorize the association

If you want to associate multiple VPCs that you created with one account with a hosted zone that you created with a different account, you must submit one authorization request for each VPC

You can’t use the AWS console either to authorize the association or associate the VPCs with the hosted zone

You can configure Amazon Route 53 to log the query information to a CloudWatch Logs log group.

Then you can use CloudWatch Logs tools to access the query logs

The logs from Route 53 cannot be directly forwarded to an S3 bucket or CloudWatch Metrics

VPC Flow Logs can capture IP traffic information going to and from network interfaces but the logs do not contain the DNS query data

Route 53 geolocation redirects traffic and does not block it. Cloudfront Georestriction can be used for blocking countries

Route 53 supports:
	A (address record)
	AAAA (IPv6 address record)
	CNAME (canonical name record) Only for non root domain e.g something.company.com
	CAA (certification authority authorization)
	MX (mail exchange record)
	NAPTR (name authority pointer record)
	NS (name server record)
	PTR (pointer record)
	SOA (start of authority record)
	SPF (sender policy framework)
	SRV (service locator)
	TXT (text record)
	Amazon Route 53 also offers alias records

## Deployment Strategies

An in-place upgrade upgrades the operating system files while your personal settings and files are intact.

A Disposable Upgrade is one were a new release is deployed on new instances while instances containing the old version are terminated.

immutable: in the same environment (so under the same load balancer) a new autoscaling group is created alongside the old one. As soon as the first new instance is created it starts to serve traffic. When the new instances are all healthy the old ones are switched off.

blue/green: a new environment is created from scratch (so another load balancer). The switch is performed at DNS level routing the traffic from the OLD to the NEW when the new environment is ready and healthy.

The main difference is that in the immutable update, the new instances serve traffic alongside the old ones, while in the blue/green this doesn't happen (you have an instant complete switch from old to new).

All-at-once deployment means all traffic is shifted from the original environment to the replacement environment all at once. All at Once deployment offers the shortest deployment time but it will incur downtime as the instances are upgraded.

## Elastic Container Service (ECS)

Part of ECS:

Service Discovery makes it easy for containers within an ECS cluster to discover and connect with each other, using Route 53 endpoints.

Task Definitions define the resource utilisation and configuration of tasks, using JSON templates.

Task Scheduling allows you to run batch processing jobs run on a schedule.

ECS Cluster is logical grouping of EC2 instances

ECS Service defines how many tasks should run and how they should run

ECS Task IAM Roles allows each task to have an IAM role

ECS Instance Profile is used to cluster ECS services to send logs to cloudwatch logs

ECS on EC2 has seamless integration with load balancer with dynamic port mapping

Architecture: ECS Cluster --> EC2 Instance --> ECS Service --> Task Definition --> ECS tasks (container applications)

ECS doesn't support third party addons, use EKS

With a diversified Spot instance type and Spot instance draining, you can allow your ECS cluster to spawn other EC2 instance types automatically to handle the load at a very low cost

You can configure various Docker networking modes that will be used by containers in your ECS task. The valid values are none, bridge, awsvpc, and host. The default Docker network mode is bridge.
* None: the task’s containers do not have external connectivity, and port mappings can’t be specified in the container definition.
* Bridge: the task utilizes Docker’s built-in virtual network which runs inside each container instance.
* Host: the task bypasses Docker’s built-in virtual network and maps container ports directly to the EC2 instance’s network interface directly. In this mode, you can’t run multiple instantiations of the same task on a single container instance when port mappings are used.
* awsvpc: the task is allocated an elastic network interface, and you must specify a NetworkConfiguration when you create a service or run a task with the task definition. When you use this network mode in your task definitions, every task that is launched from that task definition gets its own elastic network interface (ENI) and a primary private IP address. The task networking feature simplifies container networking and gives you more control over how containerized applications communicate with each other and other services within your VPCs.
awsvpc is default mode for fargate tasks

ECS Anywhere is used to easily run containers on on prem servers

ECS Container Agent and SSM agent needs to be installed on servers

Must have stable connection to AWS Regions

EKS Anywhere is also Kubernetes running on prem but here we leverage Amazon EKS Distro (Amazon's own flavour of EKS) on prem

This does not need a connection to AWS region and just needs EKS Anywhere Installer to be run on the system

Optionally, You can use EKS Connector to connect EKS Anywhere clusters to AWS

You can leverage EKS console in two modes 
* Fully Connected & Partially Disconnected
* Fully Disconnected

Fargate tasks cannot be invoked by eventbridge

## Continous Integration and Continuous delivery (CI/CD)

AWS CodeCommit is hosted github repository

AWS CodeBuild helps with compiling code, running tests and create deployment packages

AWS CodeDeploy provides all of the following deployment features like Automated deployments, Logging, Monitoring, Scaling, except Securing

AWS CodePipeline is a continuous delivery service that enables you to model, visualize, and automate the steps required to release your software

AWS CodeStar is a service that leverages other services like the ones above and has tools which helps us create templates to utilize above servies so you don't have to do things manually

CodeArtifact is only for store, publish and share artifacts. It does not allow the user to configure custom actions or scripts to perform unit tests on artifacts. It is recommended to use AWS CodeBuild for this scenario

## Cost Optimization

Appropriate Provisioning: Only Provision resources that you need and nothing more

Right-Sizing: Use lowest cost resource that still meets the technical specifications

Purchase Options: Reserved Instances provide best cost advantage, spot are good for temporary horizontal scaling, EC2 fleet lets you define target mix of OnDemand, reserved and Spot Instances.

Geographic Selection: Pricing vary from region to region

Managed Services: Leverage managed services over self-managed services to reduce cost overhead

Optimized Data Transfer: Moving data in AWS does not cost anything but moving data out of aws incurs charges. Data going between AWS regions can become a significant cost component.

## Reserved Instances

Zonal: Discount applies to specific selected AZ

Regional: If no AZ is selected, discount is applied to any instance in the family in any AZ in the region (Not guranteed if we'll get discount all the time in any AZ and it doesn't include capacity reservation)

Both of the above can be changed via console or ModifyReservedInstance API. Reserved Instances can't be moved between regions.

Why not use regional all the time?

If there's an outage in one AZ, ASGs Wil relaunch the lost capacity in other AZs. During that situation, a lot of customers will "compete" for capacity in other AZs. If you have reserved capacity, you don't have anything to worry about. If you don't, well, there's a chance you'll end up with less instances than desired. 

Only Convertible Reserved Instances allow for moving between instance families

## Spot Instances

One-Time Request: instance is terminated and data is lost

Request and Maintain: Instance can be configured to terminate, stop or hibernate until price point can be met again

## Dedicated Instances

Dedicated Instances are EC2 instances that run on hardware that's dedicated to a single customer. It incurs a premium price over regular instances

Dedicated Instances are available as on-demand, reserved and spot instances

Dedicated Hosts provide visibility and the option to control how you place your instances on a specific, physical server. Useful for softwares with hardware based licenses. Dedicated Hosts can't be used in auto scaling groups

## AWS KMS

KMS has account limit of 50,000 requests

## AWS Service Catalog

AWS Service Catalog are set of cloudformation templates that users can use based on their IAM permissions. The "products" are cloudformation templates. Users cannot use cloudformation directly, they can use products which uses cloudformation in backbone

AWS Service Catalog lets you centrally manage your cloud resources to achieve governance at scale of your infrastructure as code (IaC) templates, written in CloudFormation or Terraform

With AWS Service Catalog, you can meet your compliance requirements while making sure your customers can quickly deploy the cloud resources they need

## Service Control Policies (SCP)

The syntax for an SCP requires only one Statement element. You can have multiple objects within a single Statement element though

SCP has ultimate say whether a call goes through. Even if an IAM user is an admin but scp has no allow rule for the user, the user can't access the resource

It can also restrict root user and scps are deny by default unless explicitly allowed

SCPs do not affect any service-linked role. Service-linked roles enable other AWS services to integrate with AWS Organizations and can't be restricted by SCPs. the trust policy of a service-linked role cannot be modified

Service Control Policy (SCP) can restrict aws accounts from creating resources in restricted regions

SCP can restrict creation of untagged resources

## AWS Certificate Manager (ACM)

Having ACM provision and renew public ssl certificates for you is free of cost. ACM loads certs on below services:
* Load Balancers
* Cloudfront distributions
* APIs on API Gateways

ACM is a regional service and cannot copy certs between regions unless you're using it with cloudfront

A wildcard SSL certificate can only handle multiple sub-domains but not different domain names

You can upload all SSL certificates of different domains in the ALB using the console and bind multiple certificates to the same secure listener on your load balancer

ALB will automatically choose the optimal TLS certificate for each client using Server Name Indication (SNI)

SNI solves the problem of loading multiple SSL certs to serve multiple websites into one web server. It is newer protocol (requires client to specify hostname of target to initiate SSL handshake) and works with ALB, NLB and Cloudfront

## AWS Direct Connect

A common and cost effective way to provide a redundant link to AWS with Direct Connect is a VPN connection. In the event that the Direct Connect path fails at DC1, your on-prem router can redirect traffic over the VPN at DC2 via the DC1-DC2 link. Having dual Direct Connect links is definitely redundant but more expensive than a VPN.

Direct connect link --> Direct Connect Endpoint --(Private Virtual Interface)--> Direct connect Gateways --> Transit Gateways

Single direct connect gateway can attach upto 3 transit gateways. Transit gateways are region specific and can attach to vpcs in that region

Transit gateways can be peered with each other for cross region vpc peering

Direct Connect requires 802.1Q VLAN support

AWS supports IPv6 traffic over a virtual private gateway to an AWS Direct Connect connection

## AWS PrivateLink

AWS PrivateLink connects to endpoints within the same region as your vpc, Private link does not traverse through public internet

You can control API endpoints, sites and services reachable from your vpc

Can be used to connect to services of different aws accounts

Mainly used to connect to services outside of your vpc like API Gateway, Dynamodb, S3, lambda, etc with private endpoints or third party applications with endpoints/load balancers or service endpoints purchased from aws marketplace

Endpoint services owned by your organization in separate vpc can leverage privatelink

Can be used as a vpc peering alternative if we have endpoints available

Gateway Endpoints are used to send traffic to S3 and DynamoDB and do not use privatelink

AWS PrivateLink requires that the third party service provider have an AWS account

## AWS Snowball

With snowball devices, the staging workstation won't be able to mount the data warehouse directly

Snowball Edge's onboard S3-Compatible Endpoint makes for seamless transfer from a data warehouse S3 interface, which most of the major data warehouse vendors support

Hadoop data can be copied to the S3 endpoint after mounting it on a staging workstation

## Amazon Rekognition

Amazon Rekognition is image processing service that can extract metadata on objects in a photograph

Amazon Rekognition can store information about detected faces in server-side containers known as collections. You can use the facial information that’s stored in a collection to search for known faces in images, stored videos, and streaming videos. Amazon Rekognition supports the IndexFaces operation. You can use this operation to detect faces in an image and persist information about facial features that are detected in a collection.

## AWS Lightsail

AWS Lightsail is designed to be a very easy entry-level experience for those just starting out with virtual private servers. A WordPress site can be deployed with literally a single click and does not require AWS Console access or knowledge of EC2 or VPCs

## EC2

The two methods that AWS recommends if you lose a private key for an EC2 key pair are using Systems Manager Automation or using a secondary instance to edit the authorized_keys file

Instance Fleet (Mixed instance types and purchasing options) does not have Auto Scaling

EC2Rescue can help you diagnose and troubleshoot problems on Amazon EC2 Linux and Windows Server instances. You can run the tool manually or you can run the tool automatically by using Systems Manager Automation and the AWSSupport-ExecuteEC2Rescue document

## AWS Shield

AWS Shield and other counter-measure technologies work to protect all AWS customers from DDoS attacks. Unless AWS was aware of the test time and expected duration, its likely the traffic was blocked as suspicious. Despite being a permitted service, traffic suspected of being malicious will still be blocked.

Shield protects from layer 3 and 4 attacks, Shield advanced also protects from advanced application layer DDoS attacks

Shield gives Limited visibility on attacks, with shield advanced we get detailed logs on ddos attacks

## AWS Firewall Manager

Firewall Manager is a very effective way of managing WAF rules across many WAF instances and accounts. It does require that the accounts be linked as an AWS Organization.

## Network Load Balancer (NLB)

You cannot have more than one target group per listener on a Network Load Balancer. 

## AWS FSx

FSx is most commonly used for windows file service integration with AWS. It is used where EFS can't be used

variants of fsx are Netapp, Openzfs, windows file server(SMB based file shares highly compatible with windows), Lustre (High performance compute clusters with 1000 EC2s)

Amazon Fsx for windows can be mounted on linux instances

FSx for lustre has seamless integration with S3, can R/W S3 as a file system

FSx file system deployments

Scratch File System
* Temp Storage
* Data is not Replicated
* High Burst
Persistent File System
* Replicated multi-AZ
* Replace failed file within mins
		
FSx for Netapp OnTap supports compression, data deduplication

There is no option to Dynamically Allocate the file system size in Fsx

Workaround: Create an Amazon CloudWatch metric to monitor the FreeStorageCapacity of the file system. Write an AWS Lambda Function to increase the capacity of the Amazon FSx for Windows File Server file system using the update-file-system command. Utilize Amazon EventBridge to invoke this Lambda function when the metric threshold is reached

## Amazon Aurora

Aurora supports upto 15 Read Replicas and Aurora global supports upto 5 cross region read replicas (asynchronous sub second replication)

You cannot set Auto Scaling for the master database on Amazon Aurora. You need to Create an Aurora Replica and enable Aurora Auto Scaling for the replica.

## AWS Glue

Glue data quality provides recommendations and predefined rules to help you ensure the quality of your glue data

Using Glue data quality, you can setup alerts and data quality metrics to provide confidence in the data you are leveraging

Glue Data Catalog is a catalog of datasets which has a glue data crawler which will look into all databases or any JDBC compatible database and write metadata into data catalog

All services rely of glue data catalog to pull data like Athena, EMR, Redshift, etc

## Amazon Athena

Amazon Athena supports Apache Parquet, JSON and Apache ORC

Athena does not support XML

Athena Performance Improvements:
	* Use columner data for cost savings (Apache Parquet or ORC recommended)
	* Use Glue to convert data to Parquet or ORC
	* Compress data for smaller retrievals
	* Partition data in amazon S3 for easy querying on virtual columns
	* Use bigger files to minimize overhead (>128 MB), larger files are easier to scan and retrieve

Athena Federated Query allows you to query data from anywhere, not just S3. It can be relational, non relational, custom data sources, object, etc (AWS and on prem)

It uses data source connectors which is a lambda function to run federated queries (One Lambda Function per data source connector)

## AWS Global Accelerator 

AWS Global Accelerator is a faster way for users to get into AWS private network by bypassing long internet hops

It creates an entry point at edge location nearest to user and routes the user traffic through aws private network to the application

AWS Global Accelerator static entry points are protected by AWS shield by default

The Static IP addresses provided by global accelerator can route to endpoints in upto 10 different regions, picking closest available to the user (Multi region failover)

Mainly used to improve latency for global users

## Resource Access Manager (RAM)

Resource Access Manager is centralized resource sharing service which allows to share resources with principals accross many accounts

Share resources with individual accounts or OUs within your organizations (resources are still owned by the sharing account)

Resource sharing needs to be explictly enabled by organization for it to work

Most shared resources are regional and can only be accessed within same region

e.g. Share single VPC infra across multiple organizations instead of networking each vpc in each account together
	 share certificate authorities across accounts to reduct cost and complexity

## Secrets Manager 

IAM access permissions needs to be granted to the application to retrieve secret

AWS SDK has getSecretValue function to get secret from secrets manager as long as the application has permissions to access it

Secrets are encrypted at rest and in transit

## AWS Security Hub

Security Hub acts as a single pane view of prioritized security insights in case of multi account environment or complicated single account environment

Integrates with Inspector, GuardDuty, Firewall Manager and Macie to deliver security insights

Generates prioritized recommendations based on aws best practices

Can integrate with third party security services

## AWS Firewall Manager

Firewall manager is used to manage many firewalls across organization. Deploy security tools at scale, spanning many vpcs and accounts

Integrates with Security Groups, WAF, Shield and Network Firewall as well as third party tools

Can deliver logs and insights to security hub

Network Firewall does not have a security group

## AWS GuardDuty

GuardDuty can send events to amazon eventbridge and take automatic remidiation actions

Detecting threats with low operational overhead --> GuardDuty

GuardDuty Delegated Administrator is an account in organization which has all permissions to enable and manage guardduty across all accounts in organization. Can only be done using organization management account

## AWS Compute Optimizer

Compute optimizer is a ML tool which gives recommendations on compute resources

Increase cost efficiency and improve performance by right sizing over-provisioned/under-provisioned resources

Can be activated in one account or across organization

Gives recommendations on Lambda, EC2, ECS Fargate, EBS and Auto Scaling Groups

## Container Services

High Control and High Complexity to low control and low complexity
	EKS on Container Instances
		Requires customization to integrate with ML services like Polly, SageMaker, Batch, etc
		Adding load balancers to container instances is difficult and requires genralized abstractions
	ECS on Container Instances
		Natively Integrates with most aws services
		Seamlessly integrates with NLPs, ALBs
	ECS Fargate
	App Runner (Comes with lot of limitations)

ECS and EKS can run on AWS Wavelength, Fargate, EC2, Outposts and local zones

Fargate is a compute layer for containerized workloads, it handles scaling, security and server management. It integrates with Cloudwatch and container insights.

## App Runner

App Runner is designed exclusively for synchronous http applications which handles request/response type traffic. It supports public and private endpoints. i.e you can have internal or public facing application. It scales to zero when not used and hence is great for pocs or side projects. App Runner can get expensive at scale!

App Runner is a region scoped service so another app runner needs to be setup during cross region deployment

## AWS Wavelength

AWS Wavelength is 5G optimized edge compute solution

Wavelength is used to give ultra low latency to applications through 5G networks by deploying applications on 5G edge networks

The edge networks are wavelength zones and those can be linked with aws region

## AWS QuickSight

QuickSight is serverless pay per use service. Mainly used for Business Intelligence

It uses SPICE in memory cache and AutoGraph supplies best fit graphs for given set of data

Accessible from web browsers or mobile applications

Creates hybrid datasets from multiple sources

Can leverage ML for NLP queries and automated insights

Supports a number of third party data sources

Also supports aws services which are supported by aws glue

SPICE is an in-memory computation engine which works only with data imported into QuickSight

QuickSight has users and groups only available in QuickSight and this is not IAM

A Dashboard is a read-only snapshot of an analysis that you can share, it preserves configuration of analysis

A dashboard can be shared with users and groups in quicksight

## AWS OpenSearch

ElasticSearch is now OpenSearch: It is mostly a search engine/analytics tool and an open source alternative to AWS services. It is not serverless and runs inside a VPC

Sometimes referred to as ELK Stack

OpenSearch is an open source fork of ElasticSearch used for log search and analytics

Not serverless, needs to run on servers. Must specify instance types, multi-AZ, etc

Once OpenSearch has access to your logs, it can visualize, search, and analyze data in real time

Allows more advanced querying at cheaper cost than cloudwatch and also provides indexing capabilities

If we have Many accounts delivering messages, logs, configs or documents to a centralized account. Using opensearch on that account is a good idea

Opensearch can be used to create a search function for your application using AWS CDK

Opensearch can be used with QuickSight for analytics

Want to migrate kibana or elasticsearch, use opensearch

OpenSearch Service allows you to store and search the semi-structured JSON data. It also has tools to create dashboards and visualizations

## AWS Control Tower

AWS Control Tower uses cloudformation stacksets behind the scenes

AWS Control tower is an easy way to setup and govern compliant multi-account aws environment and it runs on top of aws organizations

It automatically sets up aws organizations to organize accounts and implement SCPs

AWS Control Tower can be used to set up and manage multiple AWS accounts. However, it will not automatically provision IAM permissions for all member accounts

AWS Control Tower - Account Factory
* Automates account provisioning and deployments. Enables to create pre-approved baselines and configs for aws accounts

AWS Control Tower - Guardrails
* Provides ongoing governance for your control tower environment
	Preventive - uses SCPs
	Detective - uses AWS Config

## IoT Landscape
Device Connection
* AWS IoT Core
* AWS IoT 1-Click
* AWS IoT Events
* AWS IoT Greengrass
* AWS IoT Things Graph
Device Management 
* AWS IoT Device Management
* AWS IoT Device Defender
Analytics and Visibility
* AWS IoT Analytics
* AWS IoT Sitewise

AWS IoT Core lets you connect billions of IoT devices and route trillions of messages to AWS services without managing infrastructure

AWS IoT Core often uses MQTT (Message queuing Telemetry Transport) Protocol to transfer messages between IoT devices

You can chose between these communication protocols. MQTT, HTTPS, MQTT over WSS, and LoRaWAN

Ingest/Publish messages to and from IoT devices

Enable connection to and from AWS Services like S3, Lambda, etc

AWS IoT Events triggers alerts when events occur

Monitor sensor data from your IoT device and trigger events

AWS IoT 1-click can directly trigger lambda with 1-click compatible IoT devices

Compatible devices are preprovisioned with certificates for secure access

Can be used to manage and group 1-click devices

AWS IoT Things Graphs Allows you to design low code logical workflows for your IoT devices

Similar to Step Functions for IoT devices

IoT device management is a device registry used to manage IoT devices at scale and can be used to send OTA updates to devices

IoT device defender uses ML for anomaly detection to publish alerts in response to device behavior

IoT Analytics is used to develop reports from time-series data

Aggregate messages from IoT devices at scale and store them in time-series format

Build reports based on standard SQL queries or ML analysis insights, can be paired with quicksight

IoT Sitewise is an edge software installed in edge datacenter (out of reach of aws network) to monitor IoT devices without internet

AWS IoT Greengrass is a client software which manages deployment of new or legacy apps across fleets using any language, packaging technology, or runtime

Manage and operate device fleets in the field locally or remotely using MQTT or other protocols

Collect, aggregate, filter, and send data locally. Manage and control what data goes to the cloud for optimized analytics and storage

Bring intelligence to edge devices, such as for anomaly detection in precision agriculture or powering autonomous devices

## AWS Budgets

Cost and Usage Reports can be used to generate CSV files to track costs by account, service or tags

With consolidated billing enabled in Organizations, track spending costs across your organization from the management account

Granularity can be adjusted to hourly, daily or monthly

Reports can be stored to S3 for further analysis by other services

Customized budget alerts can be created by triggering a sns topic which invokes a lambda fuction which in turn invokes another lambda function in account which has exceeded budget and shutdown the high cost resource

## ElastiCache

ElastiCache can be used between on prem and aws services to cache data for faster communication and delivery

Elasticache use cases:
* User Session Store
* Lazy Loading RDS queries

## AWS Single Sign-on (SSO) or IAM Identity Center

The use of AssumeRoleWithWebIdentity is only for Web Identity Federation (Facebook, Google, and other social logins). AWS does not recommend this anymore, Amazon Cognito is recommended

Web Identity Federation is used with public identity providers such as Facebook, Google, etc.

AWS SSO is new managed way of identity federation and recommended by aws

AWS SSO can have only one identity provider at a time. It can be SSO itself or Microsoft AD or External Identity Provider

AWS IAM Identity Center is successor to Single Sign-On, just a rename of service

AWS IAM Identity Center supports single sign-on to business applications through web browsers only

AWS IAM Identity Center supports only SAML 2.0–based applications

AWS IAM Identity Center uses permission sets to assign access

when using the AD connector for SSO, you cannot use both on-premises AD and third-party integrations at the same time

## Permission Sets

Permission sets are a collection of one or more IAM Policies

You don't neet to create any assumed roles, permission sets do that automatically behind the scenes

## Cloud HSM

Use cloud HSM if question asks FIPS 140-2 Level 3 validation. KMS is level

CloudHSM supports SSL acceleration where the SSL queries are offloaded to HSM and it does the computing instead of EC2 instances. Must setup a cryptographic user on cloudhsm device and make sure instances and use that user.

## Amazon Inspector

Amazon Inspector works on EC2 instances, images pushed to ECR and lambda functions. It also looks at network reachability. Reports to security hub

## AWS Logging

* Load balancer access logs => S3
* Cloudtrail Logs => S3 and Cloudwatch logs
* VPC Flow Logs => Firehose, Cloudwatch Logs, S3
* Route 53 Access Logs => Cloudwatch Logs
* S3 access logs => S3
* Cloudfront Access Logs => S3
* AWS Config => S3

## High Performing Cluster (HPC)

EC2 Enhanced Networking
* Elastic Network Adapter (ENA) upto 100 Gbps, lower latency, higher bandwidth and most recent
* Elastic Fabric Adapter (EFA) is improved ENA and only works for Linux. For High Performance Computing, tightly coupled workloads. It bypasses underlying linux OS to provide low latency and reliable transport

AWS ParallelCluster is Open Source Cluster Management tool to deploy HPC on AWS. Configurable with text files and automate creation of VPC, subnet, cluster types and instance types.

## AWS Serverless Application Model (AWS SAM)

The AWS Serverless Application Model (AWS SAM) is an extension of cloudformation

It is an open-source framework that you can use to build serverless applications on AWS

It consists of the AWS SAM template specification that you use to define your serverless applications, and the AWS SAM command line interface (AWS SAM CLI) that you use to build, test, and deploy your serverless applications

AWS Serverless Application Repository is just a managed repository for serverless applications. This solution is incomplete as you will need other AWS tools to build and deploy your application

SAM can leverage CodeDeploy and run lambda Function including traffic shifting feature

## AWS Outposts
	
AWS Outposts are server racks which aws sets up on prem for users

Outposts are setup and managed by aws for hybrid cloud users

We can start leveraging AWS services on prem and a lot of aws services run on outposts like EC2, EBS, S3, ECS, EKS, RDS, EMR

Need to setup S3 access point to leverage S3 on outposts

Can also use datasync to sync buckets on outposts and aws

## AWS Local Zones

AWS Local Zones are similar to availability zones located near to the customer. They are extension of an AWS Region for latency sensitive applications

## AWS DataSync
	
DataSync is replication of storages. on prem to cloud and vice versa

On prem devices need to install an agent, aws to aws replication doesn't need any agent

Replication tasks are not continuous, they can be scheduled hourly, daily, weekly

Has the ability to keep file permissions and metadata which includes SMB/NFS configuration

Want to install datasync but don't have capacity to do so? Use aws snowcone device, which comes with agent preinstalled. Run it on prem, it will pull your data, run agents and ship it back to aws to sync with storages

## Cloudfront Functions

Cloudfront functions are lightweight functions written in Javascript with sub ms startup times. Max execution time is <1ms. Lambda@Edge has 5s viewer triggered-30s origin triggered

Can do very quick time sensitive transformations. Cannot call external services from cloudfront, just transformations

Cloudfront functions are deployed at very edge (nearest to client), Lambda@Edge is deployed at regional edge

Cloudfront functions can be used with Lambda@Edge

Lambda@Edge can be used to route cloudfront caching to nearest origin and pull data in cache from nearest region

## Step Functions

Step Functions are used to build serverless workflow to orchestrate lambda functions

Synchronous invoking of lambda function, run a batch job, run an ECS Task and wait for it to complete, insert an item into DynamoDB, publish messages to SNS, SQS, Launch EMR, Glue, Sagemaker Jobs, launch another Step Function Workflow... 

AWS SDK Integrations with access over 200 aws services from State Machine, can be invoked by Lambda, API Gateway, CLI, EventBridge, CodePipeline and Step Functions itself

Max execution time is 1 year for standard workflow and 5 mins for express workflow. Standard Workflows were expensive and express workflow is the cheaper, faster variant

Express workflows can be synchronous and asynchronous
* Synchronous: Wait for function calls to complete and return the result
* Asynchronous: Doesn't wait for workflow to complete. It is useful for workflows that don't need an immediate response

Possibility to implement human approval feature

Chaining Lambda with Step Functions will introduce latency between API calls

Step Functions does not integrate natively with AWS Mechanical Turk, best way to do that us use SWF

Step Functions also support error handling. Use case: retry on failure

## Amazon Simple Queue Service (SQS)

SQS can handle message size of max 256KB, Kinesis data streams can do upto 1MB (Use pointer to S3 for large messages)

SQS is often used as a buffer to avoid throttling issue

SQS FIFO preserves the order of messages but have constraints of 300 messages/s without batching, 3000 /s with batching

When a consumer fails to process a message within the Visibility Timeout, the message goes back into the queue. We can set a threshold on how many times a message can get into the queue

After maximumReceives threshold is exceeded, the message goes to another queue called Dead Letter Queue (DLQ). DLQ are useful for debugging. DLQ of FIFO queue must also be FIFO queue and same goes for standard queue

DLQ has a Redrive to source feature where when our code is fixed, the messages in DLQ can be sent back to source SQS for processing

SQS cannot be used as direct input to step functions

## Simple Notification Service (SNS)

SNS also has FIFO Topic but subscribers for it can only be SQS FIFO queues

SNS also supports message filtering. It is a JSON policy for filtering messages and accordingly send relevant messages to relevant subscribers

SNS can be paired with SQS if we want same message to be pushed to multiple SQS queues. Make all SQS queues as subscriber for SNS topic and that message will be added to all SQS queues

Use Cases:
* If you want to send same S3 event to many SQS queues, use fan out pattern, subscribe that S3 event to an SNS Topic and publish it to SQS queues, lambda, etc at once for further processing

SNS also supports custom delivery policies for failure retries on custon endpoints and other customizations

After exhausting the delivery policy, messages that haven't been delivered are discarded unless you set a SNS - dead letter queue (DLQ)

DLQ are SQS/SQS FIFO queues attached to a subscription rather than a topic

SNS Mobile Push is used to send push notifs directly to mobile apps

## Managed Service for Apacke Kafka (MSK)

Managed Service for Apacke Kafka (MSK) is alternative to Kinesis. You can deploy your kafka cluster in your VPC with Multi Az upto 3 for HA

Data is stored in EBS volumes for as long as you want

Also has MSK Serverless, you run kafka on MSK without stressing about capacity, instances, etc

Kinesis has 1MB message limit, MSK defaults to 1MB but can be set to high, like 10MB

## Amazon Elastic Map Reduce (EMR)

EMR Supports auto scaling with cloudwatch

EMR has EMRFS, a native integration with S3 to store data. EMR will be faster on EBS (HDFS) but if you want something durable and multi AZ, S3 is an option

EMR can access dynamoDB using HIVE
* Master Node: Manage Health checks and coordinate
* Core Node: Run tasks and store data
* Task Node: Just to run tasks - usually Spot

## Lambda Functions

There are two types of concurrency available for lambda function:
	* Reserved concurrency – Reserved concurrency creates a pool of requests that can only be used by its function, and also prevents its function from using unreserved concurrency.
	* Provisioned concurrency – Provisioned concurrency initializes a requested number of execution environments so that they are prepared to respond to your function’s invocations.
 
These can help limit the number of concurrently invoked lambda functions thus avoiding overwhelmed consumer















SCPs don't grant any permissions, they are only used to restrict access. Use IAM Policies/Permissions to grant access.

Storage Gateway-Cached volumes can support volumes of 1,024TB in size, whereas Gateway-stored volume supports volumes of 512 TB size.

Server-side encryption with Amazon S3-managed encryption keys (SSE-S3) uses strong multi-factor encryption. Amazon S3 encrypts each object with a unique key. 
As an additional safeguard, it encrypts the key itself with a master key that it rotates regularly. Amazon S3 server-side encryption uses one of the strongest block ciphers available, 256-bit Advanced Encryption Standard (AES-256), to encrypt your data.

The signed cookies feature is primarily used if you want to provide access to multiple restricted files

NACLs cannot filter requests based on URLs.
A security group cannot filter requests based on URLs.
You can use a forward web proxy server in your VPC and manage outbound access using URL-based rules. Default routes are also removed.

Backup and restore (RPO in hours, RTO in 24 hours or less)
Pilot light (RPO in minutes, RTO in hours)
Warm standby (RPO in seconds, RTO in minutes)
Multi-region (multi-site) active-active (RPO near zero, RTO potentially zero)

Usually, mobile applications do not have complicated table relationships hence, it is recommended to use a NoSQL database like DynamoDB

You can’t directly configure a bucket ACL to allow access from Amazon CloudFront only. You will need an origin access identity (OAI) for this setup.
Create a special CloudFront user called an origin access identity (OAI) and associate it with your distribution. Configure the S3 bucket policy to only access from the OAI. 

AWS X-Ray is used to debug production and distributed applications such as those built using a microservices architecture.

Although CloudFront supports content uploads via POST, PUT, and other HTTP Methods, there is a limited connection timeout to the origin (60 seconds). If uploads take several minutes, the connection might get terminated. 
If you want to optimize performance when uploading large files to Amazon S3, it is recommended to use Amazon S3 Transfer Acceleration which can provide fast and secure transfers over long distances.

Elastic Network Interface can provide multiple IP address to instance.

When automatic failover occurs in Multi-AZ database, The CNAME record for your DB instance will be altered to point to the newly promoted standby

Running docker based containers on fargate is more expensive than serverless lambda functions

AWS::RDS::DBCluster has default delete policy of "Snapshot"

Cloudformation custom resource is created whenever you need to do something which is not supported by cloudformation. Custom Resource invokes lambda function in back end and that function can do jobs not supported by cloudformation.

If cloudformation resources are changed after they're deployed, we call it drift.
Cloudformation Drift is used to check if resources have drifted
Cloudformation can import resource and you don't need to recreate resource from cloudformation stack for it.

AWS Cloud Development Kit (CDK) lets you to define your cloud infrastructure using familiar languages like JavaScript, Python, Java, etc
The code is later compiled into CloudFormation Template (JSON/YAML)

Cloudmap is fully managed resource discovery service. Creates a map of backend services that your applications depend on. You register your application components, their locations, attributes and health status with AWS Cloud map and cloud map maps frontend with backend. Due to cloud map, New version application deployments don't require code changes

Storage Gateway Hardware Appliance can be used when on prem virtualization is not possible. Buy it from amazon.com and set it up in your infrastructure.
Storage Gateway does not directly work with glacier or glacier deep archive. Lifecycle policies can be setup on the bucket instead.

S3 can be used as a file system by using file gateway appliance in VPC and attaching it to EC2. The same gateway appliance can also be used as a read-only replica.

Snow Family Improving Performance
	perform multiple write operations at one time - from multiple terminals
	Transfer small files in batches - zip up small files till at least 1 MB to increase transfer speed
	Don't perform other operations on files during transfer
	Reduce local network use
	Eliminate unnecessary hops - direct connection preferred
Typical data transfer rate is between 25 MB/s and 40 MB/s. Use Amazon S3 Adapter for Snowball for faster speeds, between 250 MB/s and 400 MB/s

DMS can use Snowball Edge and S3 to speed up migration
	Use AWS Schema Conversion Tool to extract data locally and move it to edge device
	Ship the device back to AWS
	Edge device automatically loads its data into S3 bucket
	DMS can use S3 as a source to build a database

AWS Cloud Adoption Readiness Tool (CART) asks questions across six perspectives and generates a custom report of your level of migration readiness. Transforms idea of moving to cloud into a detailed plan that follows AWS best practices.

AWS Fault Injection Simulator (FIS) is a fully managed service for running fault injection experiments on aws workloads. Chaos Engineering - Stressing an application by creating disruptive events (Sudden increase in CPU or memory) to test functioning under load
Generate prebuilt templates to generate desired disruptions

AWS Migration Evaluator helps you build a data-driven business case for migration to AWS. Install agentless colector to conduct broad-based discovery. Takes snapshot of on-premises footprint, server dependencies, etc. Analyzes and then develops migration plan.

AWS Backup supports cross-region backups and cross account
Supports Storage Gateway (Volume Gateway)
Supports tag based backup policies
Transition backups to cold storage after specific period, specify backup retention period

Backup Vault Lock to enforce Write Once Read Many state for all the backups that you store in AWS Backup Vault. It cannot be deleted. Provides additional layer of defense for backups. Even root user cannot delete backups in Vault Lock

AWS Application Migration Service is same as CloudEndure Migration or Server Migration Service (SMS)

AWS Elastic Disaster Recovery is used to quickly and easily recover your physical, virtual and cloud-based servers into AWS. It has continuous block-level replication for servers

Comprehend - Amazon Comprehend uses natural-language processing (NLP) to help you understand the meaning and sentiment in your text.
Kendra - Intelligent search service out of Unstructured text (e.g search something from set of PDFs)
Textract - High powered OCR engine, picture to text
Forecast - Analyze time-series data with other variables to deliver highly accurate forecasts
Fraud Detector - Build fraud detection machine learning model which is highly customized based on your data
Transcribe - real-time transcription of Audio to text (CC) (Alexa)
Lex - Build conversational interfaces to understand intent and context of text. (Alexa) Build Chatbots
Polly - Converts text to natural speech (Alexa)
Rekognition - Image and video analysis to recognise objects, people, expressions, etc. Uses: Content Moderation using AI/ML, Hate Speech recognisiton, etc
SageMaker - Manage Labeling jobs for training Datasets using active learning and human labeling. Has Jupyter Notebook. Train and Tune Models. Package and deploy your machine learning models at scale.
Sagemaker NEO - Optimized Architecture Binary. Optimizes machine learning models to run on different CPU Architecture 
Translate - use deep learning to translate languages
Personalize - Recommendation engine as a service based on demographic and behavioral data

Amazon CodeGuru is an ML-Powered service for automated code reviews and application performance recommendations
	CodeGuru Reviewer: Automated code reviews, security vulnerabilities, hard to find bugs, etc. Supports Java and Python
	CodeGuru Profiler: Recommendations about application performance, helps understand runtime behavior of your application

Kinesis Video Streams: One video stream per device. source eg Cameras, smartphone, body camera, etc. Source are called producers, can use Kinesis Video Streams Producer Library.
Underlying data is stored in S3 but we don't have access to it and we cannot directly output stream data to S3. Need to build custom solution for it.
Consumed by EC2 instances for realtime analysis. Can leverage Kinesis Video Stream Parser Library to read streams.
Can have integration with Rekognition to extract metadata from video streams and send it to kinesis data streams

AWS Device Farm is application testing service for mobile and web applications. Test applications across real browsers and real mobile devices and are fully automated using framework. Automatically generates videos and logs for analysis. Can also remote into devices for debugging.

CORS is a browser based security which lets browser use headers for cross origin API calls. CORS is used in places where API calls are made and not where browser pages are hosted

Response code 429 is Quota Exceeded Error which means that client has done too many requests and has been throttled. This can be fixed by increasing transactions per second metric in API gateway to increase throttle limits.
API Gateway requests time out after 29 seconds which can lead to 504 error (Integration Failure)

AWS Service Quotas can be used to notify about resource utilization

Kinesis Data Streams will throw throughput exception if messages exceed limit of 1MB/s per shard

VPC Endpoint allows you to access an aws service privately within same VPC but it does not allow cross region access

ECR can have cross region replication similar to S3

Default cloudfront SSL certificate only works for (*.cloudfront.net) domains. ACM is preferred for custom domains

PowerUserAccess if you have users who perform application development tasks. This policy will enable them to create and configure resources and services that support AWS aware application development.
grants IAM permissions to create a service-linked role
It also grants Organizations permissions to view information about the user’s organization, including the master account email and organization limitations.

Rolling with additional batch is only applicable in Elastic Beanstalk
Rolling Splits the instances into batches and deploys to one batch at a time. 
Rolling with additional batch Splits the deployments into batches but for the first batch creates new EC2 instances instead of deploying on the existing EC2 instances.
Immutable environment updates ensure that configuration changes that require replacing instances are applied efficiently and safely
Canary deployments are preferred for lambda

Traffic Mirroring is an Amazon VPC feature that you can use to copy network traffic from an elastic network interface of Amazon EC2 instances. You can then send the traffic to out-of-band security and monitoring appliances

Redshift cluster snapshots can be easily copied by enabline cross-region snapshots
Set up a snapshot copy grant for a master key in the destination region if snapshots are encrypted by KMS



Tag Editor allows bulk tagging to easily tag your AWS resources

SQS can also be used to decouple application layer and RDS layer and store database writes. Lambda Function can then be implemented to pull from SQS and write to DB

## Instance meta-data and user-data locations
	http://169.254.169.254/latest/meta-data
	http://169.254.169.254/latest/user-data/

## API Gateway

API Gateway has AWS_IAM authorization feature so that only authorized IAM users can invoke API gate

## AWS License Manager

AWS License Manager is used to create customized licensing rules that emulate the terms of their licensing agreements, and then enforce these rules. It is not used for storing software licenses.

## Amazon Keyspaces

Amazon Keyspaces (for Apache Cassandra) is a scalable, highly available, and managed Apache Cassandra–compatible database service

## CI/CD Pipeline (CodePipeline)

Code(CodeCommit/Github) --> Build(CodeBuild/Jenkins CI) --> Test(CodeBuild/Jenkins CI) --> Deploy(CodeDeploy/Elastic Beanstalk) --> Provision(Elastic Beanstalk/Cloudformation)

CodeCommit push can trigger a lambda function which can do things like scan for leaked aws creds and disable accounts as a remedy

CodePipeline can have manual approval stage

Automatically build and store docker images using CodeBuild+ECR

CodePipeline can also do automated cloudformation deployments

CodePipeline also has Github integrations where github can trigger http webhook and codepipeline runs (Version 1)

Integration now has a new version where CodePipeline and Github are linked together with CodeStar Source Connection (Github App)
