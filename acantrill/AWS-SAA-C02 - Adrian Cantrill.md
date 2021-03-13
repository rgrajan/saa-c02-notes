---
title: AWS-SAA-C02 - Adrian Cantrill
created: '2020-09-30T21:55:02.793Z'
modified: '2021-01-21T04:43:00.233Z'
---

# AWS-SAA-C02 - Adrian Cantrill

## What is Cloud computing
On-Demand Self-service - Provision and Terminate using UI/CLI without human interaction
Broad Network Access - Access service over any networks, on any devices using standard protocols and methods
Resource Pooling - Economies of Scale, Cheaper Service
Rapid Elasticity - Scan Up and Down automatically in response to system load
Measured Service - Usage is measured and Pay what you consume

## Port Nos
Windows RDP - 3389
Linux SSH - 22

## AWS Max Limits
S3 - 5TB
Soft Limit 100 buckets per account 
Hard Limit 1000 buckets per account

## IAM 
5000 IAM User per account
IAM User can be a member of 10 groups
300 groups per account, can be increased through support ticket
there is no default group attached to IAM user

## CloudTrail
Enabled by default - 90 days event history - NO S3
Trail can be configured in S3 and CloudWatch logs
By default Management events only
IAM, STS, CloudFront -> Global Service Events log to us-east-1, otherwise it is per region
This is not realtime - 15 minutes

## S3

Identity policy is for the same account
Bucket policy is for the resource, cross-account

S3 Object versioning once enabled, can't disable that. We can suspend and re-enable.
If you are working on object, it will be marker; if you are working on version; it is permanent delete.

### Multi-part upload
Min data size 100MB
10,000 max parts of 5 MB
if if any part fails, it can restart.

## KMS
KMS is compliance of FIPS 1402 (L2)
CMK can encrypt or decrypt to the maximum of 4KB of data
AWS managed key rotation is once in 3 years and mandatory
Customer managed CMKs once in year, optional

## S3 Encryption
SSE-S3 or AES256, if the object is uploaded with this permission then we can have access to all the objects within the bucket as long as we have access to S3 service

If the object is uploaded with SSE-KMS then we can access the objects only if we have the KMS permission

## Networking & VPC
Class A - 0.0.0.0 to 127.255.255.255
Class B - 128.0.0.0 to 191.255.255.255
Class C - 192.0.0.0 to 223.255.255.255

### Private Networks
10.0.0.0 to 10.255.255.255  (single class A)
172.16.0.0 to 172.31.255.255 (16 class B)
192.168.0.0 to 192.168.255.255 (256 class C)

### CIDR (Classless inter-domain routing)
10.0.0.0/8 means 10.ANYTHING - Class A million IPs
10.0.0.0/16 means 10.0.ANYTHING - Class B 65,536 IPs
10.0.0.0/24 means 10.0.0.0-255 - Class C - 256 IPs

### VPC considerations
Minimum /28 (16 IPs), maximum /16 (65536 IPs)
1 subnet - 1 AZ
* businesss needs
* avoid the ranges i can't use
* allocate the reminder based on business physical or logical layout
decide upon vpc & subnet structure from there

### Custom VPC
min /28 or max /16
can have optionally public IP
IPV6 /56 CIDR block

### DNS in a VPC
"Base IP +2" is the DNS address of the VPC
enableDnsHostnames - gives instances DNS names
enableDnsSupport - enables DNS resolution in VPC

### VPC Subnets
1 subnet = 1 AZ, 1AZ >= 0+n number of subnets
subnets can communicate with other subnets in VPC

Reserved address
* Network, first address 10.x.x.0
* Network + 1 - VPC router - 10.x.x.1
* Network + 2 - DNS- 10.x.x.2
* Network + 3 - future - 10.x.x.3
* Broadcast address 10.x.x.255 - last IP in the subnet

### Route table
Priority will be for specific routes, not generic like 0.0.0.0/0

### NACL
It is used or impact the communication between subnets. Within subnet, it will not impact anything
* only impacts data crossing subnet border
* Processed in order of rule number, stops if matched
* NACL is stateless
* can EXPLICITLY allow and DENY
* IPs/Networks, Ports & Protocols - no logical resources
* NACLs cannot be assigned to AWS resources
* One subnet = One NACL at a time

### Security Group
* Stateful
* there is no explicitly DENY, only ALLOW is available and implicit DENY
* SGs can filter based on AWS logical resources
* by itself
* NACL will be used when the AWS resource doesn't support SG, eg: NAT gateways

### NAT gateway
* runs from public subnet
* it uses Elastic IP
* AZ resilient service (HA in that AZ)
* for region resilience - NATGW in each AZ
* scales upto 45 Gbps
* standard hourly charge $0.04 per hr, data processing charge $0.04 per G
* only NACL, SG not supported
* IPV6 not supported, only for IPV4

