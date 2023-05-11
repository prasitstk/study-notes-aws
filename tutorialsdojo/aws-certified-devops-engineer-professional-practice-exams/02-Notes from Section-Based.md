# Incident and Event Response

`CodeDeploy` can be integrated with `CloudWatch Alarms`

- You can use `CloudWatch Alarms` to track `metrics` on your new deployment and you can set thresholds for those metrics in your` Auto Scaling groups` being managed by `CodeDeploy`.

- For example, Associate an AWS `CloudWatch Alarm` to your `deployment group` that can send a notification to an `AWS SNS topic` when threshold for `5xx is reached on CloudWatch`.





`CodeDeploy` has the option to `automatically roll back` a deployment when a deployment fails

- You will also have the option to `automatically roll back` a deployment when a deployment fails or when a CloudWatch alarm is activated. CodeDeploy will `redeploy the last known working version of the application` when it rolls back.

- If rollbacks are configured, you can configure a CloudWatch alarm that triggers a rollback when the validation test in your Lambda function fails.





What you can do with `Trusted Advisor`

- Use `CloudWatch Events` to monitor `Trusted Advisor checks` and set a trigger to send an email using SNS to notify you about the results of the check.

- ** Use `Amazon CloudWatch` to create `alarms` on `Trusted Advisor metrics` in order to detect the `load balancers with low utilization`. Specify `an SNS topic` for notification.

- Write `a Lambda function` that `runs daily` to refresh `AWS Trusted Advisor via API` and then publish a message to `an SNS Topic` to notify the subscribers based on the results.

- Write `a Lambda function` that `runs daily` to refresh `AWS Trusted Advisor changes via API` and send results to `CloudWatch Logs`. Create `a CloudWatch Log metric` and have it send` an alarm notification` when it is triggered.





You can’t directly use `Step Functions` in conjunction with `AWS Config`.




The `custom remediation actions` in `AWS Config` should be used in conjunction with `AWS Systems Manager Automation` documents, NOT a `Lambda Function`.

- Ex. Set up `an AWS Config rule` with `a configuration change trigger` that will detect any changes in the S3 bucket configuration and which will also invoke `an AWS Systems Manager Automation document` with `a Lambda function` that will revert any changes.




You CANNOT add a local secondary index to an already existing DynamoDB table.
- You need to set up a new DynamoDB table with a Local Secondary Index instead.






---

# Policies and Standards Automation

`CloudWatch Event bus` is primarily used to accept events from `AWS services`, `other AWS accounts`, and `PutEvents API calls`.





Example of `AWS Config Managed Rules`

- The `s3-bucket-public-read-prohibited` and `s3-bucket-public-write-prohibited` are AWS Config managed rules that check if your Amazon S3 buckets do not allow `public read access and write access`. These rules check the Block Public Access settings, the bucket policy, and the bucket access control list (ACL).

- you can use the `approved-amis-by-id` AWS manage rule which checks whether running instances are using `specified AMIs`.

- You can use the `cloudtrail-enabled` AWS Config managed rule with `a periodic interval` of 1 hour to evaluate whether your AWS account enabled the AWS CloudTrail.
  - NOTE: the `cloudtrail-enabled` AWS Config managed rule is only available for `the periodic trigger type` and `NOT Configuration changes`.

- You can use AWS Config to ensure that all of their `EBS volumes` in all of their AWS accounts (with `AWS Config aggregators`) are `encrypted`.

NOTE: `AWS Config` can also record all of the `changes to patch for OS` and association compliance statuses.




In CloudFront, you can:
- Secure end-to-end connections to the origin servers in your CloudFront distribution by using `HTTPS` and `field-level encryption`. 

In your CloudFront origin, you can:
- Set up your origin to add `a Cache-Control max-age directive` to your objects. Apply the longest practical value for `max-age` to `increase your cache hit ratio`.





The `AWSServiceRoleForOrganizations` service-linked role is primarily used to `only allow AWS Organizations to create service-linked roles for other AWS services`. 
- This service-linked role is present in all organizations and not just in a specific OU.




You can’t deploy `an application` to `your on-premises servers` using `Elastic Beanstalk`. 
- This is `only applicable` to your `Amazon EC2 instances`.







---

# High Availability, Fault Tolerance, and Disaster Recovery

NO `Canary deployment` configuration in `Elastic Beanstalk`.



You can ONLY integrate a `Network Load Balancer` to your `Amazon API Gateway`.




