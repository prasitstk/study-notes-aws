# Review Mode Notes

## SDLC Automation

CodePipeline cross-region actions

- You can add `actions` to your pipeline that are in `an AWS Region different from your pipeline`. 
- * When an AWS service is the provider for an action, and `this action type/provider type are in a different AWS Region from your pipeline`, this is `a cross-region action`. 
  - Certain action types in CodePipeline may only be available in certain AWS Regions. 
  - Also note that there may AWS Regions where an action type is available, but a specific AWS provider for that action type is not available.

- * If you use the console to create a pipeline or cross-region actions, `default artifact buckets` are configured by CodePipeline `in the Regions where you have actions`. 

- You must create `the artifact bucket` and `encryption key` in `the same AWS Region as the cross-region action` and in `the same account as your pipeline`.

- You cannot create cross-region actions for the following action types: source actions, third-party actions, and custom actions. 

- * When a pipeline includes `a cross-region action` as part of a stage, CodePipeline `replicates only the input artifacts` of the cross-region action from the pipeline Region to the action’s Region. 

- The pipeline Region and the Region where your CloudWatch Events change detection resources are maintained remain the same. The Region where your pipeline is hosted does not change.



You can use `CodePipeline` to build a continuous delivery workflow by building a pipeline for `AWS CloudFormation` stacks. 
- CodePipeline has `built-in integration with AWS CloudFormation`, so you can specify `AWS CloudFormation-specific actions`, such as creating, updating, or deleting a stack, within a pipeline.




In designing your CI/CD process in AWS, 
- you can use `a single repository` in `AWS CodeCommit` and create `different branches` for development, master, and release. 
- You can use `CodeBuild` to build your application and run tests to verify that all of the core features of your application are working. 
- For deployment, you can either select an `in-place` or `blue/green` deployment using `CodeDeploy`.



Elastic Beanstalk deployment policies

- `All at once`: this type will cause a brief downtime during deployment. Hence, this is not ideal for critical production applications.
  - This will deploy the new version to all instances `simultaneously`. All instances in your environment are out of service for a short time while the deployment occurs. 
  - ** This is the method that provides `the least amount of time for deployment`.

- `Rolling`: this type will remove a batch of instances from the cluster while deploying the new instances. Hence, this will increase the load to the current EC2 instances.
  - This will deploy the new version in batches. 
  - ** Each batch is taken out of service during the deployment phase, reducing your environment’s capacity by the number of instances in a batch.

- `Rolling with additional batch`: this type is similar to `Rolling` but it will maintain `full capacity` during deployments. You can configure your environment to launch `a new batch of instances` before taking any instances out of service.
  - This will deploy the new version in batches, but `first launch a new batch of instances` to ensure `full capacity` during the deployment process.

- `Immutable`: this type can also maintain the compute capacity of the cluster like `Rolling with additional batch`. However, it entails `a significant amount of cost` since a new batch of instances will be launched.
  - This will deploy `the new version to a fresh group of instances` by performing an immutable update.

- ** `Blue/Green`: Like `Immutable` and `Rolling with additional batch` that has minimal impact but needs `DNS changes` to `separate environment`.
  - This will deploy `the new version` to `a separate environment`, and then `swap CNAMEs` of the two environments to redirect traffic to the new version instantly.





Update Docker Image to ECS service
- If you have updated the Docker image of your application, you can create a new task definition with that image and deploy it to your service.
- ** If your updated Docker image uses `the same tag` as what is in the existing task definition for your service (for example, my_image:latest), you do not need to create a new revision of your task definition. 
  - You can update the service using the procedure below, keep the current settings for your service, and select `Force new deployment`. The new tasks launched by the deployment pull the current image/tag combination from your repository when they start.
  - The `Force new deployment` option is also used when updating `a Fargate task` to use a more current platform version when you specify `LATEST`. 
- To verify that the container agent is running on the your container instance, run the following command:
```sh
sudo systemctl status ecs
```
- If the command output doesn’t show the service as active, run the following command to restart the service:
```sh
sudo systemctl restart ecs
```
- ** NOTE: If the ECS task is still running an old image then it is possible that the ECS agent is not running properly. To fix it, Restart the ECS agent by `sudo systemctl restart ecs` on the instance.




In `AWS CodePipeline`, you can add `an approval action` to `a stage in a pipeline` at the point where you want the pipeline execution to stop so that someone with the required AWS Identity and Access Management permissions can approve or reject the action.

If the action is approved, the pipeline execution resumes. If the action is rejected—or if no one approves or rejects the action within seven days of the pipeline reaching the action and stopping—the result is the same as an action failing, and the pipeline execution does not continue.

You might use manual approvals for these reasons:
- You want someone to perform a code review or change management review before a revision is allowed into the next stage of a pipeline.
- You want someone to perform manual quality assurance testing on the latest version of an application, or to confirm the integrity of a build artifact, before it is released.
- You want someone to review new or updated text before it is published to a company website.




Scenario: Protect an incident in which a junior developer pushed an untested code in the master branch and the changes directly went to their production environment.

Solution: Maintain the recently added AWSCodeCommitPowerUser AWS-managed policy but create an additional policy to include a Deny rule for the codecommit:GitPush action. Add a restriction for the specific repositories in the resource statement with a condition for the master branch.

Explanation:
- For example, you can create a Deny policy that denies users the ability to make changes to a branch named master, including deleting that branch, in a repository named TutorialsDojoManila. You can use this policy with the AWSCodeCommitPowerUser managed policy. Users with these two policies applied would be able to create and delete branches, create pull requests, and all other actions as allowed by AWSCodeCommitPowerUser, but they would not be able to push changes to the branch named master, add or edit a file in the master branch in the CodeCommit console, or merge branches or a pull request into the master branch. 

- ** Because Deny is applied to GitPush, you must include a Null statement in the policy, to allow initial GitPush calls to be analyzed for validity when users make pushes from their local repos.

- There are various AWS-managed policies that you can use for providing CodeCommit access. They are:

  - `AWSCodeCommitFullAccess` – Grants full access to CodeCommit. You should apply this policy only to administrative-level users to whom you want to grant full control over CodeCommit repositories and related resources in your AWS account, including the ability to delete repositories.

  - `AWSCodeCommitPowerUser` – Allows users access to all of the functionality of CodeCommit and repository-related resources, except it does `not allow them to delete CodeCommit repositories` or `create or delete repository-related resources in other AWS services`, such as `Amazon CloudWatch Events`. 
    - * It is recommended to apply this policy to most users.

  - `AWSCodeCommitReadOnly` – Grants read-only access to CodeCommit and repository-related resources in other AWS services, as well as the ability to create and manage their own CodeCommit-related resources (such as Git credentials and SSH keys for their IAM user to use when accessing repositories). 
    - * You should apply this policy to users to whom you want to grant the ability to read the contents of a repository, but not make any changes to its contents.



NOTE: Remember that you can’t modify these AWS-managed policies. In order to customize the permissions, you can add a Deny rule to the IAM Role in order to block certain capabilities included in these policies.




AWS OpsWorks Stacks :: Time-based/Load-based instances

- Unlike 24/7 instances, which you must start and stop manually, you do not start or stop time-based or load-based instances yourself. 
  - Instead, you configure `the instances` and `AWS OpsWorks Stacks` starts or stops them based on their configuration.

- `Automatic scaling` is based on two instance types, which adjust a layer’s online instances based on different criteria:
  
  - `Time-based instances` – They allow a stack to handle loads that follow a predictable pattern by including instances that run only at certain times or on certain days. 
    - For example, you could start some instances after 6PM to perform nightly backup tasks or stop some instances on weekends when traffic is lower.

  - `Load-based instances` – They allow a stack to handle variable loads by starting additional instances when traffic is high and stopping instances when traffic is low, based on any of several load metrics. 
    - For example, you can have `AWS OpsWorks Stacks` start instances when the average CPU utilization exceeds 80% and stop instances when the average CPU load falls below 60%.




CodeDeploy life cycle events that are reserved for only CodeDeploy operations. Cannot be used to run scripts are:
- DownloadBundle
- Install
- BlockTraffic
- AllowTraffic

Rather than above ones, you can use those events to run tasks on instances in `appspec.yml`.




`AWS::AutoScaling::AutoScalingGroup` resource type reference in your `CloudFormation` template

- You can add an `UpdatePolicy` attribute to your Auto Scaling group to perform `rolling updates` (or `replace` the group) when a change has been made to the group.

  - `AutoScalingReplacingUpdate` policy
    - To specify how AWS CloudFormation handles replacement updates for an Auto Scaling group, use the `AutoScalingReplacingUpdate` policy. 
    - This policy enables you to specify whether AWS CloudFormation `replaces an Auto Scaling group with a new one` or `replaces only the instances in the Auto Scaling group.` 

  1. Configure the `UpdatePolicy` of the `AWS::AutoScaling::AutoscalingGroup` resource in the `CloudFormation template` to use the `AutoScalingReplacingUpdate` policy. Update the required properties for the new Auto Scaling group policy. Set the `WillReplace` property to `true`.
    - implement an efficient strategy for deploying updates to their web application with `the ability to perform an immediate rollback of the stack`. 
    - All deployments should `maintain the normal number of active EC2 instances` to keep the performance of the application.

  2. Configure the `UpdatePolicy` of the `AWS::AutoScaling::AutoscalingGroup` resource in the `CloudFormation template` to use the `AutoScalingRollingUpdate` policy. Update the required properties for the new Auto Scaling group policy. Set the `MinSuccessfulInstancesPercent` property, you must also enable the `WaitOnResourceSignals` and `PauseTime` properties
    - this type of deployment will affect the existing compute capacity of your application. 
    - The rolling update doesn’t maintain the total number of active EC2 instances during deployment.




When you `deploy to an AWS Lambda` compute platform, the deployment configuration specifies the way traffic is shifted to the new Lambda function versions in your application. There are `three ways traffic can shift` during a deployment:

1. `Canary`: Traffic is shifted in `two increments`. 
  - You can choose from predefined canary options that specify `the percentage of traffic shifted to your updated Lambda function version in the first increment` and `the interval, in minutes, before the remaining traffic is shifted in the second increment`.

2. `Linear`: Traffic is shifted in `equal increments with an equal number of minutes between each increment`. 
  - You can choose from predefined linear options that specify `the percentage of traffic shifted in each increment` and `the number of minutes between each increment`.

3. `All-at-once`: All traffic is shifted from the original Lambda function to the updated Lambda function version all at once.





`Systems Manager Automation` simplifies `common maintenance and deployment tasks` of `Amazon EC2 instances` and `other AWS resources`. `Automation` enables you to do the following.
- Build Automation workflows to configure and manage instances and AWS resources.
- Create custom workflows or use pre-defined workflows maintained by AWS.
- Receive notifications about Automation tasks and workflows by using Amazon CloudWatch Events.
- Monitor Automation progress and execution details by using the Amazon EC2 or the AWS Systems Manager console.

Automation offers `one-click automation` for simplifying complex tasks such as creating golden Amazon Machines Images (AMIs) and recovering unreachable EC2 instances. Here are some examples:

- Use the `AWS-UpdateLinuxAmi` and `AWS-UpdateWindowsAmi` documents to `create golden AMIs from a source AMI`. 
  - You can run custom scripts before and after updates are applied. You can also include or exclude specific packages from being installed.

- Use the `AWSSupport-ExecuteEC2Rescue` document to `recover impaired instances`. 
  - An instance can become `unreachable` for various reasons, including network misconfigurations, RDP issues, or firewall settings. 
  - Troubleshooting and regaining access to the instance previously required dozens of manual steps before you could regain access. 
  - The `AWSSupport-ExecuteEC2Rescue` document lets you `regain access` by specifying `an instance ID` and clicking a button.




Problem: 
- A company development department has `a proprietary source code analysis tool` that follows the Open Web Application Security Project (OWASP) standard. 
- The tool is `hosted on a dedicated server` in its data center that checks the code quality and security vulnerabilities of their projects.
- The company plans to use this tool to run checks against the source code as part of the pipeline before the code is compiled into a deployable package in CodeDeploy. 
- The code checks `take approximately an hour` to complete.

Solution:

- Create `a pipeline` in AWS CodePipeline. 
- Set up `a custom action type` and create `an associated job worker` that runs on-premises. 
- Set the pipeline to invoke the custom action `after the source stage`. 
- Configure `the job worker` to `poll CodePipeline for job requests` for the custom action then execute the source code analysis tool and return the status result to CodePipeline.

Explanation:

NOTE: `a CodePipeline` CANNOT `directly poll` the contents of an S3 bucket. Instead you can create a job worker that poll the job request from CodePipeline

AWS CodePipeline custom actions

- When you create `a custom action`, you must also create `a job worker` that will `poll CodePipeline` for `job requests` for this custom action, execute the job, and return `the status result` to CodePipeline. 
  - This `job worker` can be located on any on-premise computer or resource as long as it has access to the public endpoint for CodePipeline (may through VPN). 
  - To easily manage access and security, consider hosting your job worker on an Amazon EC2 instance.

- When a pipeline includes a custom action as part of a stage, the pipeline will create `a job request`. 
- `A custom job worker` detects that request and performs that job (in this example, a custom process using third-party build software). 
- When the action is complete, the job worker returns either `a success result` or `a failure result`. 
  - If a success result is received, the pipeline will transition the revision and its artifacts to the next action. 
  - If a failure is returned, the pipeline will not transition the revision to the next action in the pipeline.




Problem: The Auto Scaling group is configured to use Elastic Load Balancing health checks for scaling instead of the default EC2 status checks. However, there are several occasions when some instances are automatically terminated after failing the HTTPS health checks in the ALB that purges all the logs stored in the instance. To improve system monitoring, a DevOps Engineer must implement a solution that collects all of the application and server logs effectively. The Operations team should be able to perform a root cause analysis based on the logs, even if the Auto Scaling group immediately terminated the instance.

Solution:
- Delay the termination of unhealthy Amazon EC2 instances by `adding a lifecycle hook` to your Auto Scaling group to move instances in the `Terminating` state to the `Terminating:Wait` state. 
- Set up `a CloudWatch Events rule` for the `EC2 Instance-terminate Lifecycle Action Auto Scaling Event` with an associated `AWS Systems Manager Automation document`. 
- Trigger `the CloudWatch agent` to `push the application logs` and then resume the instance termination once all the logs are sent to CloudWatch Logs.

Explanation:

The EC2 instances in an Auto Scaling group have a path, or lifecycle, that differs from that of other EC2 instances. 
- `The lifecycle starts` when the Auto Scaling group `launches an instance` and puts it into service. 
- `The lifecycle ends` when you `terminate the instance`, or the Auto Scaling group takes the instance out of service and terminates it.

You can add a lifecycle hook to your Auto Scaling group so that you can perform custom actions when instances launch or terminate.

- When Amazon EC2 Auto Scaling responds to `a scale out event`, it `launches one or more instances`. 
  - These instances start in the `Pending` state. 
  - ** If you added `an autoscaling:EC2_INSTANCE_LAUNCHING lifecycle hook` to your Auto Scaling group, the instances move from the `Pending` state to the `Pending:Wait` state. 
  - After you complete the lifecycle action, the instances enter the `Pending:Proceed` state. 
  - When the instances are fully configured, they are attached to the Auto Scaling group and they enter the `InService` state.

- When Amazon EC2 Auto Scaling responds to `a scale in event`, it `terminates one or more instances`. 
  - These instances are detached from the Auto Scaling group and enter the `Terminating` state. 
  - ** If you added `an autoscaling:EC2_INSTANCE_TERMINATING lifecycle hook` to your Auto Scaling group, the instances move from the `Terminating` state to the `Terminating:Wait` state. 
  - After you complete the lifecycle action, the instances enter the `Terminating:Proceed` state. 
  - When the instances are fully terminated, they enter the `Terminated` state.

For example, your newly launched instance completes its startup sequence and a lifecycle hook pauses the instance. While the instance is in a wait state, you can install or configure software on it, making sure that your instance is fully ready before it starts receiving traffic. For another example of the use of lifecycle hooks, when a scale-in event occurs, the terminating instance is first deregistered from the load balancer (if the Auto Scaling group is being used with Elastic Load Balancing). Then, a lifecycle hook pauses the instance before it is terminated. While the instance is in the wait state, you can, for example, connect to the instance and download logs or other data before the instance is fully terminated.





Problem: The agency purchased a proprietary software with `100 licenses`, which can be used by a maximum of 100 application servers. A DevOps Engineer needs to set up an automated solution that `dynamically allocates the software licenses to the application servers`. The Engineer also needs to provide a way to see the list of available licenses that are not in use.

