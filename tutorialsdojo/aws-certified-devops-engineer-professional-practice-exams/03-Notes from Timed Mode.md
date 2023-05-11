# Timed Mode Diagnostic Test

Plugins of AWS Systems Manager documents

- The `aws:runDocument` plugin `runs SSM documents` stored in Systems Manager or on a local share.
  - You can use this plugin with the `aws:downloadContent` plugin to `download an SSM document` from a remote location to a local share, and then run it.

- The `aws:downloadContent` plugin `downloads SSM documents and scripts` from `remote locations` including GitHub repositories (public and private), Amazon S3 buckets, Systems Manager.

- The `aws:configurePackage` plugin just `installs/uninstalls a Distributor package`.

- The `aws:softwareInventory` plugin is simply used for `gathering metadata` about applications, files, and configurations `on your managed instances`. 
  - The `files` section is an optional parameter that collects file metadata (e.g. file name, creation date, last modified).




NOTEs about SQS
- The `visibility timeout` just `prevents` consumers (AWS Lambda) from `processing the same order within a given period of time`. 

- The `redrive policy` just specifies how `failed messages` from `a source queue` are `moved to a dead letter queue`.

- The `ApproximateAgeOfOldestMessage` measures `the number of seconds since the creation of the oldest message in the queue`. 
  - This SQS metric is effective because `if it goes up, it means that messages are not being processed quickly enough`. 
  - It can also show you the messages that your consumers can’t handle and are stuck in your queue.
  - The `ApproximateAgeOfOldestMessage metric` can give you an idea that the `reserved concurrency` of `the Lambda function` is `not enough` to handle the volume of the orders which causes the delay. 





CloudFormation :: `cross-stack references` :: Export resources from one stack to be imported by `Fn::ImportValue` function to another stack

- When you organize your AWS resources based on lifecycle and ownership, you might want to `build a stack that uses resources that are in another stack`. 

- Use `cross-stack references` to `export` resources from a stack so that other stacks can use them. 

- Stacks can use `the exported resources` by calling them using the `Fn::ImportValue` function.

- For example, you might have a network stack that includes a VPC, a security group, and a subnet. You want all public web applications to use these resources. By exporting the resources, you allow all stacks with public web applications to use them.




Problem: the requirement to set up an email notification system for `particular EKS components-related log events`. The team is planning to use Amazon SNS for notification and AWS Lambda for log event evaluation.

Solution:
- Publish the `EKS components logs` to `Amazon CloudWatch Logs`. 
- Configure `a subscription filter` for each component log and select `a Lambda function` as the subscription feed destination.





Problem:  If a created bucket allows public access or has an incorrect policy, an email notification should be sent to the admin. Moreover, the incorrect policy should be automatically replaced by the proper policy.

Solution:
- Use `AWS Config rule` to check for `noncompliant bucket policy` and have it invoke a Lambda function when an S3 bucket is created. 
- Use `the Lambda function` to automatically `set the proper policy` and `trigger an SNS topic` to alert the admin.





Problem: There are three requirements in the scenario:
1. Restrict the operation of AWS resources to `specific regions` within the Asia Pacific.
2. Create a solution that sends `an alert if an activity occurs outside` the Asia Pacific.
3. The solution should automatically extend its functionality to `new regions`.

Solution:

- Attach `a service control policy` to the organization root that `allows` operations from `approved regions` and `denies` access to `non-global services` on regions outside the Asia Pacific.

- Enable `AWS CloudTrail` to deliver logs from all regions to `AWS CloudWatch Logs`. 
  - Then, create a metric filter that sends an alert whenever an activity in non-Asia Pacific regions occurs.





`AWS Security Hub` is used to provide `a comprehensive view of security alerts and security posture` across your AWS accounts. 

- With Security Hub, you have `a single place that aggregates, organizes, and prioritizes your security alerts, or findings`, from multiple AWS services, such as Amazon GuardDuty, Amazon Inspector, Amazon Macie, AWS Identity and Access Management (IAM) Access Analyzer, AWS Systems Manager, and AWS Firewall Manager, as well as from AWS Partner Network (APN) solutions. 




Problem:
- A security team manually controls `the rotation of AWS KMS customer-managed keys (CMK)` within their company’s AWS account. 
- The regulatory compliance requires keys to be rotated within 80 to 90 days. 
- ** The team `wants to be alerted` when keys are more than 90 days old.

Solution:
- * Create `an AWS Config custom rule` to check for CMKs that have not been rotated for more than 90 days. 
- Send notification to `an SNS topic` once a CMK has been identified.




NOTE: the `AWS KMS Console` does NOT have the feature to send a notification to an SNS topic if a CMK has not been rotated for more than 90 days




NOTE: `Checking of CMK rotation` is NOT included in the` AWS Trusted Advisor security checklist`.



Problem: Auto-deploy updated `Docker image` pushed to a repository to `AWS Fargate`

- In this scenario, we can `store the Docker images` to `Amazon ECR` and use `AWS CodeCodeploy` to automate the container deployment to `AWS Fargate`. 

* `AWS CodePipeline` is the glue that ties these two services together. 
  - ** With `CodePipeline`, you can trigger a deployment in `CodeDeploy` for every change made to a container in the Amazon ECR. 
  
- These three services are all fully managed by AWS so the requirement for a less operational overhead is ensured.





Problem: A company wants to employ a new standard that requires the creation of resources through AWS CloudFormation. 
- Due to this, IT administrators need to make sure that `users can only deploy stacks from pre-approved CloudFormation templates`. 
- They also need to implement `a monitoring solution that automatically detects resources that drift from the expected configuration`.

Solution:

- Create `an AWS Config rule` that evaluates whether `a CloudFormation stack` has drifted from the expected configuration.
  - You can use the `cloudformation-stack-drift-detection-check` managed rule in `AWS Config` for the automated monitoring solution. 
  - This Config rule checks if `the actual configuration of a CloudFormation stack` differs, or has drifted, from `the expected configuration`.

