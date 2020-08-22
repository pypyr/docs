---
title: pypyraws.steps.client
linktitle: client
date: 2020-08-19
description: Execute any low-level AWS client method.
card_extra_summary:
  heading: input context property
  details: "`awsClientIn` (dict)"
categories: [ steps ]
menu:
  docs:
    parent: aws-steps
    name: client
seo_article_headline: Use any low-level AWS client method without writing code.
seo_description: Interact with all the AWS service APIs without writing code. 
# social_og_description: 200 chars - if blank fall back to seo_description then description
# social_og_title: Assert dynamic value as expected in pipeline.
# topics: [ ]
---
# pypyraws.steps.client
## use any low-level aws service client
This step provides an easy way of getting at the low-level AWS api from the 
pypyr pipeline runner. So in short, pretty much anything you can do with the 
AWS api you got it as the Big O might have said.

This step lets you specify the service name and the service method you
want to execute dynamically. You can also control the service header
arguments and the method arguments themselves.

The arguments you pass to the service and its methods are exactly as
given by the AWS help documentation. So you do not have to learn yet
another configuration based abstraction on top of the AWS api that might
not even support all the methods you need.

You can actually pretty much just grab the json as written from the
excellent AWS help docs, paste it into some json that pypyr consumes and
tadaaa! Alternatively, grab the samples from the boto3 python
documentation to include in some yaml - the python dictionary structures
map to yaml without too much faff.

## supported aws services
`client` provides a low-level interface to AWS whose methods map close to
1:1 with the AWS REST service APIs. All service operations are supported
by clients.

pypyraws will automatically support new services AWS releases for the boto3 
client, in case the list above gets out of date. So while the document might not 
update, the code already will dynamically use new features and services on the 
boto3 client.

## input context
Require the following context items:

```yaml
awsClientIn:
  serviceName: 'aws service name here'
  methodName: 'execute this method of the aws service'
  clientArgs: # optional
    arg1Name: arg1Value
    arg2Name: arg2Value
  methodArgs: # optional
    arg1Name: arg1Value
    arg2Name: arg2Value
```

All inputs of `awsClientIn` support 
[text {substitution} formatting expressions]({{< ref "/docs/substitutions">}}).

### find the correct input arguments
- Go to the official aws cli [boto library documentation](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/index.html)
- Find the service you like. E.g EC2.
- Under the service, select `Client`
- There is a list of available methods.
    - For the sake of example, let's pick [create_route_table](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/ec2.html#EC2.Client.create_route_table) under EC2.
- The `methodArgs` you need are under 'Request Syntax', with full documentation 
  of what each input means.

```python
response = client.create_route_table(
    DryRun=True|False,
    VpcId='string',
    TagSpecifications=[
        {
            'ResourceType': 'client-vpn-endpoint'|'customer-gateway'|'dedicated-host'|'dhcp-options'|'elastic-ip'|'elastic-gpu'|'export-image-task'|'export-instance-task'|'fleet'|'fpga-image'|'host-reservation'|'image'|'import-image-task'|'import-snapshot-task'|'instance'|'internet-gateway'|'key-pair'|'launch-template'|'local-gateway-route-table-vpc-association'|'natgateway'|'network-acl'|'network-interface'|'placement-group'|'reserved-instances'|'route-table'|'security-group'|'snapshot'|'spot-fleet-request'|'spot-instances-request'|'subnet'|'traffic-mirror-filter'|'traffic-mirror-session'|'traffic-mirror-target'|'transit-gateway'|'transit-gateway-attachment'|'transit-gateway-multicast-domain'|'transit-gateway-route-table'|'volume'|'vpc'|'vpc-peering-connection'|'vpn-connection'|'vpn-gateway'|'vpc-flow-log',
            'Tags': [
                {
                    'Key': 'string',
                    'Value': 'string'
                },
            ]
        },
    ]
)
```

You can then map these inputs into your yaml pretty much via copy+paste:

```yaml
awsClientIn:
serviceName: ec2
  methodName: create_route_table
  methodArgs:
    DryRun: True
    VpcId: string id here
    TagSpecifications: 
      - ResourceType: client-vpn-endpoint
        Tags:
          - 'Key': 'string'
            'Value': 'string'
```


## output context
After this step completes the full response is available to subsequent
steps in the pypyr context in the `awsClientOut` key.

The AWS documentation you used above to find the correct input arguments fully 
documents the response structure for whichever method you are calling.

## example
Here is some sample yaml of using the pypyr-aws plug-in `client` step to upload 
a file to s3:

```yaml
# ./pipelines/go-go-s3.yaml
context_parser: pypyr.parser.keyvaluepairs
steps:
  - name: pypyraws.steps.client
    description: upload a file to s3
    in:
      awsClientIn:
        serviceName: s3
        methodName: upload_file
        methodArgs:
          Filename: ./testfiles/arb.txt
          Bucket: '{bucket}'
          Key: arb.txt
```

This will uploaded `arb.txt` to the bucket you specify from the cli input 
argument:

```bash
$ pypyr go-go-s3 "bucket=myuniquebucketname"
```