Solution:
- Prepare a CloudFormation template that uses an Auto Scaling group to launch the EC2 instances with an associated lifecycle hook for `Instance Terminate`. 
- Store the `100 license codes` to `a DynamoDB table`. 
- Pull an available license from the DynamoDB table using the `User Data script` (not metadata) of the instance upon launch. 
- Use the `lifecycle hook` to `update the license mapping` after the instance is terminated.




You can use `AWS CloudFormation` to automatically install, configure, and start `applications on Amazon EC2 instances`. 
- Doing so enables you to easily duplicate deployments and update existing installations `without connecting directly to the instance`, which can save you a lot of time and effort.

AWS CloudFormation includes `a set of helper scripts` (cfn-init, cfn-signal, cfn-get-metadata, and cfn-hup) that are based on `cloud-init`. 
- You call these helper scripts from your AWS CloudFormation templates to install, configure, and update `applications on Amazon EC2 instances` that are in the same template.




NOTE: Take note that you CANNOT `directly fetch the media files` from your `tape gateway` in `real-time` since this is backed up using `Glacier`.
- Although the on-premises data center is using a tape gateway, you can still set up a solution to use `a file gateway` in order to properly process the videos using `Amazon Rekognition`. 
- Keep in mind that the `tape gateway in AWS Storage Gateway service` is primarily used as an `archive solution`.





`Amazon CloudWatch Events` is a web service that monitors your AWS resources and the applications you run on AWS. You can use Amazon CloudWatch Events to `detect and react to changes` in `the state of a pipeline, stage, or action`. 

- Then, based on rules you create, CloudWatch Events invokes one or more target actions when a pipeline, stage, or action enters the state you specify in a rule. 

- Depending on `the type of state change`, you might want to send notifications, capture state information, take corrective action, initiate events, or take other actions.

You can configure notifications to be sent when the state changes for:

- Specified `pipelines` or all your pipelines. You control this by using `"detail-type": "CodePipeline Pipeline Execution State Change"` in the JSON of custom event pattern.

- Specified `stages` or all your stages, within a specified pipeline or all your pipelines. You control this by using `"detail-type": "CodePipeline Stage Execution State Change"` in the JSON of custom event pattern.

- Specified `actions` or all actions, within a specified stage or all stages, within a specified pipeline or all your pipelines. You control this by using `"detail-type": "CodePipeline Action Execution State Change"` in the JSON of custom event pattern.





NOTE: `CloudWatch Events` can directly target `an ECS task` on the `Targets` section when you create a new rule.
- Creating your own Lambda function to run the ECS tasks is not really necessary. 




Update ECS Fargate tasks

- If your updated `Docker image` uses `the same tag` as what is in the existing task definition for your service (for example, my_image:latest), you `do not need to create a new revision of your task definition`. 

- You can update your service with your custom configuration, keep the current settings for your service, and select `Force new deployment`. 
  - The new tasks launched by the deployment pull the current image/tag combination from your repository when they start.
  
- The `Force new deployment` option is also used when updating `a Fargate task` to use a more current platform version when you specify `LATEST`. 
  - For example, if you specified LATEST and your running tasks are using the 1.0.0 platform version and you want them to relaunch using a newer platform version.

* NOTE: There is NO `“Redeploy”` deployment strategy option in ECS.
NOTE: There is NO `the automatic platform version upgrade` feature in ECS





Problem: The `AutoScalingRollingUpdate` policy is used to control how CloudFormation handles rolling updates for their Auto Scaling group which replaces the old instances based on the parameters they have set. 
- ** Lately, there were `a lot of failed deployments` which has caused system unavailability issues and business disruptions. 
- They want to `find out what’s preventing their Auto Scaling group from updating correctly` during a stack update.

In this scenario, how should the DevOps engineer `troubleshoot` this issue? 

Solution:

To find out what’s preventing your Auto Scaling group from updating correctly during a stack update, work through the following troubleshooting scenarios as needed:

- In your `AutoScalingRollingUpdate policy`, set the `WaitOnResourceSignals` property to `false`.
  - NOTE: If WaitOnResourceSignals is set to true, PauseTime changes to a timeout value. AWS CloudFormation waits to receive a success signal until the maximum time specified by the PauseTime value. If a signal is not received, AWS CloudFormation cancels the update. Then, AWS CloudFormation rolls back the stack with the same settings, including the same PauseTime value.

- In your `AutoScalingRollingUpdate policy`, set the value of the `MinSuccessfulInstancesPercent` property to prevent AWS CloudFormation from `rolling back the entire stack` if only `a single instance fails to launch`
  - If you’re replacing a large number of instances during a rolling update and waiting for a success signal for each instance, complete the following: 
    - In your `AutoScalingRollingUpdate policy`, set the value of the `MinSuccessfulInstancesPercent` property. 
  - Take note that setting the `MinSuccessfulInstancesPercent` property prevents AWS CloudFormation from rolling back the entire stack if only a single instance fails to launch.

- During a rolling update, suspend the following Auto Scaling processes: `HealthCheck`, `ReplaceUnhealthy`, `AZRebalance`, `AlarmNotification`, and `ScheduledActions`
  - It is quite important to know that if you’re using your `Auto Scaling group with Elastic Load Balancing (ELB)`, you should NOT suspend the following processes: `Launch`, `Terminate`, and `AddToLoadBalancer`. 
    - These processes are `required to make rolling updates`. 
    - Take note that if an unexpected scaling action changes the state of the Auto Scaling group during a rolling update, the update can fail. 
    - The failure can result from an inconsistent view of the group by AWS CloudFormation.





The `AWS::AutoScaling::AutoScalingGroup` resource uses the `UpdatePolicy` attribute to define how an Auto Scaling group resource is updated when the AWS CloudFormation stack is updated. If you don’t have the right settings configured for the `UpdatePolicy` attribute, your rolling update can produce unexpected results.

You can use the `AutoScalingRollingUpdate policy` to control how AWS CloudFormation handles rolling updates for an Auto Scaling group. This common approach keeps the same Auto Scaling group, and then replaces the old instances based on the parameters that you set.

The `AutoScalingRollingUpdate policy` supports the following configuration options:
```
    "UpdatePolicy": {
      "AutoScalingRollingUpdate": {
        "MaxBatchSize": Integer,
        "MinInstancesInService": Integer,
        "MinSuccessfulInstancesPercent": Integer,
        "PauseTime": String,
        "SuspendProcesses": [ List of processes ],
        "WaitOnResourceSignals": Boolean
      }
    }
```

Using a rolling update has `a risk of system outages and performance degradation due to the decreased availability of your running EC2 instances`. 

** If you want to ensure `high availability of your application`, you can also use the `AutoScalingReplacingUpdate` policy to perform `an immediate rollback of the stack` without any possibility of failure.




NOTE: you have to use `Amazon SNS` in order to `send an email notification` and not `Amazon SES`.



NOTE: you have to use `CloudWatch Events` to detect the changes in the `CodePipeline pipeline stages` and not `CloudTrail`.



NOTE: `AWSCodeCommitPowerUser` policy does NOT ALLOW `repository deletion` at all. Moreover, it also ALLOWS `merge actions to your branch`.


NOTE: In `CodeDeploy`, you can configure `a timeout` as part of your deployment lifecycle event hook. 
- The `timeout` value is a specific amount of time for the script to complete its execution before the deployment is considered failed. 
- The `default value` is `3600 seconds (1 hour)`. 
  - If the script exceeds this limit, the deployment stops and the deployment to the instance fails.



NOTE: `CloudWatch Alarms` can’t receive direct test results from `AWS Lambda`.
- ** In other words,` a Lambda function` CANNOT send results to `AWS CloudWatch Alarms` directly



Problem: You are working on a web application that allows designers to create mobile themes for their android phones. The app is hosted on a cluster of Auto Scaling ECS instances and its deployments are handled by AWS CodeDeploy. ALB health checks are not sufficient to tell that new version deployments are successful, rather you have custom validation scripts that verify all APIs of the application. You want to make sure that there are no 5XX error replies on the new version before continuing production deployment and that you are notified via email if results failed. You also want to configure automatic rollback to the older version when the validation fails.

Solution:
- Create your `validation scripts` on `AWS Lambda` and define the functions on the `AppSpec lifecycle hook` to validate the app using test traffic.
  - The `AfterAllowTestTraffic` lifecycle hook of the `AppSpec.yaml` file allows you to use Lambda functions to validate the new version task set using the test traffic during the deployment. 
  - For example, a Lambda function can serve traffic to the test listener and track metrics from the replacement task set. 
  - ** If rollbacks are configured, you can configure `a CloudWatch alarm` that triggers a rollback when the validation test in your Lambda function fails.

 - Have `AWS CloudWatch Alarms` trigger `an AWS SNS notification` when the threshold for 5xx is reached on CloudWatch.

 - Associate `CloudWatch Alarms` to your `deployment group` to have it trigger a `rollback` when the `5xx error` alarm is active.

Explanation:
- You can use `CloudWatch Alarms` to track metrics on `your new deployment` and you can set `thresholds` for those metrics in your `Auto Scaling groups` being managed by `CodeDeploy`. 





As part of the deployment process, the `CodeDeploy agent` removes from each instance all the files installed by the most recent deployment. If files that weren’t part of a previous deployment appear in target deployment locations, you can choose what CodeDeploy does with them during the next deployment:
- `Fail the deployment` — An error is reported and the deployment status is changed to Failed.
- `Overwrite the content` — The version of the file from the application revision replaces the version already on the instance.
- ** `Retain the content` — The file in the target location is kept and the version in the application revision is not copied to the instance.

To sum: By default, `CodeDeploy` removes all files on the deployment location and the auto rollback will deploy the old revision files cleanly = `“Overwrite content”` option
  - Meaning that the CodeDeploy agent removes from each instance all the files installed by the most recent deployment and not retain them. 
** You should choose `“Retain the content”` option for future deployments so that `only the files included in the old app revision will be deployed` and the `existing contents will be retained`.





ECS Fargate re-deploy with the same image tag

- If your updated Docker image uses `the same tag` as what is in the existing task definition for your service (for example, my_image:latest), you do not need to create a new revision of your task definition. 
- You can update your service with your custom configuration, keep the current settings for your service, and select `Force new deployment`.
- The new tasks launched by the deployment pull the current image/tag combination from your repository when they start. 
- The `Force new deployment` option is also used when updating a Fargate task to use a more current platform version when you specify `LATEST`. 
  - For example, if you specified `LATEST` and your running tasks are using the `1.0.0` platform version and you want them to relaunch using a newer platform version.

- By default, deployments are not forced but you can use the `forceNewDeployment` request parameter (or the `--force-new-deployment` parameter if you are using the AWS CLI) to force a new deployment of the service. 
  - You can use this option to trigger a new deployment with no service definition changes. 
  - For example, you can update a service’s tasks to use a newer Docker image with the same image/tag combination (my_image:latest) or to roll Fargate tasks onto a newer platform version.
 




There are various AWS-managed policies that you can use for providing CodeCommit access. They are:

- `AWSCodeCommitFullAccess` – Grants full access to CodeCommit. You should apply this policy only to administrative-level users to whom you want to grant full control over CodeCommit repositories and related resources in your AWS account, including the ability to delete repositories.

- ** `AWSCodeCommitPowerUser` – Allows users access to all of the functionality of CodeCommit and repository-related resources, except it does `not allow them to delete CodeCommit repositories` or `create or delete repository-related resources in other AWS services`, such as Amazon CloudWatch Events. 
  ** It is recommended to apply this policy to most users.

- `AWSCodeCommitReadOnly` – Grants read-only access to CodeCommit and repository-related resources in other AWS services, as well as the ability to create and manage their own CodeCommit-related resources (such as Git credentials and SSH keys for their IAM user to use when accessing repositories). You should apply this policy to users to whom you want to grant the ability to read the contents of a repository, but not make any changes to its contents.





`Global DynamoDB Tables` builds upon DynamoDB’s global footprint to provide you with a fully managed, multi-region, and multi-master database that provides fast, local, read and write performance for massively scaled, global applications. 
- * `Global Tables` `replicates` your Amazon DynamoDB tables `automatically` across your choice of AWS regions.

- ** However, `NOT automatically create new replica tables on all AWS regions`. You have to `manually specify and create the replica tables in the specific AWS regions` where you want to replicate your data.

- *** IMPORTANT: Take note as well that the `DynamoDB Stream option` MUST be enabled in order for the `Global DynamoDB Table to work`.

- In addition, if one of the AWS Regions were to become temporarily unavailable, your customers could still access the same CustomerProfiles data in the other Regions.




Problem: Enable AWS X-Ray on AWS ECS service

Solution: 
- Produce `a Docker image` (separated from your applications) that runs the `X-Ray daemon`. 
- Upload the image to a Docker image repository, and then deploy it to your `Amazon ECS cluster`. 
- ** Configure the `network mode settings` and `port mappings` in your task definition file to allow traffic on `UDP port 2000`.

Explanation:

The `AWS X-Ray SDK` does NOT send trace data directly to `AWS X-Ray`. 
- * To avoid calling the service every time your application serves a request, the SDK `sends the trace data to a daemon`, which collects segments for multiple requests and `uploads them in batches`.
- Use a script to run the daemon alongside your application.

To properly instrument your applications in Amazon ECS, you have to create `a Docker image that runs the X-Ray daemon`, upload it to a Docker image repository, and then deploy it to your Amazon ECS cluster. You can use port mappings and network mode settings in your task definition file to `allow your application to communicate with the daemon container`.

The `AWS X-Ray daemon` is a software application that listens for traffic on `UDP port 2000`, gathers raw segment data, and relays it to the `AWS X-Ray API`. The daemon works in conjunction with the AWS X-Ray SDKs and must be running so that data sent by the SDKs can reach the X-Ray service.





`Immutable` ELB deployment

- Immutable environment updates are an alternative to rolling updates. Immutable environment updates ensure that configuration changes that require replacing instances are applied efficiently and safely. If an immutable environment update fails, the rollback process requires only terminating an Auto Scaling group. A failed rolling update, on the other hand, requires performing an additional rolling update to roll back the changes.

- To perform an immutable environment update, Elastic Beanstalk `creates a second, temporary Auto Scaling group` behind your environment’s load balancer to contain the new instances. 
  - First, Elastic Beanstalk launches a single instance with the new configuration in the new group. 
    - ** This instance `serves traffic alongside all of the instances in the original Auto Scaling group` that are running the previous configuration.

- When the first instance passes health checks, Elastic Beanstalk launches additional instances with the new configuration, matching the number of instances running in the original Auto Scaling group. 
  - ** When all of the new instances pass health checks, Elastic Beanstalk transfers them to the original Auto Scaling group, and `terminates the temporary Auto Scaling group and old instances`.





Problem: CodeDeploy :: For each deployment, you need to `verify that the set of core APIs are working` before continuing to allow production traffic to flow on the new app version. If the API tests are unsuccessful, the deployment will automatically rollback to the older version. Your `validation scripts` are `a set of functions on AWS Lambda` which should run for each deployment.

Solution: 
- Define your Lambda functions in the `AfterAllowTestTraffic` lifecycle hook of the `AppSpec.yaml` file. 
- Configure the Lambda functions to validate the deployment using `the test traffic` and `rollback if the tests fail`.

NOTE: During `an Amazon ECS deployment` with `validation tests`, `CodeDeploy` uses `a load balancer` that is configured with `two target groups`: 
- one production traffic listener and 
- one test traffic listener.





---





## Configuration Management and Infrastructure as Code

Aurora global database

- An Aurora global database consists of `one primary AWS Region` where your data is mastered, and `one read-only, secondary AWS Region`. 
  - Aurora replicates data to the secondary AWS Region with typical latency of under a second. 
  - You issue write operations directly to the primary DB instance in the primary AWS Region. 
  - An Aurora global database uses dedicated infrastructure to replicate your data, leaving database resources available entirely to serve application workloads. 
  - Applications with a worldwide footprint can use reader instances in the secondary AWS Region for low latency reads. 

- * In the unlikely event your database becomes degraded or isolated in an AWS region, you `can promote the secondary AWS Region to take full read-write workloads` in `under a minute`.

- The Aurora cluster in the primary AWS Region where your data is mastered performs both read and write operations. 

- The cluster in the secondary region enables low-latency reads. 

- You can scale up the secondary cluster independently by adding one or more DB instances (Aurora Replicas) to serve read-only workloads. 

- * For `disaster recovery`, you can remove and promote the secondary cluster to allow full read and write operations.

- * Only the primary cluster performs write operations. Clients that perform write operations connect to the DB cluster endpoint of the primary cluster.