- Use `AWS Service Catalog` with `launch constraint` (NOT `template constraint`) to authorize users to deploy CloudFormation stacks.
  - ** `The template constraint` just limits the options that are available to end-users when they launch a product. 
    - It works by `narrowing the allowable values for parameters` that are defined in the product’s underlying AWS CloudFormation template.



NOTE: There is no `the CloudFormation drift detection feature` to detect whether a CloudFormation stack has drifted from the expected configuration. 
- this is still a manual process
- You can, however, automate this process by running `a scheduled Lambda function` that calls the `DescribeStackDriftDetectionStatus` API operation.


---


# Time mode 1

To keep the application resources to be at full capacity during deployment by using a new group of instances, the MOST cost-effective deployment set up = `immutable updates`
- NOTE: although using the `blue/green deployment` configuration is an ideal option, if you keep the old environment running as a backup, it is not recommended since it entails a significant cost.



By default, `AWS Config` will `NOT automatically remediate` the accounts that `disabled its CloudTrail`. You `must manually set this up` using `a CloudWatch Events rule` and `a custom Lambda function` that calls the `StartLogging API` to enable CloudTrail back again.


the `cloudtrail-enabled` AWS Config managed rule is ONLY available for the `periodic trigger` type and `NOT Configuration changes`.



CodePipeline :: AWS CloudFormation actions :: Parameter overrides

- In a `CodePipeline` stage, you can specify `parameter overrides` for `AWS CloudFormation actions`. `Parameter overrides` let you specify `template parameter values` that override values in a template configuration file. AWS CloudFormation provides functions to help you specify dynamic values (values that are unknown until the pipeline runs).



`CloudWatch Alarms` can’t receive direct test results from `AWS Lambda`.

You can use `CloudWatch Alarms` to track metrics on your new deployment and you can set thresholds for those metrics in your Auto Scaling groups being managed by `CodeDeploy`.

You will also have the `option to automatically roll back a deployment` when a deployment fails or when a CloudWatch alarm is activated. `CodeDeploy` will redeploy `the last known working version of the application` when it rolls back.




NOTE: CloudFormation :: a `cfn-init` helper script is primarily used to `fetch metadata`, `install packages` and `start/stop services` to `your EC2 instances` that are already running.





`Custom resources` enable you to write `custom provisioning logic` in templates that `AWS CloudFormation` runs anytime you create, update (if you changed the custom resource), or delete stacks. 
- For example, you might want to include resources that aren’t available as AWS CloudFormation resource types.

Use the `AWS::CloudFormation::CustomResource` or alternatively, the `Custom::<User-Defined Resource Name>` resource type to define custom resources in your templates. 
- `Custom resources` require one property: the `service token`, which specifies `where AWS CloudFormation sends requests to`.

** When you associate `a Lambda function` with a custom resource,` the function is invoked whenever the custom resource is created, updated, or deleted`. 
- `AWS CloudFormation` calls `a Lambda API` to invoke the function and to pass all the request data (such as the request type and resource properties) to the function. 
- ** The power and customizability of Lambda functions in combination with AWS CloudFormation enable a wide range of scenarios, such as 
- `dynamically looking up AMI IDs during stack creation`, or implementing and 
- using `utility functions`, such as `string reversal functions`.

Ex. 
Problem: Normally, you might map AMI IDs to specific instance types and regions. To update the IDs, you manually change them in each of your templates. 

Solution: By using `custom resources and AWS Lambda (Lambda)`, you can create a function that gets the `IDs of the latest AMIs` for the region and instance type that you’re using so that you `don’t have to maintain mappings`.





Problem: Implement a solution that automatically detects and reacts to changes in the state of their deployments in `AWS CodeDeploy`. 
- Any changes must be `rolled back automatically` if the deployment process fails, 
- and `a notification must be sent` to the DevOps Team’s Slack channel for easy monitoring.

Solution: 
- Set up `a CloudWatch Events rule` to monitor `AWS CodeDeploy` operations with `a Lambda function as a target`. 
- Configure the rule to send out a message to the DevOps Team’s Slack Channel in the event that the deployment fails. 
- ** Configure `AWS CodeDeploy` to use the `Roll back when a deployment fails` setting.

Explanation:

You can `monitor CodeDeploy deployments` using the following CloudWatch tools: 
- `Amazon CloudWatch Events`, 
- `CloudWatch alarms`, and 
- `Amazon CloudWatch Logs`.

Reviewing the logs created by the CodeDeploy agent and deployments can help you troubleshoot the causes of deployment failures. As an alternative to reviewing CodeDeploy logs on one instance at a time, you can use `CloudWatch Logs` to monitor all logs in a central location.

You can use `Amazon CloudWatch Events` to detect and react to `changes in the state of an instance or a deployment (an “event”)` in your CodeDeploy operations. Then, based on rules you create, CloudWatch Events will invoke one or more target actions when a deployment or instance enters the state you specify in a rule. Depending on the type of state change, you might want to send notifications, capture state information, take corrective action, initiate events, or take other actions.

You can select the following types of `targets` when using `CloudWatch Events` as part of your CodeDeploy operations:
– AWS Lambda functions
– Kinesis streams
– Amazon SQS queues
– ** Built-in targets (CloudWatch alarm actions)
– Amazon SNS topics

The following are some use cases:
– ** Use `a Lambda function` to pass `a notification` to a Slack channel whenever deployments fail.
– Push data about deployments or instances to a Kinesis stream to support comprehensive, real-time status monitoring.
– **Use `CloudWatch alarm actions` to automatically `stop, terminate, reboot, or recover Amazon EC2 instances` when a deployment or instance event you specify occurs.





NOTE: You have to install the `Amazon Inspector agent` first to `the EC2 instance` before you can run the security assessments.





Problem: Implement `an automated daily check of each golden AMI` they own to monitor the latest Common Vulnerabilities and Exposures (CVE) using `Amazon Inspector`.

Solution:
- Use `AWS Step Functions` to `launch an Amazon EC2 instance for each operating system` from the golden AMI, install `the Amazon Inspector agent`, and `add a custom tag` for tracking. 

- Configure `the Step Functions` to trigger `the Amazon Inspector assessment` for all instances with the custom tag you added right after the EC2 instances have booted up. 

- Trigger the Step Functions `every day` using `an Amazon CloudWatch Events rule`.




NOTE: `AWS Trusted Advisor` doesn’t provide any information regarding `AWS-initiated RDS events`. 
- You should use the `AWS Health API` instead. 




NOTE: The `standby instance` of an `Amazon RDS Multi-AZ database` can only be placed in `the same AWS region` where the `primary instance` is hosted. 
- Thus, you CANNOT failover to the standby instance as your replacement to your primary instance in `another region`. 





Problem: Which of the following solutions should the DevOps engineer implement which will improve the RTO and RPO of the website for the cross-region failover?

Solution:
- Use `Step Functions` with `2 Lambda functions` that call the `RDS API` to `create a snapshot of the database`, `create a cross-region snapshot copy`, and `restore the database instance from a snapshot` in the backup region. 

- Use `CloudWatch Events` to trigger the function to take a database snapshot `every hour`. 

- Set up `an SNS topic` that will receive published messages from `AWS Health API`, `RDS availability` and `other events` that will trigger `the Lambda function` to create `a cross-region snapshot copy`. 

- During failover, Configure `the Lambda function` to `restore the database` from a snapshot in the backup region.





`CloudWatch Logs subscription`

You can use subscriptions to get access to `a real-time feed of log events` from `CloudWatch Logs` and have it delivered to other services such as a Amazon Kinesis stream, Amazon Kinesis Data Firehose stream, or AWS Lambda for custom processing, analysis, or loading to other systems.

`A subscription filter` defines the `filter pattern` to use for `filtering which log events get delivered` to your AWS resource, as well as information about where to send matching log events to.

`CloudWatch Logs` also produces `CloudWatch metrics` about the forwarding of log events to subscriptions.



** NOTE: `a CloudWatch Logs subscription` CANNOT be directly integrated with `an AWS Step Functions application`.




Problem: Implement a solution that will `automatically terminate any instance` in production which was `manually logged into via SSH`. All of the EC2 instances that are being used by the application already have an Amazon CloudWatch Logs agent installed.

Solution:
- Set up `a CloudWatch Logs subscription` with `an AWS Lambda function` which is configured to add a `FOR_DELETION` tag to the `Amazon EC2 instance` that `produced the SSH login event`. 

- Run `another Lambda function` every day using the` CloudWatch Events rule` to `terminate all EC2 instances` with the custom tag for deletion.






Problem: Upgrade the RDS instance to the latest `major` version (NOT minor) of MySQL database in a Multi-AZ deployments configuration. It is of utmost importance to ensure minimal downtime when doing the upgrade to avoid any business disruption.

Solution:
- In the `AWS::RDS::DBInstance` resource type in the `CloudFormation template`, update the `EngineVersion` property to the latest MySQL database version. 
- ** Create `a second application stack` and launch `a new Read Replica` with the same properties as the primary database instance that will be upgraded, which you could have used as a backup instance in the event of update failures. 
  - NOTE: If you trigger the Update Stack operation first before creating a Read Replica, you may face downtime.
- Finally, perform an `Update Stack operation` in CloudFormation.


NOTE: When you modify the `database engine` for your DB instance in `a Multi-AZ deployment`, Amazon RDS upgrades `both the primary and secondary DB instances at the same time`. 
- In this case, the database engine for the entire Multi-AZ deployment is shut down during the upgrade.
- ** This is the reason why we need `a new Read Replica` during upgrading.


NOTE: there is NO such `DBEngineVersion` property.


NOTE: By default, minor engine upgrades are applied automatically. For updating major version, update the `EngineVersion` property instead.





** NOTE: `The built-in Trusted Advisor notification feature` to automatically receive notification `emails` which include the summary of savings estimates along with Trusted Advisor check results  will be sent on `a weekly basis only`.





Problem: You want to be notified for AWS Trusted Advisor recommendation as frequently as possible.

Solution: 
- Write `a Lambda function` that runs` daily to refresh AWS Trusted Advisor via API` and then publish a message to `an SNS Topi`c to notify the subscribers based on the results.

- Write `a Lambda function` that runs `daily to refresh AWS Trusted Advisor changes via API` and send results to `CloudWatch Logs`. Create a `CloudWatch Log metric` and have it send an alarm notification when it is triggered.

- Use `CloudWatch Events` to monitor `Trusted Advisor checks` and set a trigger to send `an email using SNS` to notify you about the results of the check.





NOTE: `Detailed Monitoring feature` sends the metric data for your `EC2 instance` to CloudWatch in `1-minute periods`.

- NOTE: This level of frequency is already available for your `load balancers`. 
  - If there are requests flowing through the `load balancer`, `Elastic Load Balancing` measures and sends its metrics in `60-second intervals`.



NOTE: In `ECS`, the `awslogs` log driver passes `logs` from `Docker` to `CloudWatch Logs`.




Problem: 
- There are several occasions when some instances are automatically terminated after failing the HTTPS health checks in the ALB that purges all the logs stored in the instance. 
- To improve system monitoring, a DevOps Engineer must implement a solution that collects all of the application and server logs effectively. 
- The Operations team should be able to perform a root cause analysis based on the logs, even if the Auto Scaling group immediately terminated the instance.

Solution:
- Delay the termination of unhealthy Amazon EC2 instances by adding `a lifecycle hook to your Auto Scaling group` to move instances in the `Terminating` state to the `Terminating:Wait state`. 
- Set up `a CloudWatch Events rule` for the `EC2 Instance-terminate Lifecycle Action Auto Scaling Event` with an associated `AWS Systems Manager Automation document`. 
- Trigger `the CloudWatch agent` to push the application logs and then resume the instance termination once all the logs are sent to CloudWatch Logs.

NOTE: the `Pending:Wait` state refers to the `scale-out action` in `Amazon EC2 Auto Scaling` and not for scale-in or for terminating the instances.

NOTE: The `EC2 Instance Terminate Successful` indicates that the ASG `has terminated` an instance.
- So, you cannot collect logs from instances after this event.



ECS
- To have your service use a newly `updated Docker image with the same tag` as in the existing task definition (for example, my_image:latest) or keep the current settings for your service, select `Force new deployment`. 
- If the `ECS task` is still running `an old image` then it is possible that the ECS agent is not running properly.
  - So, `Restart the ECS agent`




 In `CodeDeploy`, during each deployment lifecycle event, hook scripts can only access the following predefined environment variables: APPLICATION_NAME, DEPLOYMENT_ID, DEPLOYMENT_GROUP_NAME, DEPLOYMENT_GROUP_ID and LIFECYCLE_EVENT. 



NOTE: You can only integrate `a Network Load Balancer` to your `Amazon API Gateway`.



NOTE: You CANNOT use `CloudFront` to `adjust the weight of the incoming traffic` to your application. You should use `Route 53` instead.



NOTE: `CodeDeployDefault.LambdaCanary10Percent30Minutes` is only applicable for `Lambda functions`.



NOTE: In CloudFormation template, you can only set the `DeletionPolicy` to either `Retain` or `Delete` for `an Amazon S3 resource`.
- ** In addition, the CloudFormation deletion will still `fail` as long as `the S3 bucket is not empty`, even if the `DeletionPolicy` attribute is already set to `Delete`.
- ** `ForceDelete` is NOT a valid value for the `deletion policy` attribute.





NOTE: The `Network Load Balancer` does NOT support `weighted target groups`, unlike the `Application Load Balancer`.




NOTE: NO `Canary routing` option in `an Application Load Balancer`.

NOTE: ** In `API Gateway`, you create `a canary release deployment` when deploying the API with canary settings as an additional input to the deployment creation operation.

NOTE: ** In `Lambda function`, you can do `a canary deployment` by using routing configuration on an `alias` to send `a portion of traffic` to `a second function version`. 
- For example, you can reduce the risk of deploying a new version by configuring the alias to send most of the traffic to the existing version, and only a small percentage of traffic to the new version. 
- You can point an alias to `a maximum of two Lambda function versions`.




In `AWS CloudTrail`, enable the `log file integrity feature` on the trail that will `automatically generate a digest file for every log file` that CloudTrail delivers. 
- Verify the `integrity of the delivered CloudTrail files` using the `generated digest files`.




`Amazon Kinesis Adapter`

Using the `Amazon Kinesis Adapter` is the recommended way to `consume streams from Amazon DynamoDB`.
- For `real-time processing`, it is better than `Lambda functions` which are reading from the `DynamoDB Streams` of the table




`Amazon Kinesis Data Analytics` is the easiest way to `transform and analyze streaming data in real time` using `Apache Flink`, an open-source framework and engine for processing data streams.




NOTE: The `AWS_RISK_CREDENTIALS_EXPOSED` CloudWatch event is only applicable from an `aws.health` event source and NOT from an `AWS Personal Health Dashboard` rule; that is, you have to use the `AWS Health API` instead of the `AWS Personal Health Dashboard` to generates an `AWS_RISK_CREDENTIALS_EXPOSED` CloudWatch Event. In response to this event, you can 
- set up an automated workflow deletes the exposed IAM Access Key, 
- summarizes the recent API activity for the exposed key, and 
- sends the summary message to an Amazon Simple Notification Service (SNS) Topic to notify the subscribers which are all orchestrated by an AWS Step Functions state machine.




To specify how` AWS CloudFormation` handles `replacement updates` for `an Auto Scaling group`, use the `AutoScalingReplacingUpdate` policy. 
- This policy enables you to specify whether AWS CloudFormation `replaces an Auto Scaling group with a new one` or `replaces only the instances` in the Auto Scaling group.
  - During replacement, AWS CloudFormation retains the old group until it finishes creating the new one. If the update fails, AWS CloudFormation can roll back to the old Auto Scaling group and delete the new Auto Scaling group. While AWS CloudFormation creates the new group, it doesn’t detach or attach any instances. 
  - After successfully creating the new Auto Scaling group, AWS CloudFormation deletes the old Auto Scaling group during the cleanup process.

- When you set the `WillReplace` parameter, remember to specify a matching `CreationPolicy`. If the minimum number of instances (specified by the MinSuccessfulInstancesPercent property) don’t signal success within the Timeout period (specified in the CreationPolicy policy), the replacement update fails and AWS CloudFormation rolls back to the old Auto Scaling group.




NOTE: `AWS Config` CANNOT detect `the utilization of AWS resources`. You have to use `AWS Trusted Advisor` instead.





Problem: To speed up the deployment process of the independent Lambda functions.

Solution:
- Execute actions for each Lambda function `in parallel` by setting up a configuration that specifies `the same runOrder value` in `CodePipeline`.

NOTE: Upgrade the compute type of the build environment in `CodeBuild` pipeline with a higher memory, vCPUs, and disk space may help `speed up the build time`, it will only improve the performance of CodeBuild and `not the entire pipeline`. 
- A better solution is to `run the tasks in parallel` in `CodePipeline`.



NOTE: There is no way to directly download the `AmazonLinux2.iso` for Amazon Linux 2. 
- but you can get the Amazon Linux 2 image for the specific virtualization platform of your choice. If you are using VMware, you can download the ESX image *.ova and for VirtualBox, you’ll get the *.vdi image file. What you should do first is to prepare the seed.iso boot image then connect it to the VM of your choice on first boot.






