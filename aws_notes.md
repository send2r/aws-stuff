# AWS Essentials
Since 2006 all on-premise work can be transferred to AWS. Learning structure
- AWS Global Infrastructure
- Networking
- Compute
- Storage
- Database
- Analytics
- Application Services
- Deployment & Management

## Avalability Zones, Regions & Edge locations
53 AZs, with 18 Geographic regions around globe

Choosing Regions : 
- Data regulatory consideration
- Proximity
- Cost
- Service availability

Availability Zones : different physical facility. Connected via highspeed network. Different natural fault lines. Cluster instances should be in different AZs in a region - enables high availability. 

Edge locations distributes and serves content from various servers across the globe. ( Eg: CloudFront, route53 )
Identity & Access Management _do not_ support __Regions__

## AWS Services ( in the order of first-steps)

### Security
__IAM__ : A root account is given while creating original AWS account. Further, users can be created. Can give _programatic access_ or _aws management console access_
An Amazon Resource Number is as `arn:aws:iam::297106433303:group/Workshop-Group` - `297106433303` is the 12 digit account Id given to the parent account. Group level policies can be set. Concept of Roles and Permissions. Hierarchy :

`AWS Account -- has -- Groups -- has -- Users`


**User** gets access key and secret key for API operations. MFA, password associated. 5000 per account. **Groups** can have policies. 100 Groups per account. **Roles** . **Policy**: `Actions` such as `ec2:StartInstances`, `Effect` such as `Allow` - `Condition` such as IP address based restrictions ( `IpAddress:Paws:SourceIp:23.23.234.2/20}`). Trust policy vs. permission policy. Is a free service.

### Networking
VLAN and VPC are synonymous. Generally VPC creation is done after creating users groups etc VPC is configured before launching new instances. Decide ACL, IP ranges etc. Within VPC, private IP address remains same. For internal communication we can use private IP address. Typical steps involved are: 
- Choose IP range
- Create subnet
- Create internet gateway
- Attach IGW to VPC
- Change route entries
- Set up security groups
Check : DirectConnect, Rout53, CloudFront

**VPC peering** is used for configuring communication between multiple VPCs. VPCs have to be in same region for this purpose. Within VPC sub-netting is done depending on purpose and other rules. VPC is a free service.

**API gateway** - publish, maintain and monitor service. A server less module. Can be a front-end to components like _lambda_, _EC2 components_ etc.

### EC2 
Basically VM instances in the cloud. With **resizable** compute capacity. **Reduces time required to obtain and boot** new service to minutes. All can be automated. Built using a base template called `AMI -- amazon machine image`. We can create a custom AMI. **Security Group** is a virtual firewall - lets you define Inbound and Outbound rules. That controls access. EBS - elastic block storage will be attached to the instance. tags provide information to the instance. Provides keypair ( pub key & priv key - cant lose) for access. All EBS volumes are in same AZ as the instance. 
One DR strategy is to take EBS snapshot and store in `s3`. Then attach it to a newly created EC2 instance. You can not attach the same EBS to same EC2 instances. EBS has a different lifecycle than the EC2 instance. You can assign static IP address to the EC2 instances. "termination protection" to avoid accidental terminations. We can create an AMI to save an instance template. Tags defined during creation help in management, billing and cost management. **Role vs access keys** - Role preferred as there is no need of propagating keys to all instances. **EBS** is tightly coupled to a given EC2 instance. For shared mount points between EC2 instances **EFS** Amazon Elastic file system has to be used. EC2 and EBS is **AZ** specific.

### Storage
**S3** simple service storage : No need of provisioning storage. Is a kind of key value storage. Versioning possible. What we store are "objects" in a "bucket". __Cross region replication__ is possible for objects stored in S3. Example: AMI images - for DR. Low cost redundancy. **policy** needs to be provided to set ex: Public anonymous access. Not associated with any ec2 instance so can be used independent of instances. **VPC endpoints** lets s3 access to happen from ec2 to node to s3 without going via public network.

**EFS** : Elastic file system. Provides standard file system semantics. Elastically grows and shrinks.`NFS V4` based. Available across AZs. Used in cases of *HA* and *DR*.

### Database

**Dynamo**: NoSQL DB. Dynamo DB is a *Managed Service*. Performance at scale. Each table **must** have a primary key. Concepts:
- Table
   - collection of items
   - except PK, schema-less

- Items
  - collection of attributes
  - any number of attributes
  - 400 KB limit on size <====

PK is also known as **Partition/HASH Key** as it is used for partitions. **Sort/Range key** defines how to order data with in a partition. Primary key becomes the input to hash function that locates partition where the key is residing.

 `ReadCapacityUnits` and `WriteCapacityUnits` are capacity parameters for tuning throughput.
