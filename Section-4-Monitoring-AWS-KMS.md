# Monitoring and Logging in AWS KMS

Monitoring is an important part of understanding the availability, state, and usage of your customer master keys (CMKs) in AWS KMS and maintaining the reliability, availability, and performance of your AWS solutions. 
As as baseline, in AWS KMS you may want to monitor:

* Activity related to cryptographic operations, such as Encrypt or Decrypt.
* Activity related to management operations on the CMKs: EnableKey, ImportKeyMarterial,etc…
* Activity on other events and metrics, such as key expiration, key rotation or time remaining until imported key material expiration.

To monitor that activity we can use [AWS CloudTrail](https://aws.amazon.com/cloudtrail/) and [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/) logs, events and alarms.

--
### AWS KMS and AWS CloudTrail

AWS KMS is integrated with [AWS CloudTrail](https://aws.amazon.com/cloudtrail/), a service that provides a record of actions performed by a user, role, or an AWS service in AWS KMS.
CloudTrail captures all API calls for AWS KMS as events, including calls from the AWS KMS console and from code calls to the AWS KMS APIs.

Let's first create a data key with the corresponding AWS KMS command we have used before a few times:

```
$ aws kms generate-data-key --key-id alias/ImportedCMK --key-spec AES_256 --encryption-context project=workshop
```


This action has been logged in AWS CloudTrail, and we can obtain its details. Let's use the console.
Go back to the AWS console in your browser, navigate to CloudTrail service, select "**Event history**" on the right panel. 
You have the full history of events that can be filtered.

In the Filter area, select "**Event name**", the name is "**GenerateDataKey**" and leave "**Select time range**" as it is. 
You will have a display of the GenerateDataKey operations that you have performed during the workshop.

![alt text](/res/S4F1.png)
<**Figure-1**>

If you open any of those you will have further details of the operation.
You can filter by many other parameters to collect all needed information to audit AWS KMS usage.
It is a powerful service by itself, and can become even more powerful if we combine it with other services.
Let's create, for example, custom notifications based on AWS Cloudtrail events coming from AWS KMS.

---

### AWS KMS. Real time notifications with AWS CloudTrail, AWS CLoudWatch and Amazon SNS.

What we are going to do is to create a notification system with [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/), [AWS CloudTrail](https://aws.amazon.com/cloudtrail/) and [Amazon SNS](https://aws.amazon.com/sns/).
We will do through the AWS console. 
First let's create a notification topic and subscribe to it, to receive notifications when an event we defined is triggered.

In the console, navigate to Simple Notification Service (SNS) service. Click on "Create Topic" and provide a topic name and a display name. Any name may work, try with "**snsworkshop**" for the name and choose any display name must not exceed 10 characters. 
Take note of the "**Topic ARN*" listed there. Something like: arn:aws:sns:eu-west-1:yout-account-id:snsworkshop

Now on the right pane, select "**Subscriptions**" and click on "**Create subscriptions**".
Provide the topic ARN that you took note in the previous step and in "Protocol" select "Email" and provide your mail.
You will receive an email to confirm your subscription.  

With this, navigate to AWS CloudWatch service in the console, and in the right pane, select "**Events**".
Leave "**Event Pattern**" Clicked and select "**Events by Service**" in "**Build event pattern to match...*".

![alt text](/res/S4F2.png)
<**Figure-2**>


Select "Service Name" as "CloudTrail" and "Event Type"  as "AWS API Call via CloudTrail".
Then select "Specific operation(s) and in the blank space, type the event name: "GenerateDataKey".

![alt text](/res/S4F3.png)
<**Figure-3**>


Now press the "**+**" symbol to aggregate it to then event source filter. We have the input part ready, now let's do the target part. 
On the right side of the screen, find "**+ Add Target^^" and click it.
On the first row, it would say "**Lambda Function**", change it to "**SNS Topic**", Then select the topic created before "**snsworkshop**". 
You are ready to hit the "**Configure details**" button on the botton of the page.
Now just provide a name to the rule and hit "**Create Rule**".

You have just created a rule that will help you audit AWS KMS usage. Everytime a Data Key is generated, you wil be notified in the email address you provided. 
Let's test it by calling the GenerateDataKey operation again.

```
$ aws kms generate-data-key --key-id alias/ImportedCMK --key-spec AES_256 --encryption-context project=workshop
```
If everything went well you should now receive an email notifying you of the operation that took place. 
**Note: ** Don´t forget to hae confirmed your subcripution to SNS topic (you should have recevied an email).
We have established a notification for a specific operation. F
or a list of the log entries that AWS KMS generates in AWS CloudTrail, please check the following [section of the AWS KMS documentation](https://docs.aws.amazon.com/kms/latest/developerguide/logging-using-cloudtrail.html).


### AWS KMS and CloudWatch Metrics

In AWS Cloudwatch you also have **metrics** about AWS KMS. For example,  when you import key material into a CMK and set it to expire, AWS KMS sends metrics and dimensions to CloudWatch.
For details on the dimensions of these metrics, you can check the [following section](https://docs.aws.amazon.com/kms/latest/developerguide/monitoring-cloudwatch.html) of the AWS KMS documentation.
With Amazon Cloudwatch you can create alarms and dashboards that will give you certain insights on AWS KMS. 

To check these metrics into Amazon CloudWatch navigate again into Amazon CloudWatch and select "**Metrics**" on the right pane.
Inside metrics, find the ones that belong to AWS KMS:

![alt text](/res/S4F4.png)
<**Figure-4**>

If you click through it you will find the metric "SecondsUntilKeyMaterialExpiration" for your CMK built with imported  key material. 
With this metric you can now build an alarm into CloudWatch to warn you about the expiration of the key material for example. 

Now, as a final assignment, let's create an Amazon Cloudwatch alarm over an AWS KMS metric. follow the Amazon Cloudwatch instructions in [this Amazon CloudWatch section](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ConsoleAlarms.html) to c
In this case we only have a metric coming from AWS KMS so it should be easy for you to identify it and creating the alarm for it. 
You can use the SNS topic that we create before: "**snsworkshop**". 

The process is well described in prevous link, so it is not reproduced here in the workshop's instructions. 
In case you need more details about building the alarm, please look into how to build an alarm from AWS KMS metrics in the followin [section of the AWS KMS documentation](https://docs.aws.amazon.com/kms/latest/developerguide/monitoring-cloudwatch.html#key-material-expiration-alarm). 

---

Workshop has finished - Thank you for completing this Workshop.

 