`All customers` can use `the Personal Health Dashboard (PHD)`, powered by the `AWS Health API`. 
- The dashboard requires no setup, and it’s ready to use for authenticated AWS users. 

** Additionally, `AWS Support customers` who have `a Business or Enterprise support plan` can use the `AWS Health API` to `integrate` with in-house and third-party systems.

You can use `Amazon CloudWatch Events` to detect and react to changes in the status of `AWS Personal Health Dashboard (AWS Health) events`. Then, based on the rules that you create, CloudWatch Events invokes one or more target actions when an event matches the values that you specify in a rule.

Only those `AWS Personal Health Dashboard (AWS Health) events` that are `specific to your AWS account and resources` are published to CloudWatch Events.
- In contrast, `Service Health Dashboard` events provide information about the regional availability of a service and are `not specific to AWS accounts`, so they are not published to CloudWatch Events. 
  - These event types have the word `“operational”` in the title in `the Personal Health Dashboard`; for example, `“SWF operational issue”`.





Ex. Tracking the `AWS-initiated activities` to your resources by:
- Use a combination of `AWS Personal Health Dashboard` and `Amazon CloudWatch Events` to track the `AWS-initiated activities` to your resources. 
- Create an event using CloudWatch Events which can invoke an AWS Lambda function to send notifications to the company's Slack channel.

NOTE: `AWS Config` is NOT capable of tracking `any AWS-initiated changes or maintenance` in your AWS resources.

NOTE: `CloudTrail` simply tracks all of the `API events of your account only` but NOT the `AWS-initiated events or maintenance`.




`Systems Manager Automation` simplifies common maintenance and deployment tasks of `Amazon EC2 instances` and `other AWS resources`. 

`Automation` offers one-click automations for simplifying complex tasks such as `creating golden Amazon Machines Images (AMIs)`, and `recovering unreachable EC2 instances`. Here are some examples:

- Use the `AWS-UpdateLinuxAmi` and `AWS-UpdateWindowsAmi` documents to create `golden AMIs` from a `source AMI`. 
  - You can run `custom scripts` before and after updates are applied. 
  - You can also include or exclude `specific packages` from being installed.

- Use the `AWSSupport-ExecuteEC2Rescue` document to recover impaired instances. 
  - An instance can become unreachable for a variety of reasons, including network misconfigurations, RDP issues, or firewall settings. 
  - Troubleshooting and regaining access to the instance previously required dozens of manual steps before you could regain access. 
  - The `AWSSupport-ExecuteEC2Rescue` document lets you `regain access` by specifying `an instance ID` and clicking a button.





Problem: They instructed their DevOps team to configure `Amazon Route 5`3 to automatically route to an alternate endpoint when their primary application stack in us-west-1 region experiences an outage or degradation of service. [`active-passive` failover]

Solution:
- Use `a Failover routing policy` configuration. 
  - Set up `alias records` in Route 53 that route traffic to AWS resources. 
  - Set the `Evaluate Target Health option` to `Yes`, then create all of the required non-alias records.

- Set up `health checks` in Route 53 for `non-alias records` to `each service endpoint`. 
  - ** Configure the `network access control list` and the `route table` to allow `Route 53` to send requests to the endpoints specified in the health checks.





NOTE: In RDS, Create `Read Replicas` in `the new region` and configure the new application stack to point to the local Amazon RDS database instance. >> So, you can use `Read Replicas` for DR in another region.

NOTE: In RDS, you cannot deploy `the standby database instance` in `the new AWS region`.





You can use `AWS Config` to record configuration changes for `Dedicated Hosts`, and instances that are launched, stopped, or terminated on them. 
- You can then use the information captured by `AWS Config` as a data source for `license reporting`.
- Set up `a custom AWS Config rule` that triggers `a Lambda function` by using the `config-rule-change-triggered` blueprint. 
- Customize the predefined evaluation logic to verify host placement to return a `NON_COMPLIANT` result whenever the EC2 instance is `not running on a Dedicated Host`. 
- Use the `AWS Config report` to address noncompliant Amazon EC2 instances.



NOTE: `CloudWatch Alarms` can’t receive direct test results from `AWS Lambda`.
- If you want to do this, `store test logs on CloudWatch logs` and have `CloudWatch Events` monitor those logs.


NOTE: It is `tedious` to automatically perform `the unit and integration tests` using `AWS Step Functions`. You can just use `CodeBuild` to handle all of the tests.




`CodePipeline` has `built-in integration with AWS CloudFormation`, so you can specify `AWS CloudFormation-specific actions`, such as creating, updating, or deleting a stack, within a pipeline.

In `a CodePipeline stage`, you can specify `parameter overrides` for `AWS CloudFormation actions`. 
- `Parameter overrides` let you specify `template parameter values` that override values in a template configuration file. 
- `AWS CloudFormation` provides `functions` to help you specify dynamic values (values that are unknown until the pipeline runs).





NOTE: ** `cloudtrail-enabled` `AWS Config managed rule` is only available for the `periodic trigger` type and NOTE `Configuration changes`.

NOTE: ** `AWS Config` will `NOT automatically remediate the accounts that disabled its CloudTrail`. You must manually set this up using a CloudWatch Events rule and a custom Lambda function that calls the StartLogging API to enable CloudTrail back again.





For `AWS::AutoScaling::AutoScalingGroup` resource type reference in your `CloudFormation template`, if both the `AutoScalingReplacingUpdate` and `AutoScalingRollingUpdate` Update policies are specified, setting the `WillReplace` property to `true` gives `AutoScalingReplacingUpdate` precedence. But since this property is set to `false`, then the `AutoScalingRollingUpdate` policy will take precedence instead.

- `AutoScalingRollingUpdate`: doesn’t maintain the total number of active EC2 instances during deployment.

- ** `AutoScalingReplacingUpdate`: instead will create `a separate Auto Scaling group` and is able to perform `an immediate rollback` of the stack in an event of an update failure.