_Read capacity unit_ up-to 4KB in size ( reads per second). Identify reads per second and configure it up front. Auto scaling lets us to increase the capacity automatically. Computation example: 
```
4 KB/s = 1 RCU
100 KB/s = 25 RCU
```
Write capacity unit also exist in similar fashion.
_Strongly consistent vs eventual consistency_ tune according to use. Duplicate insert on a given PK are `upserts`.

`ttl` can be set to specific timestamp for item expiry.

**Secondary index** can be created based on read patterns. It can be either **GSI**(Global secondary index) or **LSI**(Local secondary index). GSI lets you mention any key for a secondary index - that can be used as an additional partition key : entire data set gets duplicated for the index ( storage trade-off vs faster read ) `IndexName` is a parameter for query operation - to optimize the read. Decision is done at the app level rather than at the DB level. `projection-expression` can be used to select specific fields as part of a query. There is an expression language that lets you do this. 

### Management
__Auto scaling__: scale automatically, HA, Fault tolerance. Create instances in an _Auto scaling group_. Scaling plans: **Dynamic** based on cloudwatch alarm, or **Scheduled**. None of the state gets replicated during scaling. Auto-scaling configuration continues even after ec2 `terminate`. New instances will get spawned to meet the "minimum" parameter in scaling policy.

### Application Integration
**SNS** : Simple notification service. Sends individual to fan-out messages to large number of participants. It provides an sns _topic_ where notification can be sent, from there notification sent to bunch of subscribers ( such as email ). Used in _Cloud Watch_ integration. 

Subscription can be an:
- HTTP/HTTPS service
- Email/Email JSON
- Amazon SQS
- Application
- AWS Lambda
- SMS


**SQS** : Simple Queue service. Hosted queue for storing messages. HA, Failsafe distributed queue system. Messages are redundantly stored across SQS servers. This can be a "standard queue" or a "FIFO" queue. 
| Standard | FIFO
-----------|-----
|high throughput| FIFO
| at-least-once delivery | exactly once
| Best-effort ordering | limited throughput
||Not available in all regions






### Feature vs availability


| Feature  | Global | Regional | AZ | Edge |
|----------|----------|-------|----|------|
|  S3 Bucket||✔ |||
| EC2 Instance|||✔||
| EBS Volume|||✔||
| AMI| |✔|||
| EBS snapshot||✔|||
| ELB/AS/EIP||✔|||
| VPC/IGW/Route table||✔|||
| Subnet/NAT GW|||✔||
| IAM User | ✔ ||||
| Route53 | |||✔|

### Setting up for `node.js` development
- Install `node`
- Install `aws-sdk` using `npm`
- Configure aws keys using `aws configure`
- Include dependency `require('aws-sdk')`
- get _handler_ to service instance
- invoke API

Best practice to lock to a version ex: `var s3 = new AWS.S3({apiVersion: '2006-03-01'});`

### Serverless computing
No idle capacity required, no container configuration necessary. Cost reduction. This is achieved using **AWS Lambda**, which is part of **compute** services. Supports progarmming in java, Js, python, go. It is passive and stateless. Deployment and monitoring is managed by AWS Lambda. Role needs to be specified during creation to manage access to other services. Steps :
- Develop
- Upload code to AWS Lambda (bundle along with dependencies, deployment configs)
- Set up code to trigger(_handler_) from other AWS services such as HTTP endpoint, SQS, S3, Dynamo or in-app activity
- Lambda runs
- Pay for compute time used

Some Rules:
- Write in stateless manner
- No affinity to underlying resources
- Persist data in S3, Dynamo, RDS etc.

Creates and runs a container in the backend(docker). Lambda calls have a maximum time-out of 5 min.

### IOT
Platform for internet connected device communication with services, other devices etc. Provides gateway, rule engine and device offline services etc. Works mainly based on _topics_ and messages. **Device shadows** : A virtual concept for a real "thing". This lets us to manage an offline device and its state until it becomes available online again. **Jobs** lets you send and execute a remote operation on the device. Ex: rotate certs, remote trouble shooting, diagnostics, updates etc. A Job document describes.

### Cloudformation
Template service provided for defining system configuration.(Deployment).JSON based scripts - a free service. 
Flow : `Template(JSON| YML ) --> CloudFormation --> AWS services`
. Watch out for deletes and _changesets_ (update stack) `UPDATE STACK`, `DELETE STACK`. In-place update stack of an RDS with renaming the db will remove old DB.
### Cloudwatch
Monitoring for AWS components. Cloudwatch events and rules - `take snapshot of my ebs volume on so and so time`

## Kinesis
realtime data processing. Handles terabytes of data per hr. Concepts : data record, stream, shard, producer consumer.

