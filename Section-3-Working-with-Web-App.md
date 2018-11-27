# Working with a WebApp - AWS KMS best practices

In this section, using a Web App, we are going to implement best practices for AWS KMS.
These best practices are based on the Whitepaper "**[AWS Key Management Service Best Practices](https://d0.awsstatic.com/whitepapers/aws-kms-best-practices.pdf)**"

this section has the following parts:

---

### Part 1 - Installing the WebApp

The WebApp is very simple python web server that works as a shared file server, for internal employees for example. It allows to upload and download files to/from  S3. For downloads the WebApp keeps a local file in the instance where WebApp is running, prefixing the file with "localfile-". Remenber, our instance has a role with a policy attached to it that allow to read/write from S3.

Let's make a working directory for a our sample WebApp and install the boto3 AWS Pyhon library library: Check we are in our home directory first.

```
$ pwd
/home/ec2-user
```

Make the new directory and install the needed boto3 python library (if not present already).

```
$ sudo mkdir SampleWebApp
$ sudo pip install boto3
```

Now, get into the directory and download the sample WebApp from the following link:

```
$ sudo cd /SampleWebApp
$ sudo wget  xxxxxxxxxxx
```

You have downloaded a python application, named "**SampleWebApp.py**", that will be our test Web App.

We can run the server now with the following command:

```
sudo python WebApp.py 80
```

We will need to obtain the instance IP to connect to it from the Internet. We will get it from the metadata of the instance. If you need more information about instance metadata, please look into this section of the documentation.

```
sudo curl http://169.254.169.254/latest/meta-data/public-ipv4/
 
54.X.X.44
```

You can go to a browser now and navigate to the WebApp in the URL you obtained in the previous step:  http://54.X.X.44
**Note:** if you run into issues with reaching the server, it may be worthy to recheck the security group associated with the server, use [this link to the Security Groups documentation](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-network-security.html). 

The WebApp has the file uploader, a browser of present objects in our S3 bucket and the local files.
In your laptop create a text file with a sample text, and provide it a name like "**SampleFile-KMS.txt**"
Upload it through the WebApp.

![alt text](/res/S3F1.png)
<**Figure-1**>


You should get to a success page. 

![alt text](/res/S3F2.png)
<**Figure-2**>


Now, go back  and check that it is showing  and then click on it, to download it and display it.

If you refresh the page in your browser, you will notice the same file appears now as a local file with prefix "localfile". The Web App is designed to create also a further local cache.


---

### Part 2 - Adding Encryption to the Web App


The S3 bucket with its corresponding files is well protected under Bucket Policies and IAM policies. Currently, the role we have set in the working instance, has read and write access to the S3 bucket. 
However, for some reason, other instances or users may need read access to the bucket.
It might be desirable that we encrypt the files with the CMK we have create importing our key material, to add  protection for the files in bucket. 

In order to do that, the Web App is using boto3 python S3 APIs to upload the files. We need to use the appropriate API to upload the files using Server Side Encryption with AWS KMS and the CMK we created. 