NOTE: ** You can add `actions to your CodePipeline pipeline` that are in `an AWS Region different from your pipeline`. This is better than to maintain several pipelines in multiple AWS regions.

- When an AWS service is the provider for an action, and this action type/provider type are in a different AWS Region from your pipeline, this is `a cross-region action`. 

- Ex. You can configure `cross-region actions` to `build and deploy` the application on `multiple regions`.



NOTE: `Amazon CloudWatch Alarms` can only send notifications to `SNS` and `NOT SQS`.



NOTE: `a CloudWatch Logs subscription` CANNOT be directly integrated with `an AWS Step Functions application`.




`A scheduled action` on the `Auto Scaling group`

- To configure your `Auto Scaling group` to scale based on a schedule, you create `a scheduled action`. 
- The `scheduled action` tells `Amazon EC2 Auto Scaling` to perform `a scaling action at specified times`. 
- ** To create `a scheduled scaling action`, you specify the `start time` when the scaling action should take effect, and `the new minimum, maximum, and desired sizes` for the scaling action. 
- At the specified time, Amazon EC2 Auto Scaling updates the group with the values for `minimum, maximum, and desired size` specified by the scaling action.




NOTE: There is NO `Canary routing option` in `an Application Load Balancer`.

NOTE: The `Network Load Balancer` does NOT support `weighted target groups`, unlike `the Application Load Balancer`.

NOTE: In `API Gateway`, you create `a canary release deployment` when deploying the API with `canary settings` as an additional input to the deployment creation operation.




In `AWS CloudTrail`,` enable the log file integrity feature` on the trail is a built-in feature.
- It will automatically generate a digest file for every log file that CloudTrail delivers. Verify the integrity of the delivered CloudTrail files using the generated digest files.




---


# Time mode 2

Problem: you want to take advantage of it to make sure that all public buckets only allow List operations for intended users. 

Solution:
- Utilize `CloudWatch Events` to monitor `Trusted Advisor security recommendation results` and then set a trigger to send an email using `SNS` to notify you about the results of the check.

- Set up `a custom AWS Config rule` to check `public S3 buckets permissions` and have it send an event on `CloudWatch Events` about policy violations. Have CloudWatch Events trigger `a Lambda Function` to `update the S3 bucket permission`.

- Set up `a custom AWS Config rule` that checks `public S3 buckets permissions`. Then, send a `non-compliance notification` to `your subscribed SNS topic`.





NOTE: There is NO a custom `AWS Config` rule to `execute a default remediation action` to `update the permissions on the public S3 bucket`.

NOTE: `Trusted Advisor` ONLY sends the `summary notification` `WEEKLY` so this won’t notify you immediately about your non-compliant resources.




NOTE: Because configuring `a custom Request and Response Behavior` in `CloudFront` is `NOT enough` to automatically add `the required security headers to the HTTP response`. 
- ** You have to use `Lambda@Edge` to add the headers to satisfy the requirement of this scenario.



Problem: During deployment, CodeDeploy reports that the deployment has `failed` but upon checking `the generated logs, the deployment script has successfully finished`. Why?

Answer: CodeDeploy has reached `timeout` while waiting for `the long-running script` to finish.

Explanation: 
- In CodeDeploy, you can configure `a custom timeout` as part of your deployment lifecycle event hook. The timeout value is a specific amount of time for the script to complete its execution before the deployment is considered failed. 
- The default value is `3600 seconds (1 hour)`. 
- If the script exceeds this limit, the deployment stops and the deployment to the instance fails.





NOTE: To monitor and detect `usage patterns on your S3 data`, use `Amazon Macie`.
- *** NOT `Amazon GuardDuty` because GuardDuty is just a threat detection service that continuously monitors for malicious activity and unauthorized behavior to protect your AWS accounts and workloads, `NOT S3 Data`.




`Amazon Macie` is `an ML-powered security service` that helps you `prevent data loss` by automatically d`iscovering, classifying, and protecting sensitive data stored in Amazon S3`. 

- `Amazon Macie` uses `machine learning` to `recognize sensitive data` such as personally identifiable information (PII) or intellectual property, assigns a business value, and provides visibility into where this data is stored and how it is being used in your organization.

- ** `Amazon Macie` continuously `monitors data access activity for anomalies, and delivers alerts when it detects risk of unauthorized access or inadvertent data leaks`. A

- `mazon Macie` has ability to detect global access permissions inadvertently being set on sensitive data, detect uploading of API keys inside source code, and verify sensitive customer data is being stored and accessed in a manner that meets their compliance standards.




Problem: To find out what’s preventing their `Auto Scaling group` from updating correctly during a CloudFormation stack update using `AutoScalingRollingUpdate` policy. You need to disable roll back by:

Solution:
- In your `AutoScalingRollingUpdate` policy, set the `WaitOnResourceSignals property to false`. >> Because we gonna stop rollaback.
  - NOTE: If WaitOnResourceSignals = true, 
    - AWS CloudFormation waits to receive `a success signal` until the maximum time specified by the `PauseTime` value. 
    - If a signal is `not received`, AWS CloudFormation `cancels the update`. 
    - Then, AWS CloudFormation `rolls back` the stack with the same settings, including the same PauseTime value.

- During a rolling update, suspend the following Auto Scaling processes: `HealthCheck`, `ReplaceUnhealthy`, `AZRebalance`, `AlarmNotification`, and `ScheduledActions`.

- In your `AutoScalingRollingUpdate` policy, set the value of the `MinSuccessfulInstancesPercent` property to `prevent AWS CloudFormation from rolling back the entire stack` if only a single instance fails to launch.





NOTE: In CloudFormation stack update, using `a rolling update` has `a risk of system outages and performance degradation` due to the decreased availability of your running EC2 instances. 
- ** If you want to ensure `high availability` of your application, you can also use the `AutoScalingReplacingUpdate` policy to perform `an immediate rollback of the stack without any possibility of failure`.




`Force new deployment` option in `ECS`

