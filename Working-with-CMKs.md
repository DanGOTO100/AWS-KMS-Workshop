
# Operating with AWS KMS and CMKs


## Creating Customer Master Key (CMK)

Let 's connect to the instance and start working with the CMKs.
CMKs are the primary resources in AWS KMS. You can use a CMK to encrypt and decrypt up to 4 kilobytes (4096 bytes) of data. However, most commonly, you will use CMKs to generate, encrypt, and decrypt the data keys that you use outside of AWS KMS to encrypt your data.

There are different types of CMKs in KMS, see documentation here. Our first task in AWS KMS will be to create CMKs that will help us during the rest of the workshop.
In this section we will create a CMK with key material coming from AWS KMS, and later we will generate a CMK with your own key material. It is important to remember that CMKs never leave AWS KMS unencrypted.

### Step 1 - create CMKs

 In order to do that we issue the AWS CLI command aws kms create-key. You can check the whole command syntax in the API reference documentation, however we will make a simple call this time with no parameters.
Note that you might need to configure your region in AWS CLI. You can do so with command aws configure, leaving all blank except for the default region, choose eu-east-1.

```
$ aws kms create-key
```


The response from above command should be an error message like the one below. 

```
An error occurred (AccessDeniedException) when calling the CreateKey operation: User: arn:aws:sts::account-id:assumed-role/KMSWorkshop-InstaceInitRole/instanceid is not authorized to perform: kms:CreateKey on resource:
```

This is because the initial role we have assigned to the instance does not include the capability to create keys. We need to add a policy to the role in order to enable us to perform certain actions with AWS KMS during the workshop. 

Following **least privilege best practices**, we will be attaching policies with permissions as needed for the different AWS KMS operations we are working with. In this way it is easy track and detach the policies from the role, once they are not needed, and keep the least privilege principles.



### Step 2 - Create Policy and Attach it to the role

We can add the needed permissions via the CLI or the console. We will use the console for this operation.

Navigate again to the IAM service and click on Roles, left area of the screen.
![alt text](/res/S1F1%20IAM.png)
<**Figure-1**>

Search for the role that has been set up and attached to the instance by the CloudFormation template, its name is KMSWorkshop-InstanceInitRole. 
![alt text](/res/S1F2%KMSinitRole.png)
<**Figure-2**>

Click on the role and then on "**Attach Policy**", we are going to provide permissions so the instance can create Keys. 
![alt text](/res/S2F3%20AttachPolicy.png)
<**Figure-3**>

Search "**AWSKeyManagementSystem**", and select the policy "**AWSKeyManagementSystemPowerUser".  That is the policy we are going to use for the instance role. **Please note**, the assigment of KMS Power User permissions is **just** for the initial walk-through in KMS, a typical user might not need the whole set of permissions. Later in the workshop we will work on how to implement more fine grained "Least Privilege" access, according to best practices,  in order to assign appropriate permissions to users and roles into KMS operations.

![alt text](/res/S1F4%KMSPowerUserPolicy..png)
<**Figure-4**>



As this point we can review the policy, just by clicking on it. See the operations it is allowing the Poweruser into KMS.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "kms:CreateAlias",
                "kms:CreateKey",
                "kms:DeleteAlias",
                "kms:Describe*",
                "kms:GenerateRandom",
                "kms:Get*",
                "kms:List*",
                "kms:TagResource",
                "kms:UntagResource",
                "iam:ListGroups",
                "iam:ListRoles",
                "iam:ListUsers"
            ],
            "Resource": "*"
        }
    ]
}
```

Select the policy and click the "Attach policy"  button. The role attached to the instance is modified with his new policy and we should be able to create a Customer Master Key (CMK) now from the instance.



### Step 3 - Create the key  - again - and set an alias
Run the aws kms create-key command again and this time you will get the result from the creation on a JSON block, like the one below:
```
$ aws kms create-key

{
    "KeyMetadata": {
        "Origin": "AWS_KMS", 
        "KeyId": "your-key-id", 
        "Description": "", 
        "KeyManager": "CUSTOMER", 
        "Enabled": true, 
        "KeyUsage": "ENCRYPT_DECRYPT", 
        "KeyState": "Enabled", 
        "CreationDate": 1538497627.686, 
        "Arn": "arn:aws:kms:eu-west-1::key/", 
        "AWSAccountId": "your-account-id"
    }
}

```

If you go to the console and navigate to the IAM service, in the left area to the bottom, "Encryption Keys", the key you have just created is already listed there. Remember to select the right Region. However, as we used the create-key command without parameters, it does not contain any alias to display and looks like its alias is empty. 

Key alias are very useful. They are easier to remenber when operating keys. Most importantly, when rotation keys, as we will see later in this section, we will not have to update our code to update it with the new KeyIDs or ARN. By using alias in our code to call the CMKs by its alias, and updating the alias CMKs to point to the newly generated key, the amount of change in our code gets minimized.

Let's create it an alias, "FirstCMK",  with the command aws kms create-alias. 
Remember to replace 'your-key-id' with the value obtained from previous command (aws kms create-key).

$ aws kms create-alias --alias-name alias/FirstCMK --target-key-id 'your-key-id'

If you look now in the console, the CMK you just created displays now the right alias. 

When you create the CMK from the console, just by clicking the button "create key" there are other parameters you need to set like tags, key administrators and usage permissions. This steps will basically create a Key Policy and attach it to the key together with the tags you have set. 
For the workshop, we will see how creating CMKs, policies and tags can be done from the CLI to have greater insights on their scope and implications. 