`Network Load Balancer` does not support weighted target groups, unlike the `Application Load Balancer`.




NO `Canary routing` option in an `Application Load Balancer`.




In `API Gateway`, you create `a canary release deployment` when deploying the API with canary settings as an additional input to the deployment creation operation.




a `Multi-AZ RDS database` spans to several `Availability Zones` within `a single Region` ONLY, and NOT to an `entirely new region`.



`Amazon RDS Event Notification`

- `Amazon RDS` uses the `Amazon Simple Notification Service (Amazon SNS)` to provide notification when an Amazon RDS event occurs. 

- You can subscribe to an event category for a DB instance, DB cluster, DB cluster snapshot, DB parameter group, or DB security group.

- For example, if you subscribe to the Backup category for a given DB instance, you are notified whenever a backup-related event occurs that affects the DB instance. If you subscribe to a configuration change category for a DB security group, you are notified when the DB security group is changed. You also receive notification when an event notification subscription changes.

- For Amazon Aurora, events occur at both the DB cluster and the DB instance level, so you can receive events if you subscribe to an Aurora DB cluster or an Aurora DB instance.




Remember that when you `modify the database engine for your DB instance in a Multi-AZ deployment` configuration, Amazon RDS upgrades `both the primary and secondary DB instances at the same time` which means that the `RDS shuts down the whole database`. 

- You should create `a Read Replica` first, which you could have used as `a backup instance` in the event of update failures. 



---

# SDLC Automation


`AWSCodeCommitPowerUser` – Allows users access to all of the functionality of CodeCommit and repository-related resources, 
- ** except it does not allow them to `delete CodeCommit repositories` or `create or delete repository-related resources` in other AWS services, such as Amazon CloudWatch Events. 
- It is recommended to apply this policy to most users.



Problem: To collects all of the application and server logs from EC2 instances effectively before the Auto scaling group terminates those instances.

Solution: 
- Delay the termination of unhealthy Amazon EC2 instances by adding `a lifecycle hook to your Auto Scaling group` to move instances in the `Terminating` state to the `Terminating:Wait` state. 
- Set up `a CloudWatch Events rule` for the `EC2 Instance-terminate Lifecycle Action Auto Scaling Event` with an associated `AWS Systems Manager Automation document`. 
- Trigger `the CloudWatch agent` to push `the application logs` and then resume `the instance termination` once all the logs are sent to `CloudWatch Logs`.





You can use `CloudWatch Events` to run `Amazon ECS tasks` directly when certain AWS events occur.
- No need to invoke `Lambda function` to call the `ECS task`.

- Ex. You can set up `a CloudWatch Events rule` that runs `an Amazon ECS task` whenever a file is uploaded to a certain Amazon S3 bucket using the Amazon S3 PUT operation.




You can use the console, AWS CLI, or AWS CloudFormation to add `cross-region actions` in `CodePipeline pipelines`. 

- If you use the console to create a pipeline or cross-region actions, `default artifact buckets` are configured by CodePipeline `in the Regions where you have actions`. 

- When you use the AWS CLI, AWS CloudFormation, or an SDK to create a pipeline or cross-region actions, you provide `the artifact bucket for each Region where you have actions`. 

  - You must create `the artifact bucket` and `encryption key` in `the same AWS Region` as the cross-region action and in `the same account` as your pipeline.






You can use `CloudWatch Alarms` to track `metrics on your new deployment` and you can set thresholds for those metrics in your `Auto Scaling groups` being managed by `CodeDeploy`. 

This can invoke an action if the metric you are tracking crosses the threshold for a defined period of time.

* You will also have `the option to automatically roll back a deployment` when a deployment fails or when a CloudWatch alarm is activated. `CodeDeploy` will redeploy `the last known working version` of the application when it rolls back.



A `Blue/green deployment strategy` in your `Elastic Beanstalk environment` requires that your environment runs `independently of your production database`. This means that you have to decouple your RDS database from your environment. 

If your Elastic Beanstalk environment has an attached Amazon RDS DB instance, the data will be lost if you terminate the original (blue) environment.




---

# Monitoring and Logging

You can’t configure an `S3 bucket` to directly send `access logs` to `a CloudWatch log group`.




You must use `AWS CloudTrail first` to set up `a trail` configured to receive `the object-level events` of `your S3 bucket`.




