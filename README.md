<br /><br />
<p align="center">
  <img width="230" src="assets/icon.png" />
</p>
<br /><br />

# grafana-cluster
> Scalable Grafana cluster using AWS Fargate.

[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat-square)](contributing.md)
[![CodeFactor](https://www.codefactor.io/repository/github/aws-blocks/grafana-cluster/badge)](https://www.codefactor.io/repository/github/aws-blocks/grafana-cluster)

Current version: **1.0.1**

Lead Maintainer: [Halim Qarroum](mailto:hqm.post@gmail.com)

## Table of content

 - [Installation](#install)
 - [Features](#features)
 - [Description](#description)
 - [Deployment](#deployment)
 - [Block Parameters](#block-parameters)
 - [Block Outputs](#block-outputs)
 - [CloudWatch Integration](#cloudwatch-integration)
 - [See also](#see-also)

## Install

In order to add this block, head to your project directory in your terminal and add it using NPM.

```bash
npm install @aws-blocks/grafana-cluster
```

The stack will be available into the `node_modules/@aws-blocks` directory.

## Features

 - Serverless, scalable deployment of a Grafana cluster on AWS.
 - Load balancer and alerting mechanisms built-in.
 - Provision Grafana properties and plugins from the stack parameters.
 - Non-intrusive and can be seamlessly integrated into your own architecture.

## Description

This repository features a CloudFormation stack which creates a scalable Grafana cluster using Docker containers on AWS Fargate. You can either deploy it as a standalone stack using the AWS console or the AWS CLI, or integrate it as a sub-stack as part of your own deployment.

This stack makes use of a Multi-AZ Aurora cluster as a database storage for Grafana, as well as an optional Redis cluster (enabled by default and the recommended method) to store sessions across multiple Grafana nodes. As such, this Grafana deployment aims to be highly-available as well as being auto-scalable given the load of the application.

## Deployment

1. Start by installing this stack locally using `npm`.

2. Head to the `node_modules/@aws-blocks/grafana-cluster` in your shell and package the stack using the `aws-cli` so that the sub-stacks are bundled with the cloudformation template provided in this repository.

```bash
aws cloudformation package --s3-bucket <an-s3-bucket-you-own> --template-file cloudformation.yml --output-template-file cfn.package.yml
```

3. Once the stack is packaged, you can open the AWS Cloudformation console in your browser and select the **Create Stack** button. Select then the **Upload a template to S3** option and upload the **cfn.package.yml.yml** file that has been created.

4. At this point you can go to your [AWS CloudFormation Console](https://console.aws.amazon.com/cloudformation/home) in your AWS account and deploy the `cfn.package.yml` file.

5. Configure the parameters to match your deployment requirements in the CloudFormation console when deploying your project. More information are available on the parameters made available by this stack in the below section.

<br />
<p align="center">
  <img src="assets/parameters.png" />
</p>
<br />

6. Once your stack parameters are set click **Next** and **Next** again. On the last page, make sure that you select the **I acknowledge that AWS CloudFormation might create IAM resources.** and the **I acknowledge that AWS CloudFormation might require the following capability: CAPABILITY_AUTO_EXPAND** options before deploying, and once it is selected you can continue with deploying the stack.

<br />
<p align="center">
  <img src="assets/grafana-dashboard.png" />
  <p align="center"><sub>An screenshot example of a Grafana dashboard.</sub></p>
</p>
<br />

## Block Parameters

Below is a description of the different parameters exposed by this Cloudformation stack. The parameters which do not have a default value must be explicitely specified.

Parameter | Description | Type | Default |
--------- | ----------- | ---- | --------
**Environment** | The environment name on which you would like to deploy the project. This identifier will be used to tag created resources. | String | development
**DatabaseInstanceType** | The database instance class to associate with the Aurora storage cluster. | String | db.r4.large
**DatabaseBackupRetentionPeriod** | The number of days for which automated backups are retained. | Number | 7
**EncryptionAtRest** | Indicates whether the Aurora database storage should be encrypted. | Boolean | false
**DatabaseSubnets** | The subnets to place database instances in, specify at least 2 subnets for the RDS database. | List of `AWS::EC2::Subnet::Id` | 
**DatabaseUser** | The database master account user name of the RDS database (this field cannot be equal to the `admin` identifier, as it is a reserved keyword). | String | 
**DatabasePassword** | The database master account password of the RDS database (choose a secure password with a length of at least 8 caracters). | String | 
**DatabaseName** | The name of the database to create and which will contain the data generated by the Grafana nodes. Choose a name of at least 5 caracters. | String |
**PreferredBackupWindow** | If automated backups are enabled (see the BackupRetentionPeriod property), the daily time range in UTC during which you want to create automated backups. | String | 01:00-02:30
**PreferredMaintenanceWindow** | The weekly time range (in UTC) during which system maintenance can occur. | String | mon:03:00-mon:04:00
**CloudWatchDashboard** | Whether to create a Cloudwatch monitoring dashboard associated with the created Grafana cluster. | Boolean | false
**SessionStorageType** | (Redis only) - The session storage type for Grafana (stroging Grafana sessions in a distributed Redis is highly recommended). | String | redis
**CacheNodeType** | (Redis only) - The cache node instance class to deploy in the cluster. | String | cache.t2.micro
**CacheClusterSubnets** | (Redis only) - The subnets to place cache nodes in, specify at least 2 subnets for the caching nodes. | List of `AWS::EC2::Subnet::Id` | 
**AutomaticFailoverEnabled** | (Redis only) - Indicates whether Multi-AZ is enabled. When Multi-AZ is enabled, a read-only replica is automatically promoted to a read-write primary cluster if the existing primary cluster fails. | Boolean | true
**NumNodeGroups** | (Redis only) - The number of node groups (shards) for this Redis replication group. | Number | 1
**ReplicasPerNodeGroup** | (Redis only) - The number of replica nodes in each node group (shard). | Number | 2
**AtRestEncryptionEnabled** | (Redis only) - Indicates whether to enable encryption at rest on the ElastiCache cluster. | Boolean | false
**TransitEncryptionEnabled** | (Redis only) - Indicates whether to enable in-transit encryption on the ElastiCache cluster. | Boolean | false
**RedisCloudWatchDashboard** | (Redis only) - Whether to create a Cloudwatch monitoring dashboard associated with the created ElastiCache cluster. | Boolean | false
**PublicSubnets** | The list of public subnets to associate with the internet facing load balancer, specify at least 2 public subnets. | List of `AWS::EC2::Subnet::Id` | 
**PrivateSubnets** | The list of private subnets to associate with the Grafana container deployments, specify at least 2 private subnets. | List of `AWS::EC2::Subnet::Id` |
**VpcId** | The VPC identifier to deploy the Grafana application in. | AWS::EC2::VPC::Id |
**SSLCertificateArn** | (Optional) - An ARN associated with a TLS certificate to use from Amazon Certificate Manager (ACM). | String |
**GrafanaAdminPassword** | The Grafana admin password to configure for the user 'admin' (must have at least 6 characters and at maximum 41 characters). | String | 
**GrafanaPlugins** | A comma-delimited list of Grafana plugins to provision on the Grafana hosts. | String | See template for default values.
**DefaultContainerCpu** | The amount of CPU to allocate to the containers (see https://aws.amazon.com/fargate/pricing/ for more information). | Number | 256
**DefaultContainerMemory** | The amount of memory to allocate to the containers (see https://aws.amazon.com/fargate/pricing/ for more information). | Number | 512
**DefaultServiceScaleEvaluationPeriods**  | The number of periods over which data is compared to the specified threshold. | Number | 2
**DefaultServiceCpuScaleOutThreshold** | Average CPU value to trigger auto scaling out. | Number | 50
**DefaultServiceCpuScaleInThreshold** | Average CPU value to trigger auto scaling in | Number | 25
**DefaultTaskMinContainerCount** | The minimum number of containers to run Grafana. | Number | 1
**DefaultTaskMaxContainerCount** | The maximum number of containers to run Grafana when auto scaling out. | Number | 5
**LoadBalancerAlarmEvaluationPeriods** | The number of periods over which data is compared to the specified threshold. | Number | 2
**LoadBalancerAlarmEvaluationPeriodSeconds** | The time over which the specified statistic is applied. Specify time in seconds, in multiples of 60. | Number | 300
**LoadBalancerLatencySeconds** | The load balancer latency threshold, expressed in seconds. | Number | 2

## Block Outputs

Below is a description of the output variables which are returned by this stack upon a successful deployment. You can use these output variables from a parent stack if you are using this block as a sub-stack.

Output variable | Description
--------------- | -----------
**Name** | Grafana Stack Name
**FargateEcsClusterName** | Fargate ECS cluster name
**FargateEcsClusterArn** | Fargate ECS cluster ARN
**GrafanaServiceArn** | Grafana service ARN
**GrafanaServiceName** | Grafana service name
**GrafanaServiceUrl** | Grafana application URL
**ApplicationLoadBalancerArn** | Load balancer ARN
**ApplicationLoadBalancerDnsName** | Load balancer DNS name
**ApplicationLoadBalancerName** | Load balancer name
**ApplicationLoadBalancerListenerArn** | Load balancer listener ARN
**LoadBalancerAlarmTopicArn** | Load balancer alarm topic ARN
**LoadBalancerAlarmTopicName** | Load balancer alarm topic name

## CloudWatch Integration

This implementation can optionally create a CloudWatch dashboard displaying the metrics associated with the Grafana cluster you have deployed. Below is an example of how such a dashboard looks like, and it is a great way to always have a nice view of how your cluster is performing.

<br />
<p align="center">
  <img src="assets/dashboard.png" />
</p>

## See also

 - The [Grafana documentation](http://docs.grafana.org/) on the official Grafana website.
 - The [Grafana Docker](http://docs.grafana.org/installation/docker/) documentation.
 - The [Grafana Plugins](https://grafana.com/plugins) documentation.
 
