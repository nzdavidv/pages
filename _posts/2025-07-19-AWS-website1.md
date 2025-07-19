---
title: "AWS website using Lambda, API-Gateway, and S3 - part1"
date: 2025-07-19
---
# Overview
This is a repost from my www.nzvink.com website although I refreshed and re-tested the content.
**Web hosting using API gateway, Lambda, S3, all fronted by Cloudflare**

## Tasks overview

- Create a Lambda Python shell
- Modify IAM permissions
- Create an http API gateway, integrated with the Lambda
- Create an S3 bucket, upload a test html file
- Modify the Lambda code to read from the S3 bucket


# Lambda Python shell
Create Function, Author from Scratch, Python 3.13.
![aws-www-1](/pages/assets/images/aws-www-1.png){:width="800px"}

Configuration, General Configuration. Edit. Change the memory from 128 MB to 256 MB, save.
![aws-www-2](/pages/assets/images/aws-www-2.png){:width="800px"}

## Modify IAM permissions
In Lambda click Permissions, then click on the role name to launch IAM

Add permissions, Create inline policy

Click JSON, and paste the code below <a>https://github.com/nzdavidv/api-s3-website/blob/main/IAM-s3-read-all</a>. 
I named it 's3-read-all' and then clicked Create. TO-DO: refine the S3 permissions just to the bucket it needs.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "s3:List*",
                "s3:Get*",
                "s3:Describe*"
            ],
            "Resource": "*"
        }
    ]
}
```
to-do: refine the S3 permissions just to the bucket it needs.
Now the shell of the Lambda is complete.

# API gateway
API gateway, Create API, HTTP API, Build.

Add integration, Lambda, select the Lambda function, add a name for the API gateway, click Next

Change the Method to GET, enter /{proxy+} as the Resource path and click Next.
/{proxy+} tells the gateway to pass all requests through to the Lambda.

Leave auto-deploy on and $default stage name, Next, then Create.

### API gateway to Lambda testing
Click on Stages under Deploy then $default, then the invoke URL link and if it worked you should get a website with "Hello from Lambda!"
aws-www-8