`A VPC endpoint` enables you to privately connect your VPC to supported AWS services and VPC endpoint services powered by PrivateLink without requiring an Internet gateway, NAT device, VPN connection, or AWS Direct Connect connection. 
- Instances in your VPC `do not require public IP addresses` to communicate with resources in the service. 
- Traffic between your VPC and the other service does not leave the Amazon network.

There are two types of VPC endpoints: `interface endpoints` and `gateway endpoints`. 
- You can create the type of VPC endpoint required by the supported service. 
- `S3 and DynamoDB` are using `Gateway endpoints` while most of the services are using Interface endpoints.



Secret Manager vs. SSM Parameter Store

Although the use of the AWS `Systems Manager Parameter Store` service in securing sensitive data in ECS is valid, this service doesn’t provide dedicated storage with lifecycle management and key rotation, unlike `Secrets Manager`. 

Moreover, the AWS `Systems Manager Parameter Store` doesn’t have a built-in automatic key rotation and can only store `up to 8KB of data`.

To sum, 
- if you want `a single store for configuration and secrets`, you can use `Parameter Store`. 
- if you want `a dedicated secrets store with lifecycle management`, use `Secrets Manager`. 



Problem: `CloudFormation` stack `deletion fails` for an S3 bucket
Reason: The CloudFormation stack deletion fails for `an S3 bucket that still has contents`. 
Solution: To fix the issue, modify `the Lambda function code of the custom resource` to recursively empty the bucket if the stack is selected for deletion.

NOTE: you can only set the `DeletionPolicy` to either `Retain` or `Delete` for `an Amazon S3 resource`, NO `Snapshot` for it. 
  - ** In addition, the CloudFormation `deletion will still fail` `as long as the S3 bucket is not empty`, even if the `DeletionPolicy` attribute is already set to `Delete`.

NOTE: (x) `ForceDelete` is not a valid value for the `deletion policy` attribute.



CodeDeploy 
- is a deployment service that automates application deployments to 
  - Amazon EC2 instances, 
  - on-premises instances, 
  - serverless Lambda functions, or 
  - Amazon ECS services.
- When you deploy to `an AWS Lambda compute platform`, the `deployment configuration` specifies `the way traffic is shifted to the new Lambda function versions` in your application. There are three ways traffic can shift during a deployment:
  1. `Canary`: Traffic is shifted in `two increments`. 
    - You can choose from predefined canary options that specify the percentage of traffic shifted to your updated Lambda function version in the first increment and the interval, in minutes, before the remaining traffic is shifted in the second increment.
  2. `Linear`: Traffic is shifted in `equal increments` with `an equal number of minutes` between each increment. 
    - You can choose from predefined linear options that specify the percentage of traffic shifted in each increment and the number of minutes between each increment.
  3. `All-at-once`: All traffic is shifted from the original Lambda function to the updated Lambda function version all at once.



Amazon DynamoDB secondary indexes
- A secondary index is a data structure that contains a subset of attributes from a table, along with an alternate key to support Query operations. 
  - You can retrieve data from the index using a Query, in much the same way as you use Query with a table. 
  - A table can have multiple secondary indexes, which gives your applications access to many different query patterns.

- DynamoDB supports two types of secondary indexes:
  1. `Global secondary index`: an index with `a partition key and a sort key that can be different from those on the base table`. 
    - A global secondary index is considered `“global”` because `queries on the index can span all of the data in the base table`, across `all partitions`.

  2. * `Local secondary index`: an index that has `the same partition key as the base table, but a different sort key`. 
    - ** A local secondary index is `“local”` in the sense that `every partition of a local secondary index is scoped to a base table partition` that has `the same partition key value`.
    - A local secondary index also contains a copy of some or all of the attributes from its base table; you specify which attributes are projected into the local secondary index when you create the table. The data in a local secondary index is organized by the same partition key as the base table, but with a different sort key. This lets you access data items efficiently across this different dimension.
    - For greater query or scan flexibility, you can create `up to five local secondary indexes per table`.

  - `A projection` is `the set of attributes that is copied from a table into a secondary index`. 
    - `The partition key` and `sort key` of the table are always projected into the index; you can project other attributes to support your application’s query requirements. 
    - * When you query an index, Amazon DynamoDB can access any attribute in the projection as if those attributes were in a table of their own.



CloudFormation Nested stacks

- `Nested stacks` are stacks created as part of other stacks. You create a nested stack within another stack by using the `AWS::CloudFormation::Stack` resource.

- As your infrastructure grows, common patterns can emerge in which you declare the same components in multiple templates. You can separate out `these common components` and create `dedicated templates` for them. Then use the resource in your template to reference other templates, creating `nested stacks`.




NOTE: By setting up both the `MinSize` and `MaxSize` parameters of the `Auto Scaling group` to `1`, you can ensure that your EC2 instance can recover again in the event of systems failure with exactly the same parameters defined in the CloudFormation template. 
- This is one of the Auto Scaling strategies which provides high availability with the least possible cost. 



AWS Config triggers
1. Configuration changes
  - AWS Config runs evaluations for the rule when certain types of resources are created, changed, or deleted. 
  - You choose which resources trigger the evaluation by defining the rule’s scope. The scope can include the following:
    - One or more resource types
    - A combination of a resource type and a resource ID
    - A combination of a tag key and value
    - When any recorded resource is created, updated, or deleted
  - AWS Config runs the evaluation when it detects a change to a resource that matches the rule’s scope. 
  - You can use the scope to constrain which resources trigger evaluations. Otherwise, evaluations are triggered when any recorded resource changes.

  - Ex: Set up an AWS Config rule with a configuration change trigger that will detect any changes in the S3 bucket configuration and which will also invoke an AWS Systems Manager Automation document with a Lambda function that will revert any changes.

2. Periodic
  - AWS Config runs evaluations for the rule at a frequency that you choose (for example, every 24 hours).
  - ** NOTE: this is `not the fastest way of detecting a change` in your resource configurations in AWS. 
    - Since the rule is using a periodic trigger, the rule will run every hour and not in near real-time unlike the Configuration changes trigger. 
    - So say a new configuration was applied at 12:01 PM, the change will only be detected at 1:00 PM after the rule has been run.



Set up an active-passive failover configuration on Route 53

Problem: to configure Amazon Route 53 to automatically route to an alternate endpoint when their primary application stack in us-west-1 region experiences an outage or degradation of service.

Solution:
- Step 1: Use a `Failover routing policy` configuration. Set up `alias records` in Route 53 that route traffic to AWS resources. Set the `Evaluate Target Health` option to `Yes`, then create `all of the required non-alias records`.
- Step 2: Set up `health checks` in Route 53 for `non-alias records` to each service endpoint. Configure the network access control list and the route table to allow Route 53 to send requests to the endpoints specified in the health checks.

Explanation:

To create an active-passive failover configuration with one primary record and one secondary record, you just create the records and specify Failover for the routing policy. When the primary resource is healthy, Route 53 responds to DNS queries using the primary record. When the primary resource is unhealthy, Route 53 responds to DNS queries using the secondary record.

You can configure a health check that monitors an endpoint that you specify either by IP address or by domain name. At regular intervals that you specify, Route 53 submits automated requests over the Internet to your application, server, or other resources to verify that it’s reachable, available, and functional. Optionally, you can configure the health check to make requests similar to those that your users make, such as requesting a web page from a specific URL.

** When Route 53 checks the health of an endpoint, it sends an HTTP, HTTPS, or TCP request to the IP address and port that you specified when you created the health check. For a health check to succeed, your router and firewall rules `must allow inbound traffic from the IP addresses that the Route 53 health checkers use`.



Problem: To import Amazon Linux image (ISO) into the on-premise virtualization server for CI/CD and testing purpose before deploying it into AWS.

Solution:
- Launch an Amazon EC2 instance with the latest Amazon Linux OS in AWS. 
- ** Use the `AWS VM Import/Export service` to import `the EC2 image`, export it to `a VMware ISO` in an S3 bucket, and then import `the ISO` to an on-premises server. 
- Once done, commence the testing activity to verify the application's functionalities.

Explanation:

- The `VM Import/Export` enables you to easily import virtual machine images from your existing environment to Amazon EC2 instances and export them back to your on-premises environment. This offering allows you to leverage your existing investments in the virtual machines that you have built to meet your IT security, configuration management, and compliance requirements by bringing those virtual machines into Amazon EC2 as ready-to-use instances. You can also export imported instances back to your on-premises virtualization infrastructure, allowing you to deploy workloads across your IT infrastructure.

- To import your images, use the AWS CLI or other developer tools to import a virtual machine (VM) image from your VMware environment. If you use the VMware vSphere virtualization platform, you can also use the AWS Management Portal for vCenter to import your VM. As part of the import process, VM Import will convert your VM into an Amazon EC2 AMI, which you can use to run Amazon EC2 instances. Once your VM has been imported, you can take advantage of Amazon’s elasticity, scalability and monitoring via offerings like Auto Scaling, Elastic Load Balancing and CloudWatch to support your imported images.

- You can export previously imported EC2 instances using the Amazon EC2 API tools. You simply specify the target instance, virtual machine file format and a destination S3 bucket, and VM Import/Export will automatically export the instance to the S3 bucket. You can then download and launch the exported VM within your on-premises virtualization infrastructure.

You can import Windows and Linux VMs that use VMware ESX or Workstation, Microsoft Hyper-V, and Citrix Xen virtualization formats. And you can export previously imported EC2 instances to VMware ESX, Microsoft Hyper-V or Citrix Xen formats.-

NOTE: there is `no way` to directly download the `AmazonLinux2.iso` for Amazon Linux 2. but you can get the Amazon Linux 2 image for the specific virtualization platform of your choice. If you are using VMware, you can download the ESX image *.ova and for VirtualBox, you’ll get the *.vdi image file.



Problem:  to set up an automated solution that will provide a detailed report of all unencrypted EBS volumes of the company as well as to notify them if there is a newly launched EC2 instance which uses an unencrypted volume.

Solution: 
- Set up an AWS Config rule with a corresponding Lambda function on all the target accounts of the company. 
- Collect data from multiple accounts and AWS Regions using AWS Config aggregators. 
- Export the aggregated report to an S3 bucket then deliver the notifications using Amazon SNS.

NOTE: `AWS Systems Manager Configuration Compliance service` is more suitable for `verifying the patch compliance` of all your resources. NOT provide a detailed report of all unencrypted EBS volumes of the company. `AWS Config` is better.



An `AWS Config aggregator` is an `AWS Config` resource type that collects `AWS Config` configuration and compliance data from the following:
- Multiple accounts and multiple regions.
- Single account and multiple regions.
- An organization in AWS Organizations and all the accounts in that organization.




Problem: If the latency of the service increases more than the defined threshold then the deployment should be halted until the service has been fully recovered.

Solution: 
- Calculate the `average latency` using `Amazon CloudWatch metrics` that monitors the `Application Load Balancer`. 
- Associate a `CloudWatch alarm` with the `CodeDeploy deployment group`. 
- When latency increases beyond the defined threshold, it will automatically trigger an alarm that `automatically stops the on-going deployment`.

Reason:
- For your CodeDeploy operations, you can configure `a deployment group` to `stop a deployment` whenever `any CloudWatch alarm` you associate with `the deployment group` is activated.
- NOTE: You must grant CloudWatch permissions to your CodeDeploy service role to use this option.




Problem: There is a new requirement in which you have to implement an automated OS patching solution for all of the Windows servers hosted on-premises as well as in AWS Cloud. The AWS Systems Manager service should be utilized to automate the patching of your servers.

Solution:

1. Set up `a single IAM service role` for `AWS Systems Manager` to enable the service to execute the `STS AssumeRole operation`. Allow the generation of service tokens by registering the IAM role. Use the service role to perform `the managed-instance activation`.
  - Servers and virtual machines (VMs) in a hybrid environment require an IAM role to communicate with the Systems Manager service. The role grants AssumeRole trust to the Systems Manager service. You only need to create the service role for a hybrid environment once for each AWS account.

2. ** Then download and install the `SSM Agent` on the `hybrid servers` by using the `activation codes` and `activation IDs` that you obtained. Register the servers or virtual machines on-premises to the `AWS Systems Manager service`. In the SSM console, the hybrid instances will show with an `mi-` prefix. Apply the patches using the `Systems Manager Patch Manager`.



`Auto Scaling group` scheduled actions
- You can configure your `Auto Scaling group` to `scale based on a schedule`, you create `a scheduled action`. 
- The scheduled action tells Amazon EC2 Auto Scaling to perform a scaling action at specified times. 
- To create a scheduled scaling action, you specify the start time when the scaling action should take effect, and the new minimum, maximum, and desired sizes for the scaling action. 
- At the specified time, Amazon EC2 Auto Scaling updates the group with the values for minimum, maximum, and desired size specified by the scaling action.
- NOTE: You can create scheduled actions for scaling one time only or for scaling on a recurring schedule.



NOTE: there is NO `multiprocessing` option in `CodeBuild`.

NOTE: Upgrade the compute type of the build environment in CodeBuild pipeline with a higher memory, vCPUs, and disk space may help speed up the build time, it will only improve the performance of CodeBuild and not the entire pipeline. >> A better solution is to run the tasks in parallel in CodePipeline.



AWS CodePipeline Parallel actions

- In AWS CodePipeline, an action is part of the sequence in a stage of a pipeline. It is a task performed on the artifact in that stage. 

- ** Pipeline actions occur in a specified order, `in sequence` or `in parallel`, as determined in the configuration of the stage.

- If you manually create or edit a JSON file to create a pipeline or update a pipeline from the AWS CLI,
  - The default `runOrder` value for an action is `1`. The value must be a positive integer (natural number). You cannot use fractions, decimals, negative numbers, or zero. 
    - To specify `a serial sequence of actions`, use `the smallest number for the first action` and larger numbers for each of the rest of the actions in sequence. 
    - To specify `parallel actions`, use `the same integer for each action` you want to run in parallel.

- For example, if you want three actions to run in sequence in a stage, you would give the first action the runOrder value of 1, the second action the runOrder value of 2, and the third the runOrder value of 3. However, if you want the second and third actions to run in parallel, you would give the first action the runOrder value of 1 and both the second and third actions the runOrder value of 2.

- ** The numbering of serial actions do not have to be in strict sequence. For example, if you have three actions in a sequence and decide to remove the second action, you do not need to renumber the runOrder value of the third action. Because the runOrder value of that action (3) is higher than the runOrder value of the first action (1), it runs serially after the first action in the stage.




`Systems Manager Automation` simplifies `common maintenance and deployment tasks` of Amazon EC2 instances and other AWS resources. Automation enables you to do the following.

A `Systems Manager Automation document` defines the `Automation workflow` (the actions that Systems Manager performs on your managed instances and AWS resources). 

- ** `Automation` includes several `pre-defined Automation documents` that you can use to perform common tasks like restarting one or more Amazon EC2 instances or creating an Amazon Machine Image (AMI). 

- Documents use JavaScript Object Notation (JSON) or YAML, and they include steps and parameters that you specify. Steps run in sequential order.

- ** We can use `AWS Systems Manager Automation` to `preconfigure the AMI` by installing all of the required applications and software dependencies.





Problem:
Currently, they have to manually update their CloudFormation templates for every new available AMI of their application. This procedure is prone to human errors and entails a high management overhead on their deployment process.
Which of the following is the MOST suitable and cost-effective solution that the DevOps engineer should implement to automate this process?

Solution:
- Pull the new AMI IDs using `an AWS Lambda-backed custom resource` in the `CloudFormation template`. 
- Reference the AMI ID that the `custom resource` fetched in `the launch configuration` resource block.




`AWS Application Discovery`

- `AWS Application Discovery` Service helps you `plan your migration to the AWS cloud` by collecting usage and configuration data about `your on-premises servers`. 
  - `Application Discovery` Service is integrated with `AWS Migration Hub`, which `simplifies your migration tracking`. 
  - After performing discovery, you can view the discovered servers, group them into applications, and then `track the migration status of each application` from the `Migration Hub console`. 
  - The discovered data can be `exported` for analysis in Microsoft Excel or AWS analysis tools such as `Amazon Athena` and `Amazon QuickSight`.

- Using `Application Discovery Service APIs`, you can export the system performance and utilization data for your discovered servers. 
  - You can input this data into your cost model to compute the cost of running those servers in AWS. 
  - Additionally, you can export the network connections and process data to understand the network connections that exist between servers. 
  - This will help you determine the network dependencies between servers and group them into applications for migration planning.

- `Application Discovery Service` offers two ways of performing `discovery` and `collecting data` about your on-premises servers:

    - `Agentless discovery` can be performed by deploying the `AWS Agentless Discovery Connector (OVA file)` through your VMware vCenter. 
      - After the Discovery Connector is configured, it identifies virtual machines (VMs) and hosts associated with vCenter. 
      - The Discovery Connector collects the following static configuration data: Server hostnames, IP addresses, MAC addresses, and disk resource allocations.
      - Additionally, it collects the utilization data for each VM and computes average and peak utilization for metrics such as CPU, RAM, and Disk I/O. 
      - You can export a summary of the system performance information for all the VMs associated with a given VM host and perform a cost analysis of running them in AWS.

    - `Agent-based discovery` can be performed by deploying the `AWS Application Discovery Agen`t on each of your VMs and physical servers. 
      - The agent installer is available for both Windows and Linux operating systems. 
      - It collects static configuration data, detailed time-series system-performance information, inbound and outbound network connections, and processes that are running. 
      - You can export this data to perform a detailed cost analysis and to identify network connections between servers for grouping servers as applications.




Problem: Using `Systems Manager` on your `hybrid` machines

Solution:
- Servers and virtual machines (VMs) in a hybrid environment require `an IAM role` to communicate with the `Systems Manager` service. 
  - The role grants `AssumeRole` trust to the `Systems Manager` service. 
  - You only need to create `the service role` for a hybrid environment `once for each AWS account`.

- Users in your company or organization who will use `Systems Manager` on your `hybrid` machines must be granted permission in `IAM` to call the `SSM API`.




Problem: Using `CodeDeploy` to deploy to both Amazon `EC2` instances and `on-premises` instances.

Solution:

- An `on-premises` instance is any physical device that is not an Amazon EC2 instance that can run the `CodeDeploy agent` and connect to public AWS service endpoints. 
  - You can use `CodeDeploy` to simultaneously deploy an application to Amazon EC2 instances in the cloud and to desktop PCs in your office or servers in your own data center.

- To `register` an `on-premises` instance, you must use `an IAM identity` to authenticate your requests. You can choose from the following options for the `IAM identity` and `registration method` you use:
  - Use `an IAM User ARN` to authenticate requests
  - Use `an IAM Role ARN` to authenticate requests

- For maximum control over the authentication and registration of your on-premises instances, you can use the `register-on-premises-instance` command and periodically refreshed temporary credentials generated with the `AWS Security Token Service` (AWS STS). 
  - `A static IAM role` for the instance assumes the role of `these refreshed AWS STS credentials` to perform `CodeDeploy` deployment operations. 
  - This method is most useful when you need to register `a large number of instances`. 
  - It allows you to automate the registration process with CodeDeploy. 
  - You can use your own identity and authentication system to authenticate on-premises instances and distribute `IAM session credentials` from the service to the instances for use with CodeDeploy.

- ** Take note you `cannot directly attach an IAM Role to your on-premises servers`. 
  - You have to set up your on-premises servers as `“on-premises instances”` in `CodeDeploy` with `a static IAM Role` that your servers can assume.




Problem: Automate updating CloudFormation templates to map the latest AMI IDs of the ERP solution.

Solution: 
- Use `Systems Manager Parameter Store` in conjunction with `CloudFormation` to retrieve `the latest AMI IDs` for your template. 
- Call the `update-stack API` in `CloudFormation` in your template whenever you decide to update the `Amazon EC2 instances`.

NOTE:
- Parameters stored in Systems Manager are mutable. 
  - ** Any time you use a template containing Systems Manager parameters to create/update your stacks, CloudFormation uses the values for these Systems Manager parameters `at the time of the create/update operation`. 
  - ** So, as parameters are updated in Systems Manager, you can `have the new value of the parameter take effect` by just `executing a stack update operation`. 




Using `a NAT Gateway`, route credit card payment requests from the EC2 instances to `the external payment service`. 
- *** Associate `an Elastic IP address` to `the NAT Gateway`. 
- Update `the route table` associated with one or more of your private subnets to point `Internet-bound traffic` to the `NAT gateway`.




Dynamic references in a CloudFormation template

Dynamic references provide a compact, powerful way for you to specify external values that are stored and managed in other services, such as the Systems Manager Parameter Store, in your stack templates. When you use a dynamic reference, CloudFormation retrieves the value of the specified reference when necessary during stack and change set operations.

- CloudFormation currently supports the following dynamic reference patterns:
  - ssm, for plaintext values stored in AWS Systems Manager Parameter Store
  - ssm-secure, for secure strings stored in AWS Systems Manager Parameter Store
  - secretsmanager, for entire secrets or specific secret values that are stored in AWS Secrets Manager

- Some considerations when using dynamic references:
  - You can include up to 60 dynamic references in a stack template.
  - For transforms, such as AWS::Include and AWS::Serverless, AWS CloudFormation does not resolve dynamic references prior to invoking any transforms. Rather, AWS CloudFormation passes the literal string of the dynamic reference to the transform. Dynamic references (including those inserted into the processed template as the result of a transform) are resolved when you execute the change set using the template.
  - Dynamic references for secure values, such as ssm-secure and secretsmanager, are not currently supported in custom resources.



Traffic splitting uses an alias to switch between two functions in the backend allowing you to maintain a single instance of API Gateway + Lambda function.





Deploy your app using `Traffic shifting` with `AWS Lambda aliases`.

- By default, `an alias` points to `a single Lambda function version`. 
  - When the alias is updated to point to a different function version, `incoming request traffic in turn instantly points to the updated version`. 
    - This exposes that alias to any potential instabilities introduced by the new version. 
  
- ** To minimize this impact, you can implement the `routing-config parameter` of the `Lambda alias` that allows you to point to `two different versions` of the Lambda function and dictate `what percentage of incoming traffic is sent to each version`.

- With the introduction of `alias traffic shifting`, it is now possible to trivially implement `canary deployments of Lambda functions`. 
  - By updating additional version weights on an alias, invocation traffic is routed to the new function versions based on the weight specified. 
  - Detailed CloudWatch metrics for the alias and version can be analyzed during the deployment, or other health checks performed, to ensure that the new version is healthy before proceeding.




Problem: A DevOps Engineer has been assigned to develop an automated workflow to ensure that the required patches of all of their Windows EC2 instances are properly applied. It is of utmost importance that the EC2 instance reboots do not occur at the same time on all of their Windows instances in order to maintain their system uptime requirements. Any unavailability issues of their systems would likely cause a loss of revenue in the company since the customer transactions will not be processed in a timely manner.

Solution: 
- Set up `two Patch Groups` with unique tags that you will assign to all of your Amazon EC2 Windows Instances. 
- Associate `the predefined AWS-DefaultPatchBaseline baseline` on both `patch groups`. 
- ** Set up `two non-overlapping maintenance windows` and associate each with `a different patch group`. 
- Register `targets` with `specific maintenance windows` using `Patch Group tags`. 
- Assign the `AWS-RunPatchBaseline` document as a task within each maintenance window which has a different processing start time.




NOTE: It is valid to use `AWS WAF` with `CloudFront`, not only the `API Gateway`.



CloudWatch Logs Insights

- `CloudWatch Logs Insights` enables you to interactively search and analyze your log data in Amazon CloudWatch Logs. You can perform queries to help you quickly and effectively respond to operational issues. If an issue occurs, you can use CloudWatch Logs Insights to identify potential causes and validate deployed fixes.

`CloudWatch Logs Insights` includes a purpose-built query language with a few simple but powerful commands. CloudWatch Logs Insights provides sample queries, command descriptions, query autocompletion, and log field discovery to help you get started quickly. Sample queries are included for several types of AWS service logs.




NOTE: You can call the `EC2 CreateSnapshot API` directly as a target from `CloudWatch Events`. 
- No need to implement a custom Lambda function to call it.
- You can also run CloudWatch Events rules according to a schedule and create an automated snapshot of an existing Amazon Elastic Block Store (Amazon EBS) volume on a schedule. 
  - You can choose a fixed rate to create a snapshot at fixed intervals or use a cron expression to specify that the snapshot is made at a specific time of day.

NOTE: Alternatively, you can also use the `Amazon Data Lifecycle Manager (DLM)` for `EBS Snapshots`. 
- This service provides a simple, automated way to back up data stored on Amazon EBS volumes. 
- You can define backup and retention schedules for EBS snapshots by creating lifecycle policies based on tags. 
- With this feature, you no longer have to rely on custom scripts to create and manage your backups.





`EC2Rescue` can help you `diagnose` and `troubleshoot` problems on `Amazon EC2 Linux and Windows Server instances`. 
- ** You can run the tool manually or you can run the tool automatically by using `Systems Manager Automation` and the `AWSSupport-ExecuteEC2Rescue` document. 
- The `AWSSupport-ExecuteEC2Rescue` document is designed to perform a combination of `Systems Manager actions`, `AWS CloudFormation actions`, and `Lambda functions` that automate the steps normally required to use `EC2Rescue`.




`Systems Manager Automation` simplifies common maintenance and deployment tasks of Amazon EC2 instances and other AWS resources. Automation enables you to do the following:
- Build Automation workflows to configure and manage instances and AWS resources.
- Create custom workflows or use pre-defined workflows maintained by AWS.
- Receive notifications about Automation tasks and workflows by using Amazon CloudWatch Events.
- Monitor Automation progress and execution details by using the Amazon EC2 or the AWS Systems Manager console.




NOTE: In DynamoDB, you CANNOT add a `local secondary index` to an `already existing table`.
- ** So, you need to set up a new DynamoDB table with a Local Secondary Index that uses the DocumentName attribute with a different sort key. Migrate the data from the existing table to the new table.

NOTE: a `Global Secondary Index` in `a DynamoDB table` does NOT support `strong read consistency`.
- When you request `a strongly consistent read`, DynamoDB returns a response with `the most up-to-date data`, reflecting the updates from all prior write operations that were successful. 
- A strongly consistent read might not be available if there is a network delay or outage. 
- ** Strongly consistent reads are not supported on global secondary indexes.





`Resource-level permissions` refer to the ability to specify which resources users are allowed to perform actions on. 
- `Amazon EC2` has partial support for `resource-level permissions`. 
  - This means that for certain Amazon EC2 actions, you can control when users are allowed to use those actions based on conditions that have to be fulfilled, or specific resources that users are allowed to use. 
  - For example, you can grant users permissions to launch instances, but only of a specific type, and only using a specific AMI.




Problem: A JavaScript-based online salary calculator hosted on-premises is slated to be migrated to AWS. The application has no server-side code and is just composed of a UI powered by Vue.js and Bootstrap. Since the online calculator may contain sensitive financial data, adding HTTP response headers such as X-Content-Type-Options, X-Frame-Options and X-XSS-Protection should be implemented to comply with the Open Web Application Security Project (OWASP) standards.

Solution: 
- Host the application on an S3 bucket configured for website hosting. 
- Set up a CloudFront web distribution and set the S3 bucket as the origin with the origin response event set to trigger `a Lambda@Edge function`. 
- ** Add `the required security headers` in the `HTTP response` using the `Lambda function`.

Explanation:

`Security headers` are `a group of headers in the HTTP response` from a server that `tell your browser how to behave when handling your site’s content`. 
- For example, `X-XSS-Protection` is a header that Internet Explorer and Chrome respect to `stop the pages from loading when they detect cross-site scripting (XSS) attacks`.

- The following are some examples of security headers:
  - Strict Transport Security
  - Content-Security-Policy
  - X-Content-Type-Options
  - X-Frame-Options
  - X-XSS-Protection
  - Referrer-Policy

- ** You can set up a solution that uses `a simple single-page website`, hosted in `an Amazon S3 bucket` and using `Amazon CloudFront`. You can create `a new Lambda@Edge function` and associate it with your CloudFront distribution. Then configure `the origin response trigger` to execute the `Lambda@Edge function` add `the security headers` in the HTTP response.

- NOTE: Configuring `a custom Request and Response Behavior in CloudFront` is NOT enough to automatically add the required security headers to the HTTP response.



NOTE: Enable `sticky sessions` in the `Application Load Balancer` 
- ** just bind `user sessions` to `a specific target group`, thus, you’re still `persistently storing session data locally on the instance`. 
- This means that if `an instance fails`, you will most likely `lose the session data` that are stored on the failed instance.
- Furthermore, if the number of your web servers increases, as in a scale-up situation, traffic may be `unequally distributed` throughout the web servers because `active sessions may reside on certain servers`.




NOTE: ElastiCache offerings for In-Memory key/value stores include 
- ** ElastiCache for `Redis`, which can `support replication`, 
- ** ElastiCache for `Memcached` which does `NOT support replication`.




CloudFront :: Field-level encryption

You can already configure CloudFront to help enforce `secure end-to-end connections to origin servers` by using `HTTPS`. 
- `Field-level encryption` adds `an additional layer of security` along with `HTTPS` that lets you protect specific data throughout system processing so that `only certain applications` can see it. 
- `Field-level encryption` allows you to securely upload user-submitted sensitive information to your web servers. 
- The sensitive information provided by your clients is `encrypted at the edge closer to the user` and `remains encrypted throughout your entire application stack`, ensuring that only applications that need the data—and have the credentials to decrypt it—are able to do so.

To use `field-level encryption`, you configure your CloudFront distribution to specify `the set of fields in POST requests` that you want to be encrypted, and `the public key` to use to encrypt them. You can encrypt up to 10 data fields in a request.



CloudFront :: Improve performance by more caching objects from Origin

You can improve performance by `increasing the proportion of your viewer requests that are served from CloudFront edge caches instead of going to your origin servers` for content; that is, by `improving the cache hit ratio` for your distribution. 
- To increase your cache hit ratio, you can `configure your origin` to add `a Cache-Control max-age directive` to your `objects`, and specify `the longest practical value for max-age`. 
- The shorter the cache duration, the more frequently CloudFront forwards another request to your origin to determine whether the object has changed and, if so, to get the latest version.





The certificate issuer you must use depends on whether you want to require `HTTPS between viewers and CloudFront` or `between CloudFront and your origin`:

HTTPS between `viewers and CloudFront`
- You can use a certificate that was issued by `a trusted certificate authority (CA)` such as Comodo, DigiCert, Symantec, or other third-party providers.
- You can use a certificate provided by `AWS Certificate Manager (ACM)`

HTTPS between `CloudFront and a custom origin`
- If the origin is `NOT an ELB load balancer`, such as Amazon EC2, the certificate must be issued by `a trusted CA` such as Comodo, DigiCert, Symantec or other third-party providers.
- If your origin is `an ELB load balancer`, you can also use `a certificate provided by ACM`.

NOTE: If you’re using `your own domain name`, such as tutorialsdojo.com, you need to change several CloudFront settings. You also need to use an SSL/TLS certificate provided by `AWS Certificate Manager (ACM)`, or import a certificate from a third-party certificate authority into `ACM` or the `IAM certificate store`. Lastly, you should set the Viewer Protocol Policy to HTTPS Only in CloudFront.

NOTE: you CANNOT use an SSL/TLS certificate from a third-party certificate authority which was imported to S3.

NOTE: you CANNOT directly upload a self-signed certificate in your ALB.


---





## Monitoring and Logging

EC2 `Detailed Monitoring` feature simply sends the metric data for your instance to CloudWatch in `1-minute periods`.



ECS logging from `container` to `CloudWatch Logs`
- The type of information that is logged by the containers in your task depends mostly on their `ENTRYPOINT` command. 
- By default, the logs that are captured show the command output that you would normally see in an interactive terminal if you ran the container locally, which are the `STDOUT` and `STDERR` I/O streams. 
- * The `awslogs` log driver on ECS simply passes these logs from `Docker` to `CloudWatch Logs`.


`CloudWatch Logs subscription filter` integrated with `Amazon Kinesis` (=`the receiving source`)
- You can use subscriptions to get access to a real-time feed of log events from `CloudWatch Logs` and have it delivered to other services such as a `Amazon Kinesis stream`, `Amazon Kinesis Data Firehose stream`, or `AWS Lambda` for custom processing, analysis, or loading to other systems. 
  - To begin subscribing to log events, create `the receiving source`, such as `a Kinesis stream`, where the events will be delivered. 
  - `A subscription filter` defines the filter pattern to use for filtering which log events get delivered to your AWS resource, as well as information about where to send matching log events to.

* NOTE: `CloudWatch subscription filter` doesn’t directly support `SQS`. 
  - You should use a `Kinesis Data Stream`, `Kinesis Firehose` or `Lambda function`.

* NOTE: `CloudWatch Logs subscription` cannot be directly integrated with an `AWS Step Functions` application.

- Use case#1: implement a solution that will automatically terminate any instance in production which was manually logged into via SSH.
  - Solution: 
    - Set up `a CloudWatch Logs subscription` with `an AWS Lambda function` which is configured to add `a FOR_DELETION tag` to `the Amazon EC2 instance` that produced the `SSH login event`. 
    - Run `another Lambda function` every day using `the CloudWatch Events rule` to terminate all EC2 instances with the custom tag for deletion.



* NOTE: `Amazon CloudWatch Alarms` can only send notifications to `SNS` and `not SQS`.





Problem: Amazon Elasticsearch Service in an Audit account that collect all logs from other accounts

Solution:

You can load streaming data into your `Amazon Elasticsearch Service` domain from many different sources in AWS. 
- Some sources, like `Amazon Kinesis Data Firehose` and `Amazon CloudWatch Logs`, have built-in support for `Amazon ES`. 
- * Others, like `Amazon S3`, `Amazon Kinesis Data Streams`, and `Amazon DynamoDB`, use `AWS Lambda functions` as event handlers. 
  - `The Lambda functions` respond to new data by `processing it and streaming it` to your Amazon ES cluster.

You can use `subscriptions` to get access to a real-time feed of `log events` from `CloudWatch Logs` and have it delivered to other services such as an `Amazon Kinesis stream`, `Amazon Kinesis Data Firehose stream`, or `AWS Lambda` for custom processing, analysis, or loading to other systems.

** You can collaborate with `an owner of a different AWS account` and receive their `log events` on your AWS resources, such as `a Amazon Kinesis stream` (this is known as `cross-account data sharing`).
- A real-time stream of event data `across those accounts` can be assembled and delivered to the information security groups who can use Kinesis to attach the data to their existing security analytic systems. 

** NOTE: `Kinesis streams` are currently the ONLY resource supported as `a destination for cross-account CloudWatch Logs subscriptions`.




`Access logs` of `Elastic Load Balancer`

- `Elastic Load Balancing` provides `access logs` that capture detailed information about requests sent to your load balancer. 
  - Each log contains information such as the time the request was received, the client’s IP address, latencies, request paths, and server responses. 
  - You can use these access logs to analyze traffic patterns and troubleshoot issues.

- `Access logging` is an `optional` feature of Elastic Load Balancing that is `disabled by default`. 
  - After you enable access logging for your load balancer, Elastic Load Balancing captures the logs and stores them in `the Amazon S3 bucket` that you specify as compressed files. 
  - You can disable access logging at any time.

- Each access log file is automatically encrypted before it is stored in your `S3 bucket` and decrypted when you access it. 
  - You do not need to take any action; the encryption and decryption is performed transparently. Each log file is encrypted with a unique key, which is itself encrypted with a master key that is regularly rotated.



The Unified `CloudWatch Logs agent` can be installed on both AWS resources and on-premise resources.



Problem: To further improve their CI/CD systems, they need to implement a solution that
- `automatically detects and reacts to changes in the state of their deployments in AWS CodeDeploy`. 
- `Any changes must be rolled back automatically if the deployment process fails`, and 
- `a notification must be sent to the DevOps Team’s Slack channel for easy monitoring`.

Solution:
- Step#1: Set up `a CloudWatch Events rule` to monitor `AWS CodeDeploy operations` with `a Lambda function` as a target. 
  - Configure `the rule` to send out a message to the DevOps Team’s Slack Channel in the event that the deployment fails. 
    - You can use Amazon CloudWatch Events to detect and react to changes in the state of an instance or a deployment (an “event”) in your CodeDeploy operations. 
    - Then, based on rules you create, CloudWatch Events will invoke one or more target actions when a deployment or instance enters the state you specify in a rule.

- Step#2: Configure `AWS CodeDeploy` to use the `Roll back when a deployment fails` setting.

The following are some use cases:
- Use a Lambda function to pass a notification to a Slack channel whenever deployments fail.
- Push data about deployments or instances to a Kinesis stream to support comprehensive, real-time status monitoring.
- Use CloudWatch alarm actions to automatically stop, terminate, reboot, or recover Amazon EC2 instances when a deployment or instance event you specify occurs.



NOTE: CloudWatch Alarm can’t directly send a message to a Slack Channel. 

NOTE: A CodeDeploy agent is primarily used for deployment and not for sending custom messages to non-AWS resources such as a Slack Channel.

NOTE: Launching `a Kinesis data stream` is a more suitable option than just `a Lambda function` that will `accept the logs from other accounts and send them to Amazon ES`.



Scenario: The firm wants to automate remediation actions for issues relating to the health of its AWS resources by using the AWS Health Dashboard and the AWS Health API. They need to automatically detect any of their own IAM access key that is accidentally or deliberately listed on a public Github repository. Once detected, the IAM access key m

Solution: 
- Set up three Lambda functions in AWS Step Functions that deletes the exposed IAM access key, summarizes the recent API activity for the exposed key using CloudTrail and sends a notification to the IT Security team using Amazon SNS. 
- *** Create `a CloudWatch Events rule` with an `aws.health` event source and the `AWS_RISK_CREDENTIALS_EXPOSED` event to `monitor any exposed IAM keys from the Internet`. 
- Set the Step Functions as the target of the CloudWatch Events rule.

Explanation:
- *** AWS proactively monitors popular code repository sites for exposed AWS Identity and Access Management (IAM) access keys. 
  - On detection of an exposed IAM access key, `AWS Health` generates an `AWS_RISK_CREDENTIALS_EXPOSED CloudWatch Event`.
- In response to this event, you can set up an automated workflow deletes the exposed IAM Access Key, summarizes the recent API activity for the exposed key, and sends the summary message to an Amazon Simple Notification Service (SNS) Topic to notify the subscribers which are all orchestrated by an AWS Step Functions state machine.

NOTE: You have to use the `AWS Health API` instead of the `AWS Personal Health Dashboard`. 
  - The `AWS_RISK_CREDENTIALS_EXPOSED event` is only applicable from an `aws.health event source` and NOT from `an AWS Personal Health Dashboard rule`.




Problem: to automate infrastructure cost optimization across multiple shared environments and accounts such as the detection of low utilization of EC2 instances.

Solution: 
- ** Integrate `CloudWatch Events rule` and `AWS Trusted Advisor` to detect the EC2 instances with low utilization. 
- Create a trigger with `an AWS Lambda function` that filters out the reported data based on tags for each environment, department, and business unit. 
- Create a second trigger that will invoke another Lambda function to terminate the underutilized EC2 instances.

Explanation:

- `AWS Trusted Advisor` draws upon best practices learned from serving hundreds of thousands of AWS customers. Trusted Advisor inspects your AWS environment, and then makes recommendations when opportunities exist to save money, improve system availability and performance, or help close security gaps. All AWS customers have access to five Trusted Advisor checks. Customers with a Business or Enterprise support plan can view all Trusted Advisor checks.

- `AWS Trusted Advisor` is integrated with the Amazon CloudWatch Events and Amazon CloudWatch services. You can use Amazon CloudWatch Events to detect and react to changes in the status of Trusted Advisor checks. And you can use Amazon CloudWatch to create alarms on Trusted Advisor metrics for check status changes, resource status changes, and service limit utilization

- You can use Amazon CloudWatch Events to detect and react to changes in the status of Trusted Advisor checks. Then, based on the rules that you create, CloudWatch Events invokes one or more target actions when a check status changes to the value you specify in a rule. Depending on the type of status change, you might want to send notifications, capture status information, take corrective action, initiate events, or take other actions.




Problem: There is a new requirement to create a system that should automatically send all `AWS-scheduled maintenance notifications` to the Slack channel of the company. This will easily notify their IT Operations team if there are any `AWS-initiated changes` to their EC2 instances and other resources.

Solution: 
- ** Use a combination of `AWS Personal Health Dashboard` and `Amazon CloudWatch Events` to track the `AWS-initiated activities` to `your resources`. 
- Create an event using `CloudWatch Events` which can invoke an `AWS Lambda function` to send notifications to `the company's Slack channel`.

Explanation:

- `AWS Health` provides ongoing visibility into the state of your AWS resources, services, and accounts. The service gives you awareness and remediation guidance for resource performance or availability issues that affect your applications running on AWS. AWS Health provides relevant and timely information to help you manage events in progress. AWS Health also helps to be aware of and to prepare for planned activities. The service delivers alerts and notifications triggered by changes in the health of AWS resources, so that you get near-instant event visibility and guidance to help accelerate troubleshooting.

- All customers can use the `Personal Health Dashboard (PHD)`, powered by the `AWS Health API`. The dashboard requires no setup, and it’s ready to use for authenticated AWS users. 
  - Additionally, `AWS Support` customers who have `a Business or Enterprise support plan` can use the `AWS Health API` to integrate with in-house and third-party systems.

- You can use Amazon CloudWatch Events to detect and react to changes in the status of AWS Personal Health Dashboard (AWS Health) events. Then, based on the rules that you create, CloudWatch Events invokes one or more target actions when an event matches the values that you specify in a rule. Depending on the type of event, you can send notifications, capture event information, take corrective action, initiate events, or take other actions.

- ** Only `those AWS Health events that are specific to your AWS account and resources` are published to `CloudWatch Events`. This includes events such as EBS volume lost, EC2 instance store drive performance degraded, and all the scheduled change events. 

- ** In contrast, `Service Health Dashboard events` provide information about the regional availability of a service and are `not specific to AWS accounts`, so they are `NOT published to CloudWatch Events`.




`AWS OpsWorks Stacks Lifecycle Events`

- In `AWS OpsWorks Stacks Lifecycle Events`, each `layer` has `a set of five lifecycle events`, each of which has `an associated set of recipes` that are specific to the layer. When an event occurs on a layer’s instance, AWS OpsWorks Stacks automatically runs the appropriate set of recipes. To provide a custom response to these events, implement custom recipes and assign them to the appropriate events for each layer. AWS OpsWorks Stacks runs those recipes after the event’s built-in recipes.

- There are five lifecycle events namely: `Setup`, `Configure`, `Deploy`, `UnDeploy` and `Shutdown`. The Configure event occurs on all of the stack’s instances when one of the following occurs:
  - An instance enters or leaves the online state.
  - You associate an Elastic IP address with an instance or disassociate one from an instance.
  - You attach an Elastic Load Balancing load balancer to a layer, or detach one from a layer.

- ** The `Configure` event is therefore a good time to `regenerate configuration files`. For example, the HAProxy Configure recipes reconfigure the load balancer to accommodate any changes in the set of online application server instances.




NOTE: It is better to use `multiple separate CloudFormation templates` to handle `each logical part` of the architecture.
NOTE: So, You should create `a cross-stack reference` to export resources from one AWS CloudFormation stack to another.

- When you organize your AWS resources based on lifecycle and ownership, you might want to build a stack that uses resources that are in another stack.
- You can use cross-stack references to export resources from a stack so that other stacks can use them. 
- Stacks can use the exported resources by calling them using the `Fn::ImportValue` function.
- For example, you might have a network stack that includes a VPC, a security group, and a subnet. You want all public web applications to use these resources. By exporting the resources, you allow all stacks with public web applications to use them.

- To export resources from one AWS CloudFormation stack to another, create `a cross-stack reference`. 
  - `Cross-stack references` let you use a layered or service-oriented architecture. 
  - Instead of including all resources in a single stack, you create related AWS resources in separate stacks; then you can refer to required resource outputs from other stacks. 
- ** By restricting `cross-stack references` to `outputs`, you control the parts of a stack that are referenced by other stacks.

- To create a cross-stack reference, use the `Export` output field to flag the value of a resource output for export. Then, use the `Fn::ImportValue` intrinsic function to import the value.




Problem: You are instructed to set up a configuration management for all of your infrastructure in AWS. To comply with the company’s strict security policies, the solution should provide `a near real-time dashboard of the compliance posture of your systems with a feature to detect violations`.

Solution: Use `AWS Config` to record all configuration changes and store the data reports to `Amazon S3`. Use `Amazon QuickSight` to analyze the dataset.

- `AWS Config` provides you a visual dashboard to help you quickly spot non-compliant resources and take appropriate action. IT Administrators, Security Experts, and Compliance Officers can see a shared view of your AWS resources compliance posture.

- You can use `AWS Config rules` to evaluate the configuration settings of your AWS resources. When `AWS Config` detects that a resource violates the conditions in one of your rules, `AWS Config` flags the resource as noncompliant and sends a notification. `AWS Config` continuously evaluates your resources as they are created, changed, or deleted.

NOTE: the `Trusted Advisor` service is not suitable for `configuration management` and `automatic violation detection`. You should use `AWS Config` instead.


CloudTrail log file integrity validation

In `AWS CloudTrail`, enable the `log file integrity` feature on the `trail` that will automatically generate `a digest file` for `every log file` that CloudTrail delivers.
-  Verify `the integrity of the delivered CloudTrail files` using `the generated digest files`.

Validated log files are invaluable in security and forensic investigations. For example, a validated log file enables you to assert positively that the log file itself has not changed, or that particular user credentials performed specific API activity. The CloudTrail log file integrity validation process also lets you know if a log file has been deleted or changed, or assert positively that no log files were delivered to your account during a given period of time.

When you enable log file integrity validation, CloudTrail creates a hash for every log file that it delivers. Every hour, CloudTrail also creates and delivers a file that references the log files for the last hour and contains a hash of each. This file is called a digest file. CloudTrail signs each digest file using the private key of a public and private key pair. After delivery, you can use the public key to validate the digest file. CloudTrail uses different key pairs for each AWS region.

The digest files are delivered to the same Amazon S3 bucket associated with your trail as your CloudTrail log files. If your log files are delivered from all regions or from multiple accounts into a single Amazon S3 bucket, CloudTrail will deliver the digest files from those regions and accounts into the same bucket.

The digest files are put into a folder separate from the log files. This separation of digest files and log files enables you to enforce granular security policies and permits existing log processing solutions to continue to operate without modification. Each digest file also contains the digital signature of the previous digest file if one exists. The signature for the current digest file is in the metadata properties of the digest file Amazon S3 object.





Problem: The DevOps Team was assigned to come up with a solution to remediate CloudTrail from being disabled on some AWS accounts automatically.

Solution: 
- Use the `cloudtrail-enabled` `AWS Config managed rule` with `a periodic interval of 1 hour` to evaluate whether your AWS account enabled the AWS CloudTrail. 
- Set up `a CloudWatch Events rule` for `AWS Config rules compliance change`. 
- Launch `a Lambda function` that uses the AWS SDK and add the Amazon Resource Name (ARN) of the Lambda function as `the target in the CloudWatch Events rule`. 
- Once a `StopLogging` event is detected, the Lambda function will re-enable the logging for that trail by calling the `StartLogging` API on the resource ARN.

NOTE:
- * By default, `AWS Config` will `NOT automatically remediate the accounts that disabled its CloudTrail`. 
  - You must manually set this up using `a CloudWatch Events rule` and `a custom Lambda function` that calls the `StartLogging` API to enable CloudTrail back again. 
  
- ** Furthermore, the `cloudtrail-enabled` AWS Config managed rule is `ONLY available` for the `periodic` trigger type and NOT `Configuration` changes.




Problem: The Data Analytics team needs to determine `how the maintenance will affect their cluster` and `ensure that their Hadoop Distributed File System (HDFS) component can recover from any failure`.

Solution: 
- Create `an AWS EventBridge (Amazon CloudWatch Events) rule` for `AWS Health`. 
- Select `EC2 Service` and select `the Events` you want to get notified. 
- Set `the target` to `an Amazon SNS topic` that the Data Analytics team is subscribed to.

Explanation:

- You can use `AWS EventBridge` (Amazon CloudWatch Events) to detect and react to changes in the status of `AWS Personal Health Dashboard (AWS Health) events`. 
  - Then, based on the rules that you create, `CloudWatch Events` invokes one or more `target actions` when an event matches the values that you specify in a rule.

- Depending on the type of event, you can send notifications, capture event information, take corrective action, initiate events, or take other actions. 
  - In this scenario, you first need to create `an Events rule for AWS Health` and choose `the “EC2 service”` as well as specific categories of events or scheduled changes that are planned to your account.

- Then, you can then set the `Target` of this rule to `an SNS topic` on which the Data Analytics team will subscribe. 
  - For every event that matches this event rule, a notification will be sent to the subscribers of your SNS topic.






`AWS Systems Manager resource data sync` feature is primarily used in `AWS Systems Manager Inventory` to send `inventory data` (metadata from your managed nodes) collected from all of your `managed nodes` to a single `Amazon Simple Storage Service (Amazon S3) bucket`. 
- The `SSM resource data sync` will automatically update the centralized data in Amazon S3 when `new inventory data is collected`. 


`CloudWatch Metric Filter`: `Metric filters` define the terms and patterns to look for in `log data` as it is sent to CloudWatch Logs. 
- CloudWatch Logs uses these `metric filters` to `turn log data into numerical CloudWatch metrics` that you can graph or set an alarm on.
- you can only create a Metric filter for `CloudWatch log groups`. 