By default, deployments are not forced but you can use the `forceNewDeployment` request parameter (or the `–force-new-deployment` parameter if you are using the AWS CLI) to force a new deployment of the service in the below use cases"

1. If your `updated Docker image uses the same tag` as what is in the existing task definition for your service (for example, my_image:latest), you do not need to create a new revision of your task definition. 
- You can update your service with your custom configuration, keep the current settings for your service, and select `Force new deployment`. 
- The new tasks launched by the deployment pull the current image/tag combination from your repository when they start. 

2. The `Force new deployment` option is also used when updating a Fargate task to use `a more current platform version when you specify LATEST`. 
- For example, if you specified LATEST and your running tasks are using the 1.0.0 platform version and you want them to relaunch using a newer platform version.

By default, deployments are not forced but you can use the forceNewDeployment request parameter (or the –force-new-deployment parameter if you are using the AWS CLI) to force a new deployment of the service. You can use this option to trigger a new deployment with no service definition changes. For example, you can update a service’s tasks to use a newer Docker image with the same image/tag combination (my_image:latest) or to roll Fargate tasks onto a newer platform version.



NOTE: You can call `the EC2 CreateSnapshot API` directly as a target from `CloudWatch Events`.
- No need to call Lambda function or SSM Run Command as a target first.




NOTE: You can create `an origin group` with `two origins to set up an origin failover` in `Amazon CloudFront`. 
- Specify one as `the primary origin` and the other as `the second origin`. 
- ** This configuration will cause the CloudFront service to `automatically switch to the second origin` in the event that the `primary origin returns specific HTTP status code failure responses`.




NOTE: `CloudWatch Alarms` CANNOT receive direct test results from `AWS Lambda`.



Problem: You want to make sure that there are no 5XX error replies on the new version before continuing production deployment and that you are notified via email if results failed. You also want to configure automatic rollback to the older version when the validation fails.

Solution:
1. Create your `validation scripts on AWS Lambda` and define the functions on the `AppSpec lifecycle hook` to validate the app using test traffic.

2. Have `AWS CloudWatch Alarms` trigger an `AWS SNS notification` when `the threshold for 5xx` is reached on CloudWatch.

3. Associate` CloudWatch Alarms` to your `deployment group` to have it trigger `a rollback` when the 5xx error alarm is active.




Problem: To have better visibility on CodeCommit repositories, you want to capture all events in the repository, such as cloning a repo or creating a branch on a single location for you to review.