The API as stated in the [Amazon S3 documentation](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/s3.html#id217), has this structure:

```
s3.put_object(Bucket=BUCKET,
              Key='encrypt-key',
              Body=b'foobar',
              ServerSideEncryption='aws:kms',
              # Optional: SSEKMSKeyId
              SSEKMSKeyId=keyid)
```

We could very easily modify the code in our App to include the Server Side Encryption. 
Howeverit is more clear if we download a version of the WebApp with the changes already implemented, and hence that provides Server Side Encryption using one of our CMKs.

Stop the server from running with CTRL+C (maybe twice)
Download the version of the Web App that **adds encryption** and run the server again:

$  sudo wget XXXXXXXXXXXXXXXXXX

We are going to need the KeyId of the CMK we pretend to use for the encryption of the files. The CMK we pretend to use is the one generated with our import material and which alias was "**ImportedCMK**".

Issue the following command to display your working keys and identify the KeyId of "ImportedCMK". 

```
$ aws kms list-aliases

{
    "Aliases": [
        {
            "AliasArn": "arn:aws:kms:eu-west-1:your-account-id:alias/ImportedCMK", 
            "AliasName": "alias/ImportedCMK", 
            "TargetKeyId": "your-key-id"
        }, 

```


Copy the value under **TargerKeyID**, it is the KeyID of our CMK and we are going to need when we start our file upload server. Keep it handy, we will use several times in this section of the workshop.
Now it is time to start the server with encryption:

```
$ sudo python WebAppEncSSE.py 80
```

Enter the KeyID of the CMK we have obtained in the previous step when asked. Make sure it is the right one, otherwise the upload will fail. 

Once the server is running on port 80, go back again into your laptop's browser and refresh the file uploader server page. Try to upload again the text file you created before, "SampleFile-KMS.txt".

If the upload is successful, let's check that the file was in fact encrypted. We will use the AWS console.
Open the AWS console. Navigate to the Amazon S3 service and locate our working bucket "kms-workshop".  The file you have just uploaded will be there.  Click on its name to open its properties.

![alt text](/res/S3F3.png)
<**Figure-3**>


As you can see, the files metadata specifies that is under Server Side Encryption and displays the ARN of the KMS key used to encrypt. Any accidental access to the bucket now will not be able to display the contents of the file, as they are encrypted.

From the file browser Web App, you can download and display the file we have upload and encrypted. Remember that when using Server Side Encryption with KMS,  you don´t need to provide any additional information for getting the object; S3 is able to know how to decrypt the object from the metadata. 

Amazon S3 Server Side Encryption works with envelope encryption in a similar way as what we described in the previous section for Amazon EBS. For more details check [this S3 documentation link](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingKMSEncryption.html).

---

### Part 3 - Working with Key Policies

Key policies are the primary resource for controlling "who" has access to do "what" with your CMKs.
You have a full description about them in the following [AWS KMS link](https://docs.aws.amazon.com/kms/latest/developerguide/key-policies.html), in case you want to go deeper - and for the importance of the topic,  you should. 
We are going to work with some practical examples. 

Up to now, the assigned IAM  role ("**KMSWorkshop-InstaceInitRole**") of our working instance allows us to perform many things in AWS KMS.  Following best practices like "**Least Privilege**" and "**Separation of Duties**", it can be, for example, that our instance is meant to be used only for uploading data with server side encryption, but not decrypt it and download it.
Maybe the download and decrypt operation needs to be done from another instance with more specific security constraints. 

How can we comply with these requirements? We will use two main resources:
* IAM roles and their policies are helpful to control access to CMKs. 
* **Key policies** are resource based policies that we can use to fine grain the access to the CMKs and tweak it to our needs.

Each key that is generated in AWS KMS, has an initial policy attached. Let's looks at the initial policy set for the key we created with our import material, alias "**ImportedCMK**". Type the following command in your instance terminal (replace "your-key-id" by the "ImportedCMK" KeyId or ARN, **Note:** the Alias will not work for this operation):

```
$ aws kms list-key-policies --key-id your-key-id

{
    "PolicyNames": [
        "default"
    ]
}
```

If you don´t have a copy of the KeyId, just issue again the list-aliases command  identify the KeyId of "ImportedCMK". 

```

$ aws kms list-aliases

{
    "Aliases": [
        {
            "AliasArn": "arn:aws:kms:eu-west-1:your-account-id:alias/ImportedCMK", 
            "AliasName": "alias/ImportedCMK", 
            "TargetKeyId": "your-key-id"
        }, 
```

Note that we are able to list key policies because the IAM role assigned of our instance allows this operation.  As you can see, we have a key policy attached to the key, its name is "Default". Let's see what it contains:

```
$ aws kms get-key-policy --key-id your-key-id --policy-name default
{
    "Policy": "{\n  \"Version\" : \"2012-10-17\",\n  \"Id\" : \"key-default-1\",\n  \"Statement\" : [ {\n    \"Sid\" : \"Enable IAM User Permissions\",\n    \"Effect\" : \"Allow\",\n    \"Principal\" : {\n      \"AWS\" : \"arn:aws:iam::your-account-id:root\"\n    },\n    \"Action\" : \"kms:*\",\n    \"Resource\" : \"*\"\n  } ]\n}"
}

```

The default key policy gives the AWS account (root user) that owns the CMK full access to the CMK.
This has policy has two important effects (more information in [this link](https://docs.aws.amazon.com/kms/latest/developerguide/key-policies.html#key-policy-default):

	* Reduces the risk of the CMK becoming unmanageable.
		You cannot delete your AWS account's root user, so allowing access to this user reduces the risk of the CMK becoming unmanageable.
  
	* Enables IAM policies to allow access to the CMK.
		Giving the AWS account full access to the CMK does this; it enables you to use IAM policies to give IAM users and roles in the account access to the CMK. It does not by itself give any IAM users or roles access to the CMK, but it enables you to use IAM policies to do so.

Let's modify the permission of the role assigned to the instance, **allowing it only to encrypt, but not decrypt**.
Let's use the console. Open the AWS Console. Navigate to IAM service, left column "Roles" and search for the role currently assigned to the instance: **KMSWorkshop-InstaceInitRole**. 

![alt text](/res/S3F4.png)
<**Figure-4**>

As per image above, locate the policy we attached when working in the second section of the workshop named "**KMS-Workshop-EncDecPermissions**". Click the button "**Edit Policy**". Expand the Actions-Access Level-Write section and remove the check box on "**Decrypt**". Review policy and save it.

Let's run the server once more- remember you need the KeyID again, and try to get the file:

```
$ sudo python WebAppEncSSE.py 80
```

Now try to upload a new file to S3. It will succeed it. Now try and download it. You can also try to download the previous text file we created "SampleFile-KMS.txt". Both operations will fail, and the logs on your instance display show have something like:

```
"ClientError: An error occurred (AccessDenied) when calling the GetObject operation: Access Denied"
```
**NOTE: **  "kms:Encrypt" needs also the capability to generate data keys if it is meant to be used though AWS services (envelope encryption!) This is important when fine-graining key policies.

#### Least Privilege - Access only from the account

Now this role is able to encrypt but not to decrypt. Furthermore, we want to enforce "**Least Privilege**" access and ensure that the "encrypt" role, able to encrypt is only used from our account, and not subject to Cross-Account role access policies that could grant access to the CMK. For that we will use Key policy.

We need to identify our current role "**KMSWorkshop-InstaceInitRole**" ARN in order to link it to the key policy. Go back to the console, IAM service, click Roles. Search for the role currently assigned to the instance: **KMSWorkshop-InstaceInitRole** and click on it.  In the upper part of the screen you have the associated ARN. As part of the Role ARN you have your account Id, just after the region. This is the structure: 

```
arn:aws:iam::ACCOUNT-ID-WITHOUT-HYPHENS:role/ROLE-NAME
```
Copy the account Id and keep it close. 


![alt text](/res/S3F5.png)
<**Figure-5**>


Go back to the left column click "**Encryption Keys**". If you need to press the button "**Get Started Now**". 
Identify our CMK "**Imported CMK**" and click on it. You will land on a page with more details about the key itself. Scroll down until you see "**Key Policy**".

Modify the current policy with this one:

```
{
  "Version": "2012-10-17",
  "Id": "key-default-1",
  "Statement": [
    {
      "Sid": "Enable IAM User Permissions",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::your-acount-id:root"
      },
      "Action": "kms:*",
      "Resource": "*"
    },
    {
      "Sid": "Allow for Use only within our Account",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::your-acount-id:role/KMSWorkshop-InstaceInitRole"
      },
      "Action": "kms:*",
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "kms:CallerAccount": "your-account-id"
        }
      }
    }
  ]
}
```

Make sure you replace "**your-account-id**" for the real values of your account.  With this policy you have restricted the key to be used only within your account.

#### Least Privilege - Access only from a Role

Another type of key policy that can become very useful when enforcing "**Least Privilege**" is the capability to also ensure that this CMK is only called by our role and other roles or user can´t use the key for encryption or decryption. For that, you can use a policy like the one below:

```

{
  "Version": "2012-10-17",
  "Id": "key-default-1",
  "Statement": [
    {
      "Sid": "Enable IAM User Permissions",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::your-acount-id:root"
      },
      "Action": "kms:*",
      "Resource": "*"
    },
    {
      "Sid": "Allow for Use only within our Account",
      "Effect": "Deny",
      "NotPrincipal": { 
        "AWS": [ "arn:aws:iam::your-acount-id:role/KMSWorkshop-InstaceInitRole", "arn:aws:iam::your-acount-id:root"]
      },
      "Action": "kms:*",
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "kms:CallerAccount": "your-account-id"
        }
      }
    }
  ]
}

```
With this policy we will ensure that only instances that have the appropriate role attached are able to use the key. It can become handy to control the instances/services tha can use the CMKs, only allowing it through a specific role. This policy is set as an example of what can be done. Optionally you can enforce it, but it is not required in the workshop.


#### Key Policy - Including conditions

Furthermore,  you can also include conditions over key policies to help you fine tune access and link to several other parameters. 
As an example, conditions can work with **encryption context** to be able to restrict the operations for this KMS. Amazon S3 when calling AWS KMS to generate a Data Key and perform the envelope encryption process, it passes and encryption context to AWS KMS, see below a log from a "**GenerateDataKey**" event:
..
"eventName": "GenerateDataKey",
…
"requestParameters": {
        "keySpec": "AES_256",
        "encryptionContext": {
            "aws:s3:arn": "arn:aws:s3:::kms-workshop/SampleFile-KMS.txt"
        },
…

We could use that to add a condition in KMS with the condition kms:EncryptionContextKey. 
There is a full example in the [documentation here](https://docs.aws.amazon.com/kms/latest/developerguide/policy-conditions.html#conditions-kms-encryption-context-keys). Make sure you click the link and check that you undertand how the key policy is enforced using encryption context as condition.

Finally, let's try to add another layer of security via **MFA**. In the key policy we might request that the users that are going to use the CMK have passed through a MFA process. 
MFA is enforced through a condition as seen below:

```
{
  "Version": "2012-10-17",
  "Id": "key-default-1",
  "Statement": [
    {
      "Sid": "Enable IAM User Permissions",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::your-acount-id:root"
      },
      "Action": "kms:*",
      "Resource": "*"
    },
    {
      "Sid": "Allow for Use only within our Account",
      "Effect": "Allow",
      "Principal": { 
        "AWS": [ "arn:aws:iam::your-acount-id:user/userA",]
      },
      "Action":  [
    "kms:DeleteAlias",
    "kms:DeleteImportedKeyMaterial",
    "kms:PutKeyPolicy",
    "kms:ScheduleKeyDeletion"
     ], 
      "Resource": "*",
      "Condition": {
        "NumericLessThan":{"aws: MultiFactorAuthAge":"300"}       }
    }
  ]
}
```

In this key policy, we don´t allow certain very sensitive operations to take place, unless the user or the role has gone through a MFA authentication process in the last 5 minutes (300 seconds). Make sure you understand the policy. After the examples seen by now, you should be able to understsand how it works. We will not enforce as in the workshop we are working with roles and not with users. However the overall concept is the same.

---


### Part 4 - Creating an AWS KMS Private Endpoint

Resources can communicate with AWS KMS through a VPC private endpoint.
A VPC endpoint enables you to privately connect your VPC to supported AWS services and VPC endpoint services powered by PrivateLink without requiring an internet gateway, NAT device, VPN connection, or AWS Direct Connect connection. 

When you use a VPC endpoint, **communication between your VPC and AWS KMS is conducted entirely within the AWS network**.
You can specify the VPC endpoint in [AWS KMS API operations](https://docs.aws.amazon.com/kms/latest/APIReference/) and [AWS CLI commands](https://docs.aws.amazon.com/kms/latest/APIReference/).

We will create a VPC endpoint and use it to interact with AWS KMS.
We are going to need a few data about the network our instance is running on. We could use the console to get that data or we could use the AWS CLI to obtain metada from the instance. Let's use the latter and collect the data we need

First let's refresh the region we are running on:

```
 $ curl http://169.254.169.254/latest/meta-data/placement/availability-zone
xx-xxxx-1a
```

Now the rest of the data:

```
 $ curl http://169.254.169.254/latest/meta-data/network/interfaces/macs/

0a:xx:xx:xx:xx:7e
```

Take note of the Mac Address we have just obtained and perform a second curl operation:

```
$ curl http://169.254.169.254/latest/meta-data/network/interfaces/macs/0a:xx:xx:xx:dd:7e/vpc-id

vpc-1xxxxxx1
```

You will obtain the id of the VPC your instance is currently running on. Take good note of this value too.

Now run these two commands,using the appropriate Mac addresses obtained in the previous step and keep the output values:

```
$ curl http://169.254.169.254/latest/meta-data/network/interface:0a:xx:xx:7e/subnet-id
subnet-0axxxx50

$ curl http://169.254.169.254/latest/meta-data/network/interfaces/mac:0a:xx:xx:7e/security-group-ids 
sg-05xxxxxxxxe97
```

We have just obtained the subnet id where the instance is running on and the security group associated to the instance.
When creating the VPC endpoint, we associate it with one or several subnets different AZs. For the workshop, we will associate it with the subnet where the instance is running on.
In the same way, the VPC endpoint must have a security group. Let's associate the same as the instance has right now.

Let's create the endpoint with the follwing command. PLease, use the appropriate values we have obtained for the VPC, Security Group and Subnets for the parameters.:

```
$aws ec2 create-vpc-endpoint  --vpc-id vpc-1xxxx1  --vpc-endpoint-type Interface  --service-name com.amazonaws.eu-west-1.kms  --subnet-ids subnet-0xxx50 --security-group-id sg-05xxxxxxxxe97
```

If the command executed correctly, you will have a JSON response like this:

{
    "VpcEndpoint": {
        "VpcId": "vpc-1xxxxxx1", 
        "NetworkInterfaceIds": [
            "eni-0xxxxxxxxx3"
        ], 
        "SubnetIds": [
            "subnet-0xxxx0"
        ], 
        "PrivateDnsEnabled": true, 
        "State": "pending", 
        "ServiceName": "com.amazonaws.eu-west-1.kms", 
        "RouteTableIds": [], 
        "Groups": [
            {
                "GroupName": "sxxxxg", 
                "GroupId": "sg-05exxxxxxxxf"
            }
        ], 
        "VpcEndpointId": "vpce-0xxxxxxxxxxa", 
        "VpcEndpointType": "Interface", 
        "CreationTimestamp": "2018-xxxxxx-Z", 
        
 "DnsEntries": [
            {
                "HostedZoneId": "xxxxxxx", 
                "DnsName": "kms.eu-west-1.amazonaws.com"
            }, 
            {
                "HostedZoneId": "xxxxxxx", 
                "DnsName": "vpce-xxxxxxxxa-xxxxxx.kms.eu-west-1.vpce.amazonaws.com"
            }, 
            {
                "HostedZoneId": "ZxxxxxxxxxT", 
                "DnsName": "vpce-xxxxxxxxa-xxxxxx.kms.eu-west-1.vpce.amazonaws.com"
            }
        ]
    }
}




Congratualtions, as you can see, the "**VpcEndpointId**" is ready to be used. We are going to need the VPC endpoint Id and the DnsName of the endpoint to make the API calls within it. Take note of "**VpcEndpointId**" and the "**DnsName**" from previous JSON response.
Please note that the VPC endpoint can also easily be created from the console, you can follow the steps in this [link to the documentation](https://docs.aws.amazon.com/kms/latest/developerguide/kms-vpc-endpoint.html).

Now, let's perform an operation in AWS KMS using the endpoint. As an example, we will list the keys we currently have:

```
$ aws kms list-keys --endpoint-url https://vpce-xxxxxxxxa-xxxxxx.kms.eu-west-1.vpce.amazonaws.com
```

Congrats again, the commmand will succeed and our request and response has not gone outside the AWS network. There are many other operations you can use with the endpoint.

---

### Part 5 - VPC Endpoints and Key Policies


VPC endpoints allows us to establish more restrictive conditions to the operations we allow on AWS KMS and its CMKs.

For example, let's change the key policy of our CMK with imported key material, alias "**ImportedCMK**", to allow only certain operations from the VPC endpoint.

As explained in the section "**PWorking with Key Policies**", navigate to the console and change the key policy of the "**ImportedCMK**" CMK key with the one below. Replace the account id and the VPC Endpoint Id with the ones you have obtained.

```
{
  "Version": "2012-10-17",
  "Id": "key-default-1",
  "Statement": [
    {
      "Sid": "Enable IAM User Permissions",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::your-account-id:root"
      },
      "Action": "kms:*",
      "Resource": "*"
    },
    {
      "Sid": "Allow for Use only within our VPC",
      "Effect": "Deny",
      "Principal": {
        "AWS": "arn:aws:iam::your-account-id:role/KMSWorkshop-InstaceInitRole"
      },
      "Action": [
        "kms:Encrypt",
        "kms:Decrypt",
        "kms:ReEncrypt*",
        "kms:GenerateDataKey*"
      ],
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "aws:sourceVpce": "vpce-1xxxxxxxx1"
        }
      }
    }
  ]
}
```

When a VPC Endpoint is setup in the VPC,  The standard AWS KMS DNS hostname (https://kms.<region>.amazonaws.com) resolves to your VPC endpoint. This is only If you enabled private hostnames when you created your VPC endpoint. If so, you do not need to specify the VPC endpoint URL in your CLI commands or application configuration

However, If you disable DNS Hostnames for you VPC with the simple procedure outlined in this [VPC documentation link](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-dns.html#vpc-dns-hostnames), section "**Updating DNS Support for Your VPC**" and try to operate with AWS KMS **outside** the VPC Endpoint, it will fail to execute **due to the policy we have just enforced** that requires certain operations with AWS KMS to be performed through a VPC Endpoint.

Disable DNS Hostnames options and try to execute one of the commands in the policy (use the id of CMK alias "ImportedCMK"):

```
$ aws kms generate-data-key --key-id xxxxxxxxxxxxx

An error occurred (AccessDeniedException) when calling the GenerateDataKey operation: User: arn:aws:sts::48xxxxx5:assumed-role/KMSWorkshop-InstaceInitRole/i-081xxxxxx30 is not authorized to perform: kms:GenerateDataKey on resource: arn:aws:kms:eu-west-1:your-account-id:key/xxxxxxxxxxxxxxxx with an explicit deny
```

Let's use the try the same operation but forcing the request to go through the VPC Endpoint (replace the vpce endpoint with the one you created):

```


$ aws kms generate-data-key --key-id xxxxxxxxxxxxx --endpoint-url https://vpce-xxxxxxxxxxxxxx-xxxxxxxxx.kms.eu-west-1.vpce.amazonaws.com
```

In this case the operation will succeed. We have added and extra layer of protection to our key, making AWS KMS only available internally within AWS for certain sensitive operations.


---



### Part 6 - Key Tagging

Tagging is an important strategy for managning CMKs in AWS KMS.
You can add, change, and delete tags for customer managed CMKs. Each tag consists of a tag key and a tag valuethat you define.
You can add tags to a CMK when you first create them. Then, add, edit, and delete tags at any time. 

To add a tag to the CMK we have been working with, you can use the console or the CLI. Let's tag our CMK "**ImportedCMK**", with a project it may belong to, just an a example.

```
$ aws kms tag-resource --key-id your-key-id --tags TagKey=project,TagValue=kmsworkshop
```

The other usual operations with tags are also available: list tags for resource and untag the resource.
Let's list the resource tags:

```
$ aws kms list-resource-tags --key-id your-key-id
```

Add a few more tags to the CMK and try to remove the tags (untag) with commad "untag-resource".
For information on the command, please use [this section of the AWS KMS documentation](https://docs.aws.amazon.com/kms/latest/developerguide/tagging-keys.html).