Problem: You have a fleet of 400 Amazon EC2 instances for your High Performance Computing (HPC) application. The fleet is configured to use target tracking scaling and is behind an ALB. You want to create a simple web page hosted on an S3 bucket that displays the status of the EC2 instances on the fleet. This web page will be updated whenever a new instance is launched or terminated. You are also required to keep a searchable log of these events so they can be reviewed later.

Solution: 
- Write your own `Lambda function` to `update the simple webpage on S3` and `send event logs to CloudWatch Logs`. 
- Create a `CloudWatch Events rule` to invoke the Lambda for `scale-in/scale-out events`.

Explanation:
- You can create `an AWS Lambda function` that logs the changes in state for an Amazon EC2 instance. 
- * You can create `a rule` that runs the function whenever there is a state transition or a transition to one or more states that are of interest. 
  - Be sure to assign proper permissions to your Lambda function to write to S3 and to send logs to CloudWatch Logs. 

- ** After creating `the Lambda function`, create `a rule on CloudWatch Events` that will `watch for the scale-in/scale-out events`. 
  - Then set a trigger to run your Lambda function which will then `update the S3 webpage` and `send logs to CloudWatch Logs`.



CodeCommit to trigger push event and publish message to an SNS topic. However, your CodeCommit trigger is not working as expected because:
- ** Ans: `Your CodeCommit repository` and `AWS SNS topic` MUST be on `the same region` for the triggers to work.

** NOTE: 
- * For `Amazon SNS topics`, you `do NOT need to configure additional IAM policies or permissions` if the Amazon SNS topic is created using `the same account` as `the CodeCommit repository`. 
- ** However this will `still not work` if the `SNS topic` and `CodeCommit repository` are `on a different region`.

NOTE: 
- Scenarios like notifying an external build system `require writing a Lambda function` to `interact with other applications`. 
- The `email` scenario simply `requires creating an Amazon SNS topic`.





S3 Server access logging
- `S3 Server access logging` is primarily used to provide `detailed records for the requests` that are made to a bucket. 
  - Each `access log record` provides details about `a single access request`, such as the requester, bucket name, request time, request action, response status, and an error code, if relevant.

** NOTE: It is more appropriate to use `CloudWatch or CloudTrail` to track the `S3 bucket policy changes`.
** NOTE: You can’t directly send the `S3 server access logs` to `CloudWatch logs`.




Problem: You want to be notified of `any S3 bucket policy changes` so that you can quickly identify and take action if any problem occurs.

Solution: 
- Create `a CloudTrail trail` that sends logs to `a CloudWatch Log group`. 

- Create `a CloudWatch Metric Filter` for `S3 bucket policy events` on `the log group`. 
  - you need to create a Metric to filter specific S3 events that change the bucket policy of your bucket. 

- Create `an Alarm` that will send you a notification whenever this metric threshold is reached.
  - In this case, you can create `an Amazon CloudWatch alarm` that is triggered when `an Amazon S3 API call` is made to `PUT or DELETE` bucket policy, bucket lifecycle, bucket replication, or to PUT a bucket ACL. 
  - For notification, you will then create `a CloudWatch Alarm` for `this Metric` with `a threshold of >=1` and set your email as a notification recipient.

** NOTE: you can’t use `CloudWatch Events` to filter your `log groups` directly. You need to do it through `CloudWatch Metric Filter`.




Problem: You want to be able to monitor the actions made on your S3 objects such as PUT, GET, and DELETE operations. You want to easily search and review these actions for auditing purposes.

Solution:
- Create `an AWS CloudTrail trail` to track and store your `S3 API call` logs to an Amazon S3 bucket. 
- Create `a Lambda function` that `logs data events` of your S3 bucket. 
- Trigger `this Lambda function` using `CloudWatch Events rule` for `every action taken on your S3 objects`. 
- View the logs on the `CloudWatch Logs group`.






Problem: Which of the following is the MOST appropriate monitoring set up that notifies you for any changes to your AWS accounts in an AWS Organization?

Solution:

- Monitor the compliance of your AWS Organizations using `AWS Config`. Launch `a new SNS Topic` or `Amazon CloudWatch Events` that will send alerts to you for `any changes`.

- Launch `a new trail` in `Amazon CloudTrail` to capture `all API calls` to your AWS Organizations, including calls from the AWS Organizations console. Also, track all code calls to the AWS Organizations APIs. Integrate CloudWatch Events and Amazon SNS to raise events when administrator-specified actions occur in an organization and configure it to send a notification.






You can create an `Amazon CloudWatch alarm` that monitors `an Amazon EC2 instance` and `automatically recovers the instance if it becomes impaired` due to an underlying `hardware failure` or` a problem that requires AWS involvement to repair`. 

Terminated instances cannot be recovered. `A recovered instance` is `identical` to `the original instance`, including the instance ID, private IP addresses, Elastic IP addresses, and all instance metadata. If the impaired instance is in a placement group, the recovered instance runs in the placement group.

When the `StatusCheckFailed_System` alarm is triggered, and the recover action is initiated, you will be notified by the Amazon SNS topic that you selected when you created the alarm and associated the recover action. 
- During instance recovery, the instance is migrated during an instance reboot, and any data that is in-memory is lost. 
- When the process is complete, information is published to the SNS topic you’ve configured for the alarm. 
- Anyone who is subscribed to this SNS topic will receive an email notification that includes the status of the recovery attempt and any further instructions. You will notice an instance reboot on the recovered instance.

Examples of problems that cause system status checks to fail include:
- Loss of network connectivity
- Loss of system power
- Software issues on the physical host
- Hardware issues on the physical host that impact network reachability

If your instance has a public IPv4 address, it retains the public IPv4 address after recovery.

NOTE: There is NO `built-in instance recovery failure feature` for Amazon EC2. 
- ** You have to use a combination of `CloudWatch` and `a Lambda function` to automatically recover the  EC2 instance from failure.
  - Here it is >>> Set up a CloudWatch Events rule to trigger an AWS Lambda function to start a new EC2 instance in an available Availability Zone when the instance status reaches a failure state. 
    - You must also configure an Amazon CloudWatch Events rule to monitor AWS Personal Health Dashboard (AWS Health) events for your instance. Then, you are notified of the results of automatic recovery actions for an instance.





Amazon Macie

- `Amazon Macie` is an ML-powered security service that helps you `prevent data loss` by automatically discovering, classifying, and protecting sensitive data stored in `Amazon S3`. 
  - Amazon Macie uses `machine learning` to recognize sensitive data such as personally identifiable information (PII) or intellectual property, assigns a business value, and provides visibility into where this data is stored and how it is being used in your organization.

- ** `Amazon Macie` continuously `monitors data access activity` for anomalies, and delivers alerts when it detects risk of `unauthorized access or inadvertent data leaks`. 
  - Amazon Macie has ability to `detect global access permissions` inadvertently being set on sensitive data, detect uploading of API keys inside source code, and verify sensitive customer data is being stored and accessed in a manner that meets their compliance standards.




NOTE: You can only configure `Amazon CloudWatch Events`, `Amazon SNS`, or `Amazon SQS` as notification targets for `the lifecycle hook` of `an Auto Scaling group`.
- you CANNOT use `AWS Lambda` as a notification target for `the lifecycle hook of an Auto Scaling group`.
  - BUT If you would like to trigger `AWS Lambda` function from `the lifecycle hook of an Auto Scaling group`, please do it through `Amazon CloudWatch Events`.
    - `the lifecycle hook of an Auto Scaling group` >> `Amazon CloudWatch Events` >> `AWS Lambda` function




---





## Policies and Standards Automation

Problem: If an IAM user has permission with `s3:PutObject` action on an `Amazon Simple Storage Service (Amazon S3) bucket` and got an `HTTP 403: Access Denied` error when uploading an object, then there might be an issue with `the bucket` or `VPC endpoint policy`.
- The Amazon S3 `bucket policy` is not properly configured
- The Amazon Virtual Private Cloud (Amazon VPC) `endpoint policy` is not properly configured




Problem: A company has a fleet of Linux and Windows servers which they use for their enterprise application. They need to implement `an automated daily check of each golden AMI they own` to monitor the latest `Common Vulnerabilities and Exposures (CVE)` using `Amazon Inspector`.

Solution: 
1. Use `AWS Step Functions` to launch an Amazon EC2 instance for each operating system from `the golden AMI` >> install the `Amazon Inspector agent` >> and `add a custom tag for tracking`. 
2. Configure `the Step Functions` to trigger the `Amazon Inspector assessment` for all instances `with the custom tag` you added right after the EC2 instances have booted up. 
3. Trigger `the Step Functions every day` using `an Amazon CloudWatch Events rule`.

Explanation:
- `Amazon Inspector` tests the network accessibility of your Amazon EC2 instances and the security state of your applications that run on those instances.
- `Amazon Inspector` assesses applications for exposure, vulnerabilities, and deviations from best practices. 
- After performing an assessment, `Amazon Inspector` produces `a detailed list of security findings` that is organized by level of severity.

NOTE: You have to install the `Amazon Inspector agent` first to the EC2 instance `before you can run the security assessments`.

NOTE: `CloudWatch Event bus` is primarily used to accept events from `AWS services`, `other AWS accounts`, and `PutEvents API calls`.




Problem: You are working as a DevOps Engineer for a leading pharmaceutical company. After the recent annual IT audit, several Amazon EC2 instances were discovered running an outdated operating system, and there were many security vulnerabilities left unpatched. To safeguard your systems from various cybersecurity attacks, mitigating these vulnerabilities as soon as possible is crucial. You are also required to record all of the changes to patch and association compliance statuses.

Solution:

Manage, record, and deploy the OS security patches of the Amazon EC2 instances using a combination of `AWS Systems Manager Patch Manager` and AWS `Config`.

Explanation:

`AWS Systems Manager Patch Manager` automates the process of patching managed instances with security-related updates. 
- For Linux-based instances, you can also install patches for non-security updates. 
- You can patch fleets of `Amazon EC2 instances` or your `on-premises servers and virtual machines (VMs)` by operating system type. 
  - This includes supported versions of Windows, Ubuntu Server, Red Hat Enterprise Linux (RHEL), SUSE Linux Enterprise Server (SLES), Amazon Linux, and Amazon Linux 2. 
- You can scan instances to see only a report of missing patches, or you can scan and automatically install all missing patches.

`Patch Manager` uses `patch baselines`, which include `rules for auto-approving patches within days of their release`, as well as a list of approved and rejected patches. 
- You can install patches on a regular basis by scheduling patching to run as `a Systems Manager maintenance window task`. 
- You can also install patches individually or to large groups of instances by using `Amazon EC2 tags`. (Tags are keys that help identify and sort your resources within your organization.) 
- You can add `tags` to your `patch baselines` themselves when you create or update them.

Since you are also required to `record all of the changes to patch and association compliance statuses`, you can use `AWS Config` to meet this requirement. 
- `AWS Config` is a service that enables you to assess, audit, and evaluate the configurations of your AWS resources. 
- Config `continuously monitors and records your AWS resource configurations` and allows you to automate the evaluation of recorded configurations against desired configurations.




`AWS Systems Manager Patch Manager`

- `AWS Systems Manager Patch Manager` automates the process of patching managed instances with security-related updates. 
  - * For `Linux-based instances`, you can also install patches for `non-security updates`. 
  - You can patch fleets of `Amazon EC2 instances` or `your on-premises servers and virtual machines (VMs)` by operating system type.

- `Patch Manager` uses `patch baselines`, which include rules for auto-approving patches within days of their release, as well as a list of approved and rejected patches. 
  - You can install patches on a regular basis by scheduling patching to run as a `Systems Manager Maintenance Window task`. 
    - You can also install patches individually or to large groups of instances by using `Amazon EC2 tags`. 
  - For each auto-approval rule that you create, you can specify `an auto-approval delay`. 
    - This delay is `the number of days to wait after the patch was released` before the patch is automatically approved for patching.

- `A patch group` is an optional means of organizing instances for patching. 
  - For example, you can create patch groups for different operating systems (Linux or Windows), different environments (Development, Test, and Production), or different server functions (web servers, file servers, databases). 
  - `Patch groups` can help you avoid deploying patches to the wrong set of instances. 
    - They can also help you avoid deploying patches before they have been adequately tested.

  - You create `a patch group` by using `Amazon EC2 tags`. 
    - ** Unlike other tagging scenarios across Systems Manager, a patch group must be defined with the tag key: `Patch Group`. 
    - After you create a patch group and tag instances, you can `register the patch group with a patch baseline`. 
    - By registering the patch group with a patch baseline, you ensure that the correct patches are installed during the patching execution.







Problem: A DevOps Engineer was instructed to set up a monitoring system that will notify the IT Operations team if there are any existing or new buckets that have public read or public write access. The Security Team will verify if the detected S3 buckets are indeed intended to be accessed publicly or not.

Solution:
- Create `s3-bucket-public-read-prohibited` and `s3-bucket-public-write-prohibited` managed rules in `AWS Config`. 
- Configure `a CloudWatch Events rule` that sends a notification to `an SNS topic` when AWS Config detects `an S3 bucket ACL or policy violation`.





Problem: A DevOps Engineer was given the task to synchronize the patch baselines being used on-premises to all of the EC2 instances in your VPC, as well as to automate the patching schedule.

Solution:
- Install the `SSM Agent` to all of your Amazon EC2 instances. 
- Use `AWS Systems Manager Maintenance Windows` to automate the patching schedule of your resources. 
- Use `AWS Systems Manager Patch Manager` to manage and deploy the security patches of your EC2 instances 
  - ** based on `the patch baselines` from your `on-premises network`.




Problem: A company has a PROD, DEV, and TEST environment in its software development department, each contains hundreds of EC2 instances and other AWS services. 

DevOps engineer was instructed to immediately patch all of their affected EC2 instances in all the environments, except for the PROD environment. The EC2 instances in their PROD environment will only be patched after the initial patches have been verified to work effectively in their non-PROD environments. 

** Each environment also has different baseline patch requirements that you will need to satisfy.

Solution:
- `Tag` each instance based on its `environment, business unit, and operating system`. 
- Set up `a patch baseline` in `AWS Systems Manager Patch Manager` for `each environment`. 
- Categorize each Amazon EC2 instance based on its tags using `Patch Groups`. 
- Apply the required patches specified in the corresponding `patch baseline` to each `Patch Group`.




NOTE: `Oracle RAC` is supported via the deployment using Amazon EC2 only since `Amazon RDS and Aurora do not support it`. 
- Amazon RDS does not support certain features in Oracle such as Multitenant Database, Real Application Clusters (RAC), Unified Auditing, Database Vault and many more.


`Amazon Data Lifecycle Manager (DLM)` for `EBS Snapshots` provides a simple, automated way to back up data stored on `Amazon EBS volumes`. 
- You can define `backup and retention schedules` for EBS snapshots by creating lifecycle policies based on `tags`. 
- With this feature, you no longer have to rely on custom scripts to create and manage your backups.



NOTE: you can use the `approved-amis-by-id` AWS Config manage rule which checks whether running instances are using specified AMIs.




`AWS Trusted Advisor` is primarily used to check if your cloud infrastructure is in compliance with the best practices and recommendations across five categories: cost optimization, security, fault tolerance, performance, and service limits.
- NOTE: Their security checks for EC2 does `not cover the checking of individual AMIs` that are being used by your EC2 instances.





Problem: A multinational corporation has `multiple AWS accounts` that are consolidated using `AWS Organizations`. For security purposes, a new system should be configured that `automatically detects suspicious activities in any of its accounts`, such as SSH brute force attacks or compromised EC2 instances that serve malware. All of the gathered information must be centrally stored in its `dedicated security account` for audit purposes, and the events should be stored in an S3 bucket.

Solution:
- ** Automatically detect SSH brute force or malware attacks by enabling `Amazon GuardDuty` in `every account`. 
- Configure the `security account` as the `GuardDuty Administrator` for every member of the organization. 
- Set up a new `CloudWatch rule` in the security account. 
- Configure the rule to send all findings to `Amazon Kinesis Data Firehose`, which will push the findings to the `S3 bucket`.

Explanation:

`Amazon GuardDuty` is `a threat detection service that continuously monitors for malicious activity and unauthorized behavior to protect your AWS accounts and workloads`. 
- With the cloud, the collection and aggregation of account and network activities is simplified, but it can be time consuming for security teams to continuously analyze event log data for potential threats. 
- With GuardDuty, you now have an intelligent and cost-effective option for continuous threat detection in the AWS Cloud. 
  - The service uses `machine learning, anomaly detection, and integrated threat intelligence` to identify and prioritize potential threats. 