Solution:
- Create `a Lambda function` that sends event logs to `AWS CloudWatch Logs` and set `a trigger` by selecting the CodeCommit repository. 
- On `CodeCommit`, go to the repository settings and select the `Lambda `function` on the `Triggers` list when creating a new trigger.

Explanation:
- You can set up notification rules for a repository so that repository users receive emails about the repository event types you specify. `Notifications` are sent when events match the notification rule settings. You can create `an Amazon SNS topic` to use for notifications or use an existing one in your AWS account. You use the CodeCommit console and the AWS CLI to configure notifications.

- ** Although you can configure `a trigger` to use `Amazon SNS` to send emails about some repository events, those events are `limited to operational events`, such as creating branches and pushing code to a branch. Triggers do not use CloudWatch Events rules to evaluate repository events. They are more limited in scope.

- You can integrate `Amazon SNS topics` and `Lambda functions` with `triggers in CodeCommit`, but you must first create and then configure resources with a policy that grants CodeCommit the permissions to interact with those resources. 

- You must create the resource `in the same AWS Region as the CodeCommit repository`. For example, if the repository is in US East (Ohio) (us-east-2), the Amazon SNS topic or Lambda function must be in US East (Ohio).




NOTE: Create a CloudWatch Event rule to detect S3 object PUT operations and set the target to the ECS cluster to run a new ECS task directly. 
- Create a Lambda function that will run Amazon ECS API command to increase the number of tasks on ECS is NOT necessary.



Problem: A multinational corporation has multiple AWS accounts that are consolidated using AWS Organizations. For security purposes, a new system should be configured that `automatically detects suspicious activities in any of its accounts`, such as SSH brute force attacks or compromised EC2 instances that serve malware. All of the gathered information must be centrally `stored in its dedicated security account for audit purposes`, and the events should be stored in an S3 bucket.

Solution:
- Automatically detect `SSH brute force or malware attacks` by enabling `Amazon GuardDuty` in `every account`. 
- ** Configure `the security account` as the `GuardDuty Administrator` for every member of the organization. 
- Set up `a new CloudWatch rule` in `the security account`. Configure the rule to send all findings to `Amazon Kinesis Data Firehose`, which will push the findings to `the S3 bucket`.

Explanation:

`GuardDuty` makes enablement and management across multiple accounts easy. Through the `multi-account feature`, all member accounts findings can be aggregated with a `GuardDuty administrator account`. This enables security team to manage all GuardDuty findings from across the organization in one single account. The aggregated findings are also available through CloudWatch Events, making it easy to integrate with an existing enterprise event management system.





NOTE: You can integrate `Amazon SNS topics` and `Lambda functions` with `triggers in CodeCommit`, but 
- you MUST first create and then configure resources with a policy that grants CodeCommit the permissions to interact with those resources. 
- ** You MUST create the resource in `the same AWS Region as the CodeCommit repository`. 
  - For example, if the repository is in US East (Ohio) (us-east-2), the Amazon SNS topic or Lambda function must be in US East (Ohio).




NOTE: You CANNOT configure an S3 bucket to `directly send access logs` to `a CloudWatch log group`.

NOTE: `The object-level events of your S3 bucket` means `S3 API call` that you must use `AWS CloudTrail` first to set up a trail configured to receive these events.
- Then you can store your `S3 API call logs` to an `Amazon S3 bucket`.
- Then create `a Lambda function` that `logs data events of your S3 bucket` that triggered by `CloudWatch Events rule for every action taken on your S3 objects` updated by `AWS CloudTrail`.
- View the logs on `the CloudWatch Logs group`.



NOTE: Unlike Immutable, the deployment setting of the Elastic Beanstalk environment using `rolling deployments with additional batch` are affected and could be unavailable in the event of deployment failure.

- If an `immutable` environment update fails, the `rollback process` requires only `terminating an Auto Scaling group`. 
- `A failed rolling update`, on the other hand, requires performing `an additional rolling update to roll back` the changes.





NOTE: You can create a `CloudTrail trail` that sends logs to `a CloudWatch Log group`, NOT only `S3 bucket`.



`S3 Server access logging` is primarily used to provide `detailed records for the requests` that are made to a bucket. 
- Each access log record provides details about a single access request, such as the requester, bucket name, request time, request action, response status, and an error code, if relevant.

NOTE: To track the S3 bucket policy changes, It is more appropriate to use CloudWatch or CloudTrail.



NOTE: To create an alarm based on logs on `a CloudWatch Log group`, you must first create `a metric filter` to filter specific S3 events and then configure an alarm based on the filter for this Metric with `a threshold of >=1` and set your email as a notification recipient.




NOTE: In `Elastic Beanstalk` environment, a `Blue/green deployment` requires that your environment runs `independently of your production database`. 
- This means that you have to `decouple your RDS database from your environment`. 
- ** If your Elastic Beanstalk environment has an attached Amazon RDS DB instance, the `data will be lost` if you terminate the original (blue) environment.




NOTE: a `Multi-Master Amazon Aurora` database does `NOT work across multiple regions` by default. 




To set up `a Global DynamoDB table`
- In your `preferred AWS region`, set up `a Global DynamoDB table` and then `enable the DynamoDB Streams option`. 
- Set up `replica tables` in the `other AWS regions` where you want to `replicate your data`. 
  - In each local region, store the individual transactions to a DynamoDB replica table in the same region.

** NOTE: `Global DynamoDB Tables` will `NOT automatically create new replica tables on all AWS regions`. 
- You have to` manually specify and create the replica tables in the specific AWS regions` where you want to replicate your data.

** NOTE: `the DynamoDB Stream option` must be enabled in order for the `Global DynamoDB Table` to work.

- For example, you could create a global table consisting of your three Region-specific CustomerProfiles tables. DynamoDB would then automatically replicate data changes among those tables so that changes to CustomerProfiles data in one Region would seamlessly propagate to the other Regions. In addition, if one of the AWS Regions were to become temporarily unavailable, your customers could still access the same CustomerProfiles data in the other Regions.




NOTE: By default, `CodeDeploy removes all files on the deployment location` and the auto rollback will deploy the old revision files cleanly. 
- ** You should choose `“Retain the content”` option for future deployments so that only the files included in the old app revision will be deployed and the existing contents will be retained.





Problem: you want to be notified of any security check recommendations from Trusted Advisor and automatically solve the non-compliance based on the results.

Solution:
- Create `a Lambda function` and integrate `CloudWatch Events` and AWS Lambda to execute the function on a regular schedule to check `AWS Trusted Advisor via API`. Based on the results, publish a message to `an SNS Topic` to notify the subscribers.

- Set up `custom AWS Config rule` that checks security groups to make sure that port 22 is not open to public. Send a notification to `an SNS topic` for non-compliance.

- ** Set up `custom AWS Config rule` to `execute a remediation action` using `AWS Systems Manager Automation` to update the publicly open port 22 on the instances and restrict to only your office’s public IP.

NOTE: the `custom remediation actions` in `AWS Config` should be used in conjunction with `AWS Systems Manager Automation documents`.




NOTE: You can use `AWS EventBridge (Amazon CloudWatch Events)` to detect and react to changes in the status of `AWS Personal Health Dashboard (AWS Health) events`. Then, based on the rules that you create, CloudWatch Events invokes one or more target actions when an event matches the values that you specify in a rule.




NOTE: you can only create a Metric filter for CloudWatch log groups, NOT for AWS Health.




NOTE: With `Match Viewer`, `CloudFront` communicates with your custom origin using HTTP or HTTPS, depending on the protocol of the viewer request. 
- For example, if you choose `Match Viewer` for `Origin Protocol Policy` and the viewer uses HTTP to request an object from CloudFront, CloudFront also uses the same protocol (HTTP) to forward the request to your origin.






Remember that there are rules on which type of SSL Certificate to use if you are using an EC2 or an ELB as your origin. This question is about setting up an end-to-end HTTPS connection between the Viewers, CloudFront, and your custom origin, which is an ALB instance.

The certificate issuer you must use depends on whether you want to require HTTPS between viewers and CloudFront or between CloudFront and your origin:

- HTTPS between viewers and CloudFront
  - ** You can use a certificate that was issued by `a trusted certificate authority (CA)` such as Comodo, DigiCert, Symantec, or other third-party providers.
    - ** You need to import a certificate from a third-party certificate authority into `ACM` or `the IAM certificate store`
  - You can use a certificate provided by `AWS Certificate Manager (ACM)` directly.

- HTTPS between CloudFront and a custom origin
  - If the origin is `not an ELB load balancer`, such as `Amazon EC2`, the certificate `must be issued by a trusted CA` such as Comodo, DigiCert, Symantec or other third-party providers.
  - If your origin is `an ELB load balancer`, you can `also use a certificate provided by ACM`.
    - **  For an SSL certificate that is signed by `a trusted third-party certificate authority` to `ELB`, you need to import a certificate from a third-party certificate authority into `ACM`.



NOTE: You CANNOT use `AWS Lambda` as a notification target for the `lifecycle hook of an Auto Scaling group`. 
- ** You can only configure `Amazon CloudWatch Events`, `Amazon SNS`, or `Amazon SQS` as notification targets.



NOTE: You can set up the AWS Config Managed Rule which automatically checks whether your running EC2 instances are using approved AMIs.


