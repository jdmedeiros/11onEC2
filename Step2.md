Create a Linux instance in AWS and make sure the IAM role associated with the instance has the following inline policy:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": "iam:CreateRole",
            "Resource": "*"
        }
    ]
}
```

Get the other relevants parts [from here](http://peterforgacs.github.io/2017/04/14/How-to-create-and-run-a-custom-Windows-10-virtual-machine-on-Amazon-AWS-EC2/), but in summary they are:


Create an S3 bucket
The bucketname must be unique.
```
aws s3 mb s3://my-unique-bucket --region us-east-1
```
Upload the **.ova** or **.vhd** image to the s3 bucket.
```
aws s3 cp myimage.ova s3://my-unique-bucket --region us-east-1
```
Create a trust policy in the file **trust-policy.json**.
```
{
   "Version": "2012-10-17",
   "Statement": [
      {
         "Effect": "Allow",
         "Principal": { "Service": "vmie.amazonaws.com" },
         "Action": "sts:AssumeRole",
         "Condition": {
            "StringEquals":{
               "sts:Externalid": "vmimport"
            }
         }
      }
   ]
}
```
Create a vmimport role and add vimimport/export access to it.
```
aws iam create-role --role-name vmimport --assume-role-policy-document file://trust-policy.json
```
Create a file named **role-policy.json** replace the !!REPLACEME!! to the bucketname you are using.
```
{
   "Version": "2012-10-17",
   "Statement": [
      {
         "Effect": "Allow",
         "Action": [
            "s3:ListBucket",
            "s3:GetBucketLocation"
         ],
         "Resource": [
            "arn:aws:s3:::!!REPLACEME!!"
         ]
      },
      {
         "Effect": "Allow",
         "Action": [
            "s3:GetObject"
         ],
         "Resource": [
            "arn:aws:s3:::!!REPLACEME!!/*"
         ]
      },
      {
         "Effect": "Allow",
         "Action":[
            "ec2:ModifySnapshotAttribute",
            "ec2:CopySnapshot",
            "ec2:RegisterImage",
            "ec2:Describe*"
         ],
         "Resource": "*"
      }
   ]
}
```
Add the policy to the vmimport role.
```
aws iam put-role-policy --role-name vmimport --policy-name vmimport --policy-document file://role-policy.json
```
Create a configuration file on your computer called **containers.json**.
Replace my-unique-bucket and myimage.ova with your bucket and image name.

[{ "Description": "Windows 11 Base Install", "Format": "ova", "UserBucket": { "S3Bucket": "my-unique-bucket", "S3Key": "myimage.ova" } }]

Create EC2 AMI from S3 OVA image
```
aws ec2 import-image --description "Windows 11" --disk-containers file://containers.json --region us-east-1
```
This may take a while you can check on the status of the import.
```
aws ec2 describe-import-image-tasks --region us-east-1
```
When the import status is completed you can create an instance based on that AMI. However you will get a message like:

**"...instance type does not support an AMI with TPM version v2.0. Specify an instance type that supports TPM version v2.0, and try again."**

if you select an instance type that does not support TPM version v2.0 The lowest priced [instances that I found that supports TPM version v2.0](https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/enable-nitrotpm-prerequisites.html) are C5 instances.