- GuardDuty analyzes tens of billions of events across multiple AWS data sources, such as AWS CloudTrail, Amazon VPC Flow Logs, and DNS logs. 
  - With a few clicks in the AWS Management Console, GuardDuty can be enabled with no software or hardware to deploy or maintain. 
- By integrating with AWS CloudWatch Events, GuardDuty alerts are actionable, easy to aggregate across `multiple accounts`, and straightforward to push into existing event management and workflow systems

`GuardDuty` makes enablement and management across multiple accounts easy. 
- Through the multi-account feature, all member accounts findings can be aggregated with `a GuardDuty administrator` account. 
- This enables security team to manage all GuardDuty findings from across the organization in one single account. 
- The aggregated findings are also available through CloudWatch Events, making it easy to integrate with an existing enterprise event management system.

`GuardDuty` can also detect `compromised EC2 instances` serving malware or mining bitcoin. 
- It also monitors AWS account access behavior for signs of compromise, such as unauthorized infrastructure deployments, like instances deployed in a region that has never been used, or unusual API calls, like a password policy change to reduce password strength.




NOTE: `AWS Inspector` is just an automated security assessment service that helps `improve the security and compliance of applications deployed on AWS`. 
- ** It will not detect possible `malicious activity on your instances` >> Use `Amazon GuardDuty` instead.





Problem: To have better visibility, you want to capture all events in the CodeCommit repository, such as cloning a repo or creating a branch on a single location for you to review.

Solution:
- Create `a Lambda function` that sends event logs to `AWS CloudWatch Logs` and set a trigger by selecting `the CodeCommit repository`. 
- On `CodeCommit`, go to the repository settings and select the `Lambda function` on the `Triggers` list when creating a new trigger.


---





## Incident and Event Response

`Default` Network ACL is configured to ALLOW ALL traffic.



`Amazon Kinesis Data Analytics` 
- is the easiest way to `transform and analyze streaming data in real time` using `Apache Flink`, an open-source framework and engine for processing data streams. 
- `Amazon Kinesis Data Analytics` simplifies building and managing `Apache Flink` workloads and allows you to easily integrate applications with other AWS services.



`DynamoDB Streams Kinesis Adapter`
- Using the `Amazon Kinesis Adapter` is the recommended way to consume streams from `Amazon DynamoDB`. 

- The `DynamoDB Streams API` is intentionally similar to that of `Kinesis Data Streams`, a service for real-time processing of streaming data at a massive scale. 
  - In both services, data streams are composed of shards, which are containers for stream records. Both services’ APIs contain `ListStreams`, `DescribeStream`, `GetShards`, and `GetShardIterator` operations. 
  - (Although these DynamoDB Streams actions are similar to their counterparts in Kinesis Data Streams, they are not 100 percent identical.)

- You can write applications for Kinesis Data Streams using the `Kinesis Client Library (KCL)`. 
  - The `KCL` simplifies coding by providing useful abstractions above the low-level Kinesis Data Streams API.

- ** As `a DynamoDB Streams user`, you can use the design patterns found within the `KCL` to process `DynamoDB Streams` shards and stream records. 
  - ** To do this, you use `the DynamoDB Streams Kinesis Adapter`. 
  - `The Kinesis Adapter` implements `the Kinesis Data Streams interface` 
    - *** so that the `KCL` can be used for consuming and processing records from `DynamoDB Streams`.



`AWS Systems Manager Configuration Compliance` service is primarily used to scan your fleet of managed instances for `patch compliance` and `configuration inconsistencies`. 



`Amazon Inspector` is just `an automated security assessment` service that helps improve the security and compliance of applications deployed on AWS.




Lambda Validation in CodeDeploy to Lambda function (Start >> [BeforeAllowTraffic] >> AllowTraffic >> [AfterAllowTraffic] >> End)

Problem: You are developing a mobile news homepage that curates several news sources to a single page. The app is mainly composed of several Lambda functions configured as a deployment group on AWS CodeDeploy. For each new app version, you need to test all APIs before fully deploying it to production. The APIs are using a set of AWS Lambda validation scripts. You want the ability to check the APIs during deployments and be notified for any API errors as well as automatic rollback if the validation fails.

Solution:
- Define your `Lambda validation scripts` on the `AppSpec lifecycle hook` `during deployment` (not after) to run the validation using `test traffic` and trigger `a rollback` if checks fail.
- Configure your `Lambda validation scripts` to run `during deployment` (not after) and configure `a CloudWatch Alarm` that will trigger a rollback when the function validation fails.
- Associate `an AWS CloudWatch Alarm` to `your deployment group` that can send `a notification to an AWS SNS topic` when `threshold for 5xx is reached` on CloudWatch.

Reason:
- You can use `CloudWatch Alarms` to track `metrics on your new deployment` and you can set` thresholds for those metrics` in your `Auto Scaling groups` being managed by `CodeDeploy`. 
- This can invoke `an action` if the `metric` you are tracking crosses the `threshold` for a defined period of time. 
- You can also monitor metrics such as `instance CPU utilization`, `Memory utilization` or `custom metrics` you have configured. 
- If the alarm is activated, CloudWatch initiates actions such as `sending a notification to Amazon Simple Notification Service`, `stopping a CodeDeploy deployment`, or `changing the state of an instance`. 
- You will also have the option to automatically `roll back a deployment` when a deployment fails or when a CloudWatch alarm is activated. 
- `CodeDeploy` will redeploy `the last known working version of the application` when it rolls back.

- ** The `BeforeAllowTraffic` and `AfterAllowTraffic` lifecycle hooks of the `AppSpec.yaml` file allows you to use `Lambda functions to validate the new version task set` using the test traffic during the deployment. 
- For example, a Lambda function can serve traffic to the test listener and track metrics from the replacement task set. 
- ** If `rollbacks` are configured, you can configure `a CloudWatch alarm` that `triggers a rollback` when the validation test in your Lambda function fails.




Lambda Validation in CodeDeploy to ECS application (Start >> [BeforeInstall] >> Install >> [AfterInstall] >> AllowTestTraffic >> [** AfterAllowTestTraffic **] >> [BeforeAllowTraffic] >> AllowTraffic >> [AfterAllowTraffic] >> End)

You can use a Lambda function to validate part of the deployment of an updated Amazon ECS application. During an Amazon ECS deployment with validation tests, CodeDeploy can be configured to use a load balancer with two target groups: one production traffic listener and one test traffic listener. To add a validation test, you first implement the test in a Lambda function. Next, in your deployment AppSpec file, you specify the Lambda function for the lifecycle hook you want to test. If a validation test fails, the deployment stops, it is rolled back, and marked as failed. If the test succeeds, the deployment continues to the next deployment lifecycle event or hook.

** The content in the ‘hooks’ section of the AppSpec file varies, depending on the compute platform for your deployment. The ‘hooks’ section for an EC2/On-Premises deployment contains mappings that link deployment lifecycle event hooks to one or more scripts. The ‘hooks’ section for a Lambda or an Amazon ECS deployment specifies Lambda validation functions to run during a deployment lifecycle event. If an event hook is not present, no operation is executed for that event.

** When the deployment starts, the deployment lifecycle events start to execute one at a time. Some lifecycle events are hooks that only execute Lambda functions specified in the AppSpec file. An AWS Lambda hook is one Lambda function specified with a string on a new line after the name of the lifecycle event. On the AfterAllowTestTraffic hook, you can specify Lambda functions that can validate the deployment using the test traffic

For example, a Lambda function can serve traffic to the test listener and track metrics from the replacement task set. If rollbacks are configured, you can configure a CloudWatch alarm that triggers a rollback when the validation test in your Lambda function fails. After the validation tests are complete, one of the following occurs:
- If validation fails and rollbacks are configured, the deployment status is marked Failed and components return to their state when the deployment started.
- If validation fails and rollbacks are not configured, the deployment status is marked Failed and components remain in their current state.
- If validation succeeds, the deployment continues to the BeforeAllowTraffic hook.

** To sum, Create your `validation scripts` in `AWS Lambda` and define them on the `AfterAllowTestTraffic` lifecycle hook of the `AppSpec.yaml` file. 
- The functions can validate the deployment using `the test traffic` and `rollback` if the tests fail.

NOTE: because in the `BeforeAllowTraffic` lifecycle hook, validation has already succeeded which defeats its purpose. 
- You have to use this hook to perform `additional actions before allowing production traffic to flow` to the `new task version`.

NOTE: because in the `AfterAllowTraffic` lifecycle hook, `production traffic` is rerouted from the old task set `to the new task set`. 
- You want to verify the application before opening it to production traffic, and `not after`.




Problem: There is a new mandate by the IT Security team that only a trusted group of administrators can change the objects in the bucket including the object permissions. The team instructed a DevOps Engineer to develop a solution that provides `near real-time`, automated checks to monitor their data.

Solution: 
- Set up `an Amazon S3 data events` in `CloudTrail` to track `any object changes`. 
- Create a rule to run your `Lambda function` in response to `an Amazon S3 data event` that checks if the user who initiated the change is an administrator.

Explanation:

- When an event occurs in your account, `CloudTrail` evaluates whether the event matches the settings for your `trails`. 
  - Only `events` that match your trail settings are delivered to your `Amazon S3 bucket` and `Amazon CloudWatch Logs log group`.

- You can configure your `trails` to log the following:

  1. `Data events`: 
    - These events provide insight into `the resource operations` performed on or within a resource. 
    - These are also known as `data plane operations`.
    - Data events are often high-volume activities.
    - Example data events include:
      - Amazon S3 object-level API activity (for example, GetObject, DeleteObject, and PutObject API operations)
      - AWS Lambda function execution activity (the Invoke API)
    - Data events are `disabled by default` when you create a trail. 
      - To record CloudTrail data events, you must explicitly add the supported resources or resource types for which you want to collect activity to a trail.
  
  2. `Management events`: 
    - Management events provide insight into `management operations` that are performed on resources in your AWS account. 
    - These are also known as `control plane operations`. 
    - Management events can also include `non-API events` that occur in your account. 
      - For example, when `a user logs in` to your account, CloudTrail logs the `ConsoleLogin` event.

- You can configure multiple trails differently so that the trails process and log only the events that you specify. 
  - For example, one trail can log read-only data and management events, so that all read-only events are delivered to one S3 bucket. 
    - Another trail can log only write-only data and management events, so that all write-only events are delivered to a separate S3 bucket.
  - You can also configure your trails to have one trail log and deliver all management events to one S3 bucket, 
    - and configure another trail to log and deliver all data events to another S3 bucket.

- `Data events` provide insight into the resource operations performed on or within a resource. These are also known as data plane operations. 

** NOTE: Although this type of trigger will cause the `AWS Config` to run evaluations when `there is any change in Amazon S3`, the use of `Amazon S3 Data Event` is still a more preferred way to track the changes in `near real-time`. 

** NOTE: You can’t directly use `Step Functions` in conjunction with `AWS Config`.




Problem: You want to be notified for `AWS Trusted Advisor` recommendation as frequently as possible.

Solution:
- Write `a Lambda function` that runs `daily` to `refresh AWS Trusted Advisor via API` and then publish a message to `an SNS Topic` to notify the subscribers based on the results.
- Write `a Lambda function` that runs `daily` to `refresh AWS Trusted Advisor changes via API` and send results to `CloudWatch Logs`. Create a CloudWatch Log metric and have it send an alarm notification when it is triggered.
- ** Use `CloudWatch Events` to monitor `Trusted Advisor checks` and set a trigger to send `an email` using `SNS` to notify you about the results of the check.

Explanation:
You can use `Amazon CloudWatch Events` to detect and react to changes in `the status of Trusted Advisor checks`.

NOTE: the `CloudWatch Events` integration for sending email notification should use `SNS`, NOT `SES`.

NOTE: Enable `the built-in Trusted Advisor notification feature` to automatically receive notification emails which include `the summary of savings estimates` along with Trusted Advisor `check results` >> the notification will be sent on `a weekly basis only`, which can already incur a lot of cost by the time it notifies.




You can configure rules in `Amazon CloudWatch Events` to alert you to changes in `AWS OpsWorks Stacks` resources, and direct CloudWatch Events to take actions based on event contents.

The `initiated_by` field is only populated when the instance is in the requested, terminating, or stopping states. The `initiated_by` field can contain one of the following values.

- `user` – A user requested the instance state change by using either the API or AWS Management Console.

- `auto-scaling` – The AWS OpsWorks Stacks `automatic scaling` feature initiated the instance state change.

- `auto-healing` – The AWS OpsWorks Stacks `automatic healing` feature initiated the instance state change.
  - AWS OpsWorks restarts an instance that is considered unhealthy or unable to communicate with the service endpoint.




Problem: You want to be notified when `any public S3 bucket have the wrong permissions` and have it `automatically changed if needed`.

Solution:

1. Set up `a custom AWS Config rule` that checks public S3 buckets permissions. Then, send `a non-compliance notification` to your `subscribed SNS topic`.

2. Set up `a custom AWS Config rule` to check public S3 buckets permissions and have it send an event on `CloudWatch Events` about policy violations. Have CloudWatch Events trigger `a Lambda Function` to `update the S3 bucket permission`.

3. ** Utilize `CloudWatch Events` to monitor `Trusted Advisor security recommendation results` and then set a trigger to send `an email using SNS` to notify you about the results of the check.

NOTE: There is NO `default remediation action` to `update the permissions on the public S3 bucket`.

NOTE: `Trusted Advisor` ONLY `sends the summary notification every week` so this won’t notify you immediately about your non-compliant resources.






AWS Config now includes `remediation capability` with AWS Config rules. 
- This feature gives you the ability to associate and execute remediation actions with AWS Config rules to address noncompliant resources. 
- You can choose from a list of available remediation actions. 
  - For example, you can create an AWS Config rule to check that your Amazon S3 buckets do not allow public read access. 
  - You can then associate a remediation action to disable public access for noncompliant S3 buckets.

NOTE: Any custom remediation actions in `AWS Config` should be used in conjunction with `AWS Systems Manager Automation documents`, NOT AWS Lambda.





NOTE: The automated notification on `AWS trusted Advisor` is only sent on `a weekly basis`, which can be quite long if you are concerned about security issues.
- So, instead, You can also write `a scheduled Lambda function` to check `Trusted Advisor` `regularly`.
- You can specify a fixed rate (for example, execute a Lambda function every hour or 15 minutes), or you can specify a Cron expression using CloudWatch Events. 
- You can retrieve and refresh Trusted Advisor results programmatically. 
- ** The `AWS Support` service enables you to write applications that interact with `AWS Trusted Advisor`.





Problem: Use AWS Shield Advanced to enable enhanced DDoS attack detection and monitoring for application-layer traffic of the company's AWS resources. Ensure that every security group in the VPC only allows certain ports and traffic from authorized servers or services. Protect your origin servers by putting it behind a CloudFront distribution.

Solution:

- Set up `AWS WAF rules` that identify and block common DDoS request patterns to effectively mitigate a DDoS attack on the company's cloud infrastructure. 

- Ensure that the `Network Access Control Lists (ACLs)` only allow the required ports and network addresses in the VPC.

- Use `AWS Shield Advanced` to enable enhanced DDoS attack detection and monitoring for application-layer traffic of the company's AWS resources. 

- Ensure that every `security group` in the VPC only allows certain ports and traffic from authorized servers or services. Protect your origin servers by putting it behind a CloudFront distribution.





---





## High Availability, Fault Tolerance, and Disaster Recovery

To create a custom AMI, launch an Elastic Beanstalk platform AMI in Amazon EC2, customize the software and configuration to your needs, and then stop the instance and save an AMI from it.


Create a CloudFront web distribution to cache images stored in the S3 bucket.


Use DynamoDB Accelerator (DAX) to cache repeated read requests on the web application in case that an Amazon DynamoDB table stores all of the data of the application.

- Amazon DynamoDB is designed for scale and performance. In most cases, the DynamoDB response times can be measured in single-digit milliseconds. However, there are certain use cases that require response times in `microseconds`. For these use cases, DynamoDB Accelerator (DAX) delivers fast response times for accessing eventually consistent data.

- `DAX` is a DynamoDB-compatible `caching service` that enables you to benefit from fast `in-memory` performance for demanding applications.




Amazon RDS Read Replicas
- provide BOTH `enhanced performance` and `durability` for database (DB) instances.
  - For `enhanced performance`, You can create one or more replicas of a given source DB Instance and serve high-volume application `read` traffic from multiple copies of your data, thereby increasing aggregate read throughput.
  - For `enhanced durability`, Read replicas can also be promoted when needed to become standalone DB instances.
    - It can provide a complementary availability mechanism to `Amazon RDS Multi-AZ Deployments`.
    - ** You can also replicate DB instances `across AWS Regions` as part of your disaster recovery strategy which is `NOT available with Multi-AZ Deployments` since this is only applicable in a single AWS Region. 