### NAT instance
* disable Source/Destination checks

## EC2

* AZ Resilient

### Block storage
* volume to OS without structure, OS should format it to use
* Mountable & Bootable

### File storage
* file share
* Mountable and NOT bootable

### Object storage
* collection of objects
* not mountable, not bootable

### EBS
* Volume = one AZ, Resilient in AZ

### Volume Types
* General Purpose SSD - gp2
* Provisional IOPS SSD - io1
* Throughput optimized HDD - st1
* Cold HDD - sc1

io1 - 50:1 IOS to GiB ration vs 3:1 gp2

gp1 - only one can be attached to EC2, if you need another one, you have to detach and attach again
io1 - multiattach

* HDD cannot be used for boot volume
* volumes are in AZ, isolated in that AZ
* HA and resilient in that AZ
* one volume - one instance gp2, multi-attach for io1
* EBS MAX 80k IOPS (Instance), 64k (Vol) (io1)
* MAX 2375 MB/s (instance), 1000 MiB/s (vol)(io1)

## Instance Store
* physically connected to one EC2 host
* Highest storage performance in AWS
* included in instance price
* attached at launch
* lost on move, resize or hardware failure or restart
* TEMPORARY

D2 = 3.5GBps read and 3.1 GBps write
I3 = 16 GB/second of sequential throughput

## EBS VS Instance Store
* Highly available at AZ
* independent from EC2 instance
* clusters - multi-attache feature of io1
* region resilient backups, instance store cannot take backup
* 64k IOPS and 1000 MiB/s per volume

---
* include in the cost of the instance
* more than 80k IOPS & 2375 MB/s

### EBS Encryption
* default CMK - encrypt by default
* 1 unique DEK per volume
* snapshots & future volumes use the same DEK
* cannot change a volume to NOT be encrypted
* OS isn't aware of the encryption - no performance loss
* full disk encryption if there is no AES256, it can be done on top of encrypted or decrypted EBS volume

### ENI
* Secondary ENI + MAC = Licensing
* multi-homed subnets management & data
* different security groups - multiple interfaces
* OS doesn't see public IPv4
* IPv4  public IPs are dynamic.. stop & start will change, restart will not

### AMI
* AMI = one region; only works in that one region
* AMI Baking - creating an AMI from a configured instance + application
* An AMI can't be edited.. launch instance update configuration and create new AMI
* can be copied between region
* default is my account. 

###  ECS Concepts
* Container definition - Image & Ports
* Task Definition - Security(Task Role), Container(s), Resources
* Task Role - IAM Role which the Task assumes
* Service - How many copies, HA, restarts

### Placement Groups
Cluster
* Cluster placement group is the only way we can achieve 10Gbps single stream performance
* performance, fast speeds, low latency
Spread
* highest level of availability and resilient.
* 7 instances per AZ
* not supported dedicated hosts
Partitions
* 7 partitions per AZ
* we can control the instances into which partition
* or auto placed
* not supported on dedicated hosts
* parallel -topology aware, HDFS, cassandra

### Dedicated Hosts
* RHESL, SUSE LINUX AND WINDOWS AMIS are not supported
* Amazon RDS instances are not supported
* placement groups are not supported
* hosts can be shared with other ORG accounts

## RDS Multi-AZ
* no free tier
* standby can't be directly used
* 60-120 seconds faillover (highly available and not fault tolerence)
* same region only (other AZs in the VPC)
* Backups taken from Standby (remove performance impact)
* AZ outage, primary failure, manual failover, instance type change and software patching
* synchronous replication**** means multi-az

### RDS restore
* when backup is restored, it will be new instance
* snapshots are single in time
* automated 5 minutes transaction log
* restores aren't fast - need to consider for RTO

###
Asynchronous means Read Replica
Synchronouse means Multi-AZ

## EFS
* EFS is only for Linux
* General purpose and I/O performance modes
* bursting (gp2) and provisioned (io1) throughput
* Standard & Infrequent Access
* Lifecycle policies can be used with Classes

## ALB
* Targets are one compute resources
* rules are path based or host based
* support EC2, ECS, EKS, Lambda, HTTPS, HTTP/2 and websockets
* ALB can use SNI for multiple SSL certs - host based rules

## NLB
* If there is a requirement of whitelisting of IPs for load balancer then NLB should be used.
* 1 static IP per AZ
* SSL passthrough (end to end encryption?)
* can load balance http or https

## SQS
* standard = at-least-once, FIFO=exactly-once
* FIFO - 3000 per messages per second for batching, 300 mesages per second
* billed based on requests
* Polling - short(immediate) vs long (waitTimeSecond) upt p 20 sec
* Encryption at rest (KMS) & in-transit
* Queue policy