See a worked example for 
[pypyr aws s3](https://github.com/pypyr/pypyr-example/blob/master/pipelines/aws-s3.yaml).

## full list of supported aws services
`client` can run any method on any of the following aws low-level client 
services.

With the speed AWS introduces new features and services it's pretty unlikely 
I'll get round to updating the list each and every time. This doesn't really 
matter, the pypyraws plug-in will be able to use the latest functionality that 
Amazon publishes to boto3 automatically.

- AccessAnalyzer
- ACM
- ACMPCA
- AlexaForBusiness
- Amplify
- APIGateway
- ApiGatewayManagementApi
- ApiGatewayV2
- AppConfig
- ApplicationAutoScaling
- ApplicationInsights
- AppMesh
- AppStream
- AppSync
- Athena
- AutoScaling
- AutoScalingPlans
- Backup
- Batch
- Braket
- Budgets
- CostExplorer
- Chime
- Cloud9
- CloudDirectory
- CloudFormation
- Event
- Stack
- StackResource
- StackResourceSummary
- CloudFront
- Examples
- CloudHSM
- CloudHSMV2
- CloudSearch
- CloudSearchDomain
- Client
- CloudTrail
- CloudWatch
- CodeArtifact
- CodeBuild
- CodeCommit
- CodeDeploy
- CodeGuruReviewer
- CodeGuruProfiler
- CodePipeline
- CodeStar
- CodeStarconnections
- CodeStarNotifications
- CognitoIdentity
- CognitoIdentityProvider
- CognitoSync
- Client
- Comprehend
- ComprehendMedical
- ComputeOptimizer
- ConfigService
- Connect
- ConnectParticipant
- CostandUsageReportService
- DataExchange
- DataPipeline
- DataSync
- DAX
- Detective
- DeviceFarm
- DirectConnect
- ApplicationDiscoveryService
- DLM
- DatabaseMigrationService
- DocDB
- DirectoryService
- DynamoDB
- DynamoDBStreams
- EBS
- EC2
- EC2InstanceConnect
- ECR
- ECS
- EFS
- EKS
- ElasticInference
- ElastiCache
- ElasticBeanstalk
- ElasticTranscoder
- ElasticLoadBalancing
- ElasticLoadBalancingv2
- EMR
- ElasticsearchService
- EventBridge
- Firehose
- FMS
- ForecastService
- ForecastQueryService
- FraudDetector
- FSx
- GameLift
- Glacier
- GlobalAccelerator
- Glue
- Greengrass
- GroundStation
- GuardDuty
- Health
- Honeycode
- IAM
- IdentityStore
- imagebuilder
- ImportExport
- Inspector
- IoT
- IoTDataPlane
- IoTJobsDataPlane
- IoT1ClickDevicesService
- IoT1ClickProjects
- IoTAnalytics
- IoTEvents
- IoTEventsData
- IoTSecureTunneling
- IoTSiteWise
- IoTThingsGraph
- IVS
- Kafka
- kendra
- Kinesis
- KinesisVideoArchivedMedia
- KinesisVideoMedia
- KinesisVideoSignalingChannels
- KinesisAnalytics
- KinesisAnalyticsV2
- KinesisVideo
- KMS
- LakeFormation
- Lambda
- LexModelBuildingService
- LexRuntimeService
- LicenseManager
- Lightsail
- CloudWatchLogs
- MachineLearning
- Macie
- Macie2
- ManagedBlockchain
- MarketplaceCatalog
- MarketplaceEntitlementService
- MarketplaceCommerceAnalytics
- MediaConnect
- MediaConvert
- MediaLive
- MediaPackage
- MediaPackageVod
- MediaStore
- MediaStoreData
- MediaTailor
- MarketplaceMetering
- MigrationHub
- MigrationHubConfig
- Mobile
- MQ
- MTurk
- Neptune
- NetworkManager
- OpsWorks
- Personalize
- PersonalizeEvents
- PersonalizeRuntime
- PI
- Pinpoint
- PinpointEmail
- PinpointSMSVoice
- Polly
- Pricing
- QLDB
- QLDBSession
- QuickSight
- RAM
- RDS
- RDSDataService
- Redshift
- Rekognition
- ResourceGroups
- ResourceGroupsTaggingAPI
- RoboMaker
- Route53
- Route53Domains
- Route53Resolver
- S3
- S3Control
- SageMaker
- AugmentedAIRuntime
- SageMakerRuntime
- SavingsPlans
- Schemas
- SimpleDB
- SecretsManager
- SecurityHub
- ServerlessApplicationRepository
- ServiceQuotas
- ServiceCatalog
- ServiceDiscovery
- SES
- SESV2
- Shield
- signer
- SMS
- PinpointSMSVoice
- Snowball
- SNS
- SQS
- SSM
- SSO
- SSOOIDC
- SFN
- StorageGateway
- STS
- Support
- SWF
- Synthetics
- Textract
- TranscribeService
- Transfer
- Translate
- WAF
- WAFRegional
- WAFV2
- WorkDocs
- WorkLink
- WorkMail
- WorkMailMessageFlow
- WorkSpaces
- XRay