NOTE: `Read Replica` is still the `best choice` for providing `a high RTO and RPO` for `disaster recovery`.
- `a cross-region snapshot` doesn’t provide a high RPO compared with a Read Replica since the snapshot takes significant time to complete.


* NOTE: an `ELB` CANNOT distribute traffic to `different AWS Regions`



** NOTE: In `RDS`, because the scope of the `Multi-AZ deployments` is `bound to a single AWS Region` only. You `CANNOT host your standby DB instance` to `another AWS Region`. 
  - ** For cross-region disaster recovery, you should either use `Read Replicas` or `cross-region snapshots` instead.



NOTE: `Network Load Balancer` does NOT support `weighted target groups`, unlike the `Application Load Balancer`.



NOTE: NO `Canary routing option` in an `Application Load Balancer`.
- One weight little and another weight a lot. The first one is for testing new version before deployment 100% if everything is OK.



`Canary deployment` in serverless services is found in `Lambda function` and `API Gateway`

- `AWS Lambda function`
  - You can create `one or more aliases` for `your AWS Lambda function`. 
    - A Lambda alias is like a pointer to a specific Lambda function version. 
    - Each alias has a unique ARN. 
    - An alias can only point to a function version, not to another alias. 
    - You can update an alias to point to a new version of the function. 
  - * Use `routing configuration` on `an alias` to send `a portion of traffic` to `a second function version`.
    - For example, you can reduce the risk of deploying a new version by configuring the alias to send most of the traffic to the existing version, and only a small percentage of traffic to the new version. 
    - You can point `an alias` to `a maximum of two` Lambda function versions. (Canary deployment)\
  - Ex. Set up one AWS Lambda Function Alias that points to both the current and new versions. Route 20% of incoming traffic to the new version and once it is considered stable, update the alias to route all traffic to the new version.

- `API Gateway`
  - In API Gateway, you create `a canary release deployment` when deploying the API with canary settings as an additional input to the deployment creation operation.
  - Ex. Set up a canary deployment in Amazon API Gateway that routes 20% of the incoming traffic to the canary release. Promote the canary release to production once the initial tests have passed.




NOTE: you cannot create a RDS snapshot using the `Amazon RDS scheduled instance lifecycle events`.

NOTE: The `AWS Trusted Advisor` doesn’t provide any information regarding `AWS-initiated RDS events`. You should use the `AWS Health API` instead.


Problem: 
- A popular e-commerce website which has customers across the globe is hosted in the us-east-1 AWS region with a backup site in the us-west-1 region. Due to an unexpected regional outage in the us-east-1 region, the company initiated their disaster recovery plan and turned on the backup site. However, they discovered that the actual failover still entails several hours of manual effort to prepare and switch over the database. They also noticed that the database is missing up to three hours of data transactions when the regional outage happened.

- Which of the following solutions should the DevOps engineer implement which will improve the RTO and RPO of the website for the cross-region failover?

Solution:
- Use Step Functions with 2 Lambda functions that call the `RDS API` to `create a snapshot` of the database, create `a cross-region snapshot copy`, and `restore the database instance from a snapshot in the backup region`. 
  1. Use `CloudWatch Events` to trigger the function to `take a database snapshot every hour`. 
  2. Set up `an SNS topic` that will receive published messages from `AWS Health API`, RDS availability and other events that will trigger the Lambda function to create `a cross-region snapshot copy`. 
  3. During failover, Configure the Lambda function to `restore the database from a snapshot in the backup region`.





`AWS Elastic Beanstalk` deployment options: All at once, Rolling, Rolling with additional batch, and Immutable

- With `rolling` deployments, 
  - Elastic Beanstalk splits the environment’s EC2 instances into batches and deploys the new version of the application to one batch at a time, leaving the rest of the instances in the environment running the old version of the application. 
  - During a rolling deployment, some instances serve requests with the old version of the application, while instances in completed batches serve other requests with the new version.
  - * this policy will deploy the new version in batches where `each batch is taken out of service during the deployment phase`, `reducing your environment’s capacity` by the number of instances in a batch.

- With `rolling deployment with an additional batch`,
  - ** To maintain `full capacity` during deployments, you can configure your environment to launch a new batch of instances before taking any instances out of service. 
  - This option is known as a rolling deployment with an additional batch.
  - *  When the deployment completes, Elastic Beanstalk `terminates the additional batch of instances`.

- `Immutable` deployments 
  - perform an immutable update to launch `a full set of new instances` running `the new version of the application` in `a separate Auto Scaling group`, alongside the instances running the old version. 
  - `Immutable` deployments can prevent issues caused by `partially completed rolling deployments`. 
  - If the new instances don’t pass health checks, Elastic Beanstalk terminates them, leaving the original instances untouched.



NOTE: `Immutable` deployments perform an immutable update to launch `a full set of new instances running the new version` of the application in `a separate Auto Scaling group`, alongside the instances running the old version.
- `Immutable` deployments can prevent issues caused by partially completed rolling deployments > caused by `Rolling with additional batch policy`
- Rollback by: If the new instances don’t pass health checks, Elastic Beanstalk terminates them, leaving the original instances untouched.

NOTE: `Rolling with additional batch policy` for application deployments
- ** this maintain full capacity during deployments
- However, this type of configuration could `potentially cause partially completed` rolling deployments. 
- The new batch of instances is within the same Auto Scaling group and not in a new one.
- The `rollback process is also cumbersome` to implement unlike the `Immutable` or `Blue/Green deployment`.

NOTE: Although using the `blue/green deployment` configuration is an ideal option than `immutable`, 
- *** keeping the old environment running is not recommended since it `entails a significant cost` than `immutable`. 
  - Take note that the scenario asks for `the most cost-effective solution`, which is why the old environment should be deleted.
- Rollback by: Swap URL

NOTE: `AWS Elastic Beanstalk` provides support `for running Amazon Relational Database Service (Amazon RDS) instances in your Elastic Beanstalk environment`. 
- This works great for `development` and `testing` environments. 
- ** However, it isn’t ideal for a `production` environment because it ties the lifecycle of the database instance to the lifecycle of your application’s environment.




Problem: A company is planning to deploy a new version of their legacy application in AWS which is deployed to an Auto Scaling group of EC2 instances with an Application Load Balancer in front. To avoid any disruption of their services, they need to implement canary testing first before all of the traffic is shifted to the new application version.

Solution: Prepare `another stack` that consists of `an Application Load Balancer` and `Auto Scaling group` which contains the new application version for `blue/green environments`. Create `weighted Alias A records` in `Route 53` for the `two Application Load Balancers` to adjust the traffic.

Reason:

- The purpose of `a canary deployment` is to reduce the risk of deploying a new version that impacts the workload. 
  - The method will incrementally deploy the new version, making it visible to new users in a slow fashion. 
  - As you gain confidence in the deployment, you will deploy it to replace the current version in its entirety.

- To properly implement the canary deployment, you should do the following steps:
  - Use a router or load balancer that allows you to send `a small percentage of users` to `the new version`.
  - Use a dimension on your KPIs to indicate which version is reporting the metrics.
  - Use the metric to measure the success of the deployment; this indicates whether the deployment should continue or rollback.
  - Increase the load on the new version until either all users are on the new version or you have fully rolled back.


NOTE: When they say about `Legacy web appliction`, `Amazon API Gateway` service is NOT appropriate to be used because it is only applicable for APIs, not full-fledged web applications.

** NOTE: You can only integrate `a Network Load Balancer` to your `Amazon API Gateway`.

NOTE: You can’t use `CloudFront` to adjust the weight of the incoming traffic to your application. You should use `Route 53` instead.




Problem: Using CloudFormation to upgrade the `RDS instance` to the latest `major` version of MySQL database in `a Multi-AZ deployments` configuration. It is of utmost importance to ensure minimal downtime when doing the upgrade to avoid any business disruption.

Solution:
1. In the `AWS::RDS::DBInstance` resource type in the `CloudFormation template`, update the `EngineVersion` property to the latest MySQL database version. 
2. Create `a second application stack` and launch `a new Read Replica` with the same properties as the primary database instance that will be upgraded. 
3. Finally, perform an `Update Stack operation` in CloudFormation.

Explanation: If your MySQL DB instance is currently in use with a production application, you can follow a procedure to upgrade the database version for your DB instance that can reduce the amount of downtime for your application.

- Periodically, Amazon RDS performs maintenance on Amazon RDS resources. Maintenance most often involves updates to the DB instance’s underlying hardware, underlying operating system (OS), or database engine version. Updates to the operating system most often occur for security issues and should be done as soon as possible
  - Some maintenance items require that Amazon RDS take your DB instance offline for a short time. Maintenance items that require a resource to be offline include the required operating system or database patching. Required patching is automatically scheduled only for patches that are related to security and instance reliability. Such patching occurs infrequently (typically once every few months) and seldom requires more than a fraction of your maintenance window.

- ** When you modify the database engine for your DB instance in `a Multi-AZ deployment`, Amazon RDS `upgrades both the primary and secondary DB instances at the same time`. 
  - In this case, the database engine for `the entire Multi-AZ deployment` is `shut down during the upgrade`.

- ** You need to create `a Read Replica` before updating the stack, which you could have used as `a backup instance` in the event of update failures.



Properties in `AWS::RDS::DBInstance` resource type in the `CloudFormation` template
- `EngineVersion` property = The database version (`Major` + `Minor` version)
  - (x) IMPORTANT: there is NO such `DBEngineVersion` property.
  - This property can be updated.

- `AutoMinorVersionUpgrade` = a value that indicates whether `minor` engine upgrades are applied automatically to the DB instance during the maintenance window.
  - By default, minor engine upgrades are applied automatically. 

- `AllowMajorVersionUpgrade` = a value that indicates whether `major` version upgrades are allowed.



Problem: All data in the storage solution must be encrypted both at rest and in transit. In addition, all of its data must also be replicated in two locations that are at least 450 miles apart from each other.

Solution: 
- Set up primary and secondary S3 buckets in two separate `AWS Regions` that are at least 450 miles apart. 
- Create `a bucket policy` to enforce access to the buckets only through HTTPS 
- and enforce `S3-Managed Keys (SSE-S3) encryption` on all objects uploaded to the bucket. 
- Enable `cross-region replication (CRR)` between the two buckets.

Explanation:

NOTE: the `AZs` are physically separated by a meaningful distance, many kilometers, from any other AZ, although all are `within 100 km (60 miles of each other)`.

By default, `Amazon S3` allows both HTTP and HTTPS requests. 
- ** To comply with the `s3-bucket-ssl-requests-only` rule, confirm that your `bucket policies` explicitly deny access to HTTP requests. 
  - `Bucket policies` that allow HTTPS requests without explicitly denying HTTP requests might not comply with the rule.

- To determine HTTP or HTTPS requests in a bucket policy, use a condition that checks for the key `“aws:SecureTransport”`. When this key is `true`, this means that the request is sent through HTTPS.

- To be sure to comply with the `s3-bucket-ssl-requests-only` rule, create `a bucket policy` that explicitly denies access when the request meets the condition `“aws:SecureTransport”: “false”`. This policy explicitly denies access to HTTP requests.

NOTE: `IAM Role` cannot enforce access to the S3 buckets only through HTTPS like S3 `bucket policies`.






Problem: deploy the new web portal for the mobile app without having any impact on the company’s 10 million users

Solution:
- Launch `a brand new OpsWorks stack` that contains a new layer with the latest PCI-DSS compliant web portal version. 
- Use `a Blue/Green deployment strategy` with `Amazon Route 53` that shifts traffic between the existing stack and new stack. 
- Route only a small portion of incoming production traffic to use the new application stack while maintaining the old application stack. 
- Check the features of the new portal and once it's 100% validated, slowly increase incoming production traffic to the new stack. 
- Change the `Route 53` to revert to old stack if there are issues on the new stack.

NOTE: 
- `Blue/green deployments` provide near zero-downtime release and rollback capabilities. 

- The fundamental idea behind blue/green deployment is to shift traffic between two identical environments that are running different versions of your application. 
  - ** The `blue environment` represents `the current application version` serving production traffic. 
  - ** In parallel, the `green environment` is `staged running a different version of your application`. 

- ** After the `green environment` is ready and tested, production traffic is redirected `from blue to green`. 
  - If any problems are identified, you can roll back by reverting traffic back to `the blue environment`.





NOTE: There is no `Canary deployment` configuration in `Elastic Beanstalk`. This type of deployment strategy is usually used in `Lambda`.




NOTE: Much like `Amazon RDS Multi-AZ`, the `Aurora multi-master` has its data replicated across `multiple Availability Zones` only 
- ** but NOT to `another AWS Region`. 
- ** You should use `Amazon Aurora Global Database` instead.
- ** `Read replica` can be in different regions.



Problem: Which among the options below should be taken to MINIMIZE downtime of the portal in the event that the `primary database` in RDS fails?

Solution:
- ** Launch `a read replica` of the `primary database` to the `second region`. 
- Set up `Amazon RDS Event Notification` to publish status updates to an SNS topic. 
- Create `a Lambda function` subscribed to the topic to monitor database health. 
- ** Configure `the Lambda function` to promote `the read replica` as the `primary` in `the event of a failure`. 
- Update `the Route 53 record` to redirect traffic from the primary region to the secondary region.






---

## Other notes

`Amazon Macie` is just a security service that uses machine learning to automatically discover, classify, and protect sensitive data in AWS.


`Amazon GuardDuty` is a threat detection service that continuously monitors for malicious activity and unauthorized behavior to protect your AWS accounts and workloads 


`Amazon Inspector` service is primarily used to help you check for unintended network accessibility of your Amazon EC2 instances and for vulnerabilities on those EC2 instances.
- Also it is an automated security assessment service that helps improve the security and compliance of applications deployed on AWS.


`AWS Shield` is a managed `Distributed Denial of Service (DDoS) protection` service


`AWS Elemental MediaStore` is a video origination and `storage service` that offers the performance, consistency, and low latency required to deliver `live video content` combined with the security and durability Amazon offers across its services.


`AWS Elemental MediaPackage` is a video origination and `just-in-time (JIT) packaging service` that allows video providers to securely and reliably deliver `live streaming content` at scale.


`Lambda@Edge` is primarily used to run code closer to users of your application in order to improve application performance and reduce latency. 
- * `Serving static content` using `Lambda@Edge` is NOT a suitable use case.


`AWS CloudShell` provides an easy and secure way of interacting with AWS resources via `browser-based shell`.


`AWS Systems Manager State Manager` is just a secure and scalable configuration management service that automates the process of keeping your Amazon EC2 and hybrid infrastructure in a state that you define.


`AWS Systems Manager Session Manager` is just a fully managed AWS Systems Manager capability that lets you manage your Amazon EC2 instances through an interactive one-click browser-based shell or through the AWS CLI.


`AWS Service Catalog` service allows organizations to create and manage catalogs of IT services that are approved for use on AWS.


`AWS Site-to-Site VPN` enables you to securely connect your on-premises network or branch office site to your Amazon Virtual Private Cloud (Amazon VPC). 
- `A Site-to-Site VPN connection` offers `two VPN tunnels` between `a virtual private gateway or a transit gateway` on the AWS side, and `a customer gateway` (which represents a VPN device) on the remote (on-premises) side.


NOTE: You can create `CNAME records only for subdomains` and NOT for the `zone apex` or `root domain`.

NOTE: You should create `an alias record` with the `root DNS name` and not a non-alias A record.

NOTE: Keep in mind that you have to create `an alias record` in Amazon Route 53 in order to point to `your load balancer`.





`AWS Systems Manager State Manager` is primarily used as a secure and scalable configuration management service that `automates the process of keeping your Amazon EC2 and hybrid infrastructure in a state that you define`. 
- With the State Manager, you can 
  - configure your instances to `boot with a specific software at start-up`; 
  - `download and update agents` on a defined schedule; 
  - configure `network settings` and many others, 
- ** but not the patching of your EC2 instances.
  - This does not handle patch management, unlike `AWS Systems Manager Patch Manager`. 


`Session Manager` is primarily used to comply with corporate policies that require `controlled access to instances`, strict security practices, and fully auditable logs with instance access details.




`Amazon Inspector` is just an automated security assessment service that helps improve the security and compliance of applications deployed in AWS. 
- ** It does NOT have the capability to detect EC2 instances that are using `unapproved AMIs`, unlike `AWS Config`.




`AWS Firewall Manager` is primarily used to `manage your Firewall across multiple AWS accounts` under your AWS Organizations. 