`AWS Trusted Advisor` is integrated with the `Amazon CloudWatch Events` and `Amazon CloudWatch services`. 
- You can use `Amazon CloudWatch Events` to detect and react to `changes in the status of Trusted Advisor checks`.
- And you can use `Amazon CloudWatch to create alarms` on `Trusted Advisor metrics` for check status changes, resource status changes, and service limit utilization




You can ONLY create `a Metric filter` for `CloudWatch log groups`. 




`AWS Health events` are NOT automatically sent to `CloudWatch Log groups`; thus, you have to `manually set up a data stream` for AWS Health.






`CloudWatch Logs subscription` CANNOT be directly integrated with `an AWS Step Functions` application.




`Amazon CloudWatch Alarms` can only send notifications to `SNS` and `not SQS`.




Your `CodeCommit repository` and `AWS SNS topic` MUST be on `the same region` for the `triggers` to work.





Kinesis Data Streams (Sub-accounts) >> Kinesis Data Streams (Audit account)

Kinesis Firehouse, CloudWatch Logs >> Amazon ES

Kinesis Data Streams, S3, DynamoDB >> * Lambda Function >> Amazon ES





You can ONLY configure `Amazon CloudWatch Events`, `Amazon SNS`, or `Amazon SQS` as notification targets for `the lifecycle hook of an Auto Scaling group`.

You CANNOT use `AWS Lambda` as a notification target for `the lifecycle hook of an Auto Scaling group`. 






`Amazon Macie` can also `monitor and detect usage patterns` on `your S3 data`, not only recognize sensitive data such as personally identifiable information (PII) or intellectual property, assigns a business value, and provides visibility into where this data is stored and how it is being used in your organization.

NOTE: `Amazon Macie` is available to protect data stored in `Amazon S3` ONLY.




The `cloudtrail-enabled` AWS Config managed rule is only available for `the periodic trigger type` and `NOT Configuration changes`.






You can create `an Amazon CloudWatch alarm` that monitors `an Amazon EC2 instance` and automatically recovers the instance if it becomes `impaired due to an underlying hardware failure` or `a problem that requires AWS involvement to repair`. 

When the `StatusCheckFailed_System` alarm is triggered, and the recover action is initiated, you will be notified by the Amazon SNS topic that you selected when you created the alarm and associated the recover action. 

During instance recovery, the instance is migrated during an instance reboot, and any data that is in-memory is lost. When the process is complete, information is published to the SNS topic you’ve configured for the alarm. Anyone who is subscribed to this SNS topic will receive an email notification that includes the status of the recovery attempt and any further instructions. You will notice an instance reboot on the recovered instance.

You can configure a CloudWatch alarm to automatically recover impaired EC2 instances and notify you through Amazon SNS. 
- ** However, the `SNS notification` by itself doesn’t include `the results of the automatic recovery action`. 

You must also configure `an Amazon CloudWatch Events rule` to monitor `AWS Personal Health Dashboard (AWS Health) events` for your instance. Then, you are notified of `the results of automatic recovery actions for an instance`.

There is no built-in instance recovery failure feature (After trying auto healing above) for Amazon EC2. You have to use a combination of CloudWatch and a Lambda function to automatically recover the  EC2 instance from failure.



---

# Configuration Management and Infrastructure as Code

You can use AWS `Systems Manager Automation` (`AWS-UpdateLinuxAmi` and `AWS-UpdateWindowsAmi` documents) to `preconfigure the AMI` by `installing all of the required applications and software dependencies`.




`Automation` offers one-click automations for simplifying complex tasks such as creating golden Amazon Machines Images (AMIs), and recovering unreachable EC2 instances. Here are some examples:

- Use the `AWS-UpdateLinuxAmi` and `AWS-UpdateWindowsAmi` documents to create golden AMIs from a source AMI. You can run custom scripts before and after updates are applied. You can also include or exclude specific packages from being installed.

- Use the `AWSSupport-ExecuteEC2Rescue` document to recover impaired instances. An instance can become unreachable for a variety of reasons, including network misconfigurations, RDP issues, or firewall settings. Troubleshooting and regaining access to the instance previously required dozens of manual steps before you could regain access. The `AWSSupport-ExecuteEC2Rescue` document lets you regain access by specifying an instance ID and clicking a button.




You can simply configure `a scheduled action` for the `Auto Scaling group` to adjust `the minimum number of the available EC2 instances` without using CloudWatch Events or a Lambda function.









---
