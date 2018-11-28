
# AWS KMS Workshop

AWS Key Management Service (KMS) makes it easy for you to create and manage keys and control the use of encryption across a wide range of AWS services and in your applications. AWS KMS is a secure and resilient service that uses FIPS 140-2 validated hardware security modules to protect your keys.

This workshop pretends to provide a better understanding on AWS Key Management Service (KMS) through a set of practical exercises.
The workshop is aligned with the AWS KMS best practices "must-read" Whitepaper "**[AWS Key Management Service Best Practices](https://d0.awsstatic.com/whitepapers/aws-kms-best-practices.pdf)**" and the practices follows its guidelines.

---

# Workshop content:
The workshop contains four different sections (to be followed in order) covering areas like AWS CMKs operations, Types of encryption in AWS KMS with focus on envelope encryption, key policies and best practices working with a demo Web App and AWS KMS monitoring.

* [Section I - Operating with AWS KMS and the CMKs](https://github.com/DanGOTO100/Draft-AWS-KMS-Workshop/blob/master/Section-1-Operating-with-AWS-KMS.md)
* [Section II - Encryption with AWS KMS](https://github.com/DanGOTO100/Draft-AWS-KMS-Workshop/blob/master/Section-2-Encryption-with-AWS-KMS.md)
* [Section III - Key Policies and best practices - Working with a Web App](https://github.com/DanGOTO100/Draft-AWS-KMS-Workshop/blob/master/Section-2-Encryption-with-AWS-KMS.md)
* [Section IV - Monitoring AWS KMS]()

The workshop is mostly practical and will operate in AWS KMS using the AWS CLI (through an EC2 instance), AWS console and AWS KMS API calls, to get a better understanding of the different options. The workshop is designed to work under **eu-west-region (Paris)**. 

---

# Pre - Requisites:

In order to set up the working environment for the workshop, you need the following:

* An AWS account.
* An user with enough permissions to generate policies and create/modify roles in IAM, launch EC2 instances and run CloudFormation templates.
* A set of key pairs in order to access the instances that will be launched from CloudFormation. If you don´t have it, create them as per the instrucions in the AWS EC2 documentation: [creating a  key pairs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html#having-ec2-create-your-key-pair).
* Capability to connect to the EC2 instances via terminal. For options, check the following [link to the EC2 documentation](https://docs.aws.amazon.com/quickstarts/latest/vmlaunch/step-2-connect-to-instance.html)

**Once you are ready**, click on the following **CloudFormation template** to launch the needed resources and go to the first section of the workshop:[Section I - Operating with AWS KMS and the CMKs](https://github.com/DanGOTO100/Draft-AWS-KMS-Workshop/blob/master/Section-1-Operating-with-AWS-KMS.md).


