# Draft - AWS KMS Workshop

AWS Key Management Service (KMS) makes it easy for you to create and manage keys and control the use of encryption across a wide range of AWS services and in your applications. AWS KMS is a secure and resilient service that uses FIPS 140-2 validated hardware security modules to protect your keys.

This workshop pretends to provide a better understanding on AWS Key Management Service (KMS) through a set of practical exercises.
The workshop is aligned with the AWS KMS best practices "must-read" Whitepaper "**[AWS Key Management Service Best Practices](https://d0.awsstatic.com/whitepapers/aws-kms-best-practices.pdf)**" and the practices follows its guidelines.


The workshop contains four different sections (to be followed in order) covering areas like AWS CMKs operations, Types of encryption in AWS KMS with focus on envelope encryption, key policies and best practices working with a demo Web App and AWS KMS monitoring.

* [Section I - Operating with AWS KMS and the CMKs](https://github.com/DanGOTO100/Draft-AWS-KMS-Workshop/blob/master/Section-1-Operating-with-AWS-KMS.md)
* [Section II - Encryption with AWS KMS](https://github.com/DanGOTO100/Draft-AWS-KMS-Workshop/blob/master/Section-2-Encryption-with-AWS-KMS.md)
* [Section III - Key Policies and best practices - Working with a Web App](https://github.com/DanGOTO100/Draft-AWS-KMS-Workshop/blob/master/Section-2-Encryption-with-AWS-KMS.md)
* [Section IV - Monitoring AWS KMS]()

The workshop is mostly practical and will operate in AWS KMS using the AWS CLI, AWS console and AWS KMS API calls, to get a better understanding of the different options. The Sections

# Pre - Requisites:

In order to set up the working environment for the workshop, you need the following:

	• An AWS account
	• An user with enough permissions to generate policies and create/modify roles in IAM, launch instances and run CloudFormation templates.

Once you are ready, click on the following CloudFormation template to launch the needed resources:


