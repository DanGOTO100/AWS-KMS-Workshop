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

PIC

You should get to a success page. 

PIC


Now, go back  and check that it is showing  and then click on it, to download it and display it.

If you refresh the page in your browser, you will notice the same file appears now as a local file with prefix "localfile". This is to show how a further cache could be created.

