
#PART 1: Operating with AWS KMS and CMKs


##Creating Customer Master Key (CMK)

Let's connect to the instance and start working with the CMKs.
CMKs are the primary resources in AWS KMS. You can use a CMK to encrypt and decrypt up to 4 kilobytes (4096 bytes) of data. However, most commonly, you will use CMKs to generate, encrypt, and decrypt the data keys that you use outside of AWS KMS to encrypt your data.

There are different types of CMKs in KMS, see documentation here. Our first task in AWS KMS will be to create CMKs that will help us during the rest of the workshop.
In this section we will create a CMK with key material coming from AWS KMS, and later we will generate a CMK with your own key material. It is important to remember that CMKs never leave AWS KMS unencrypted.

###Step 1 - create CMKs

 In order to do that we issue the AWS CLI command aws kms create-key. You can check the whole command syntax in the API reference documentation, however we will make a simple call this time with no parameters.
Note that you might need to configure your region in AWS CLI. You can do so with command aws configure, leaving all blank except for the default region, choose eu-east-1.

'''
$ aws kms create-key
'''

The response from above command should be an error message like the one below. 

An error occurred (AccessDeniedException) when calling the CreateKey operation: User: arn:aws:sts::account-id:assumed-role/KMSWorkshop-InstaceInitRole/instanceid is not authorized to perform: kms:CreateKey on resource: *

This is because the initial role we have assigned to the instance does not include the capability to create keys. We need to add a policy to the role in order to enable us to perform certain actions with AWS KMS during the workshop. 
Following least privilege best practices, we will be attaching policies with permissions as needed for the different AWS KMS operations we are working with. In this way it is easy track and detach the policies from the role, once they are not needed, and keep the least privilege principles.

