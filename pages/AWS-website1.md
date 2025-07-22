---
title: "AWS website using Lambda, API-Gateway, and S3 - part1"
date: 2025-07-19
---
# Overview
This is a repost from my www.nzvink.com website although I refreshed and re-tested most of the content.
**Web hosting using API gateway, Lambda, S3, all fronted by Cloudflare**

## Tasks overview

- Create a Lambda Python shell
- Modify IAM permissions
- Create an http API gateway, integrated with the Lambda
- Create an S3 bucket, upload a test html file
- Modify the Lambda code to read from the S3 bucket


# Lambda Python shell
Create Function, Author from Scratch, Python 3.13.
<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/aws-www-1.png" alt="aws-www-1" width="1000px"></kbd>

Configuration, General Configuration. Edit. Change the memory from 128 MB to 256 MB, save.
<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/aws-www-2.png" alt="aws-www-2" width="1000px"></kbd>

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/aws-www-3.png" alt="aws-www-3" width="400px"></kbd>

## Modify IAM permissions
In Lambda click Permissions, then click on the role name to launch IAM
<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/aws-www-4.png" alt="aws-www-4" width="500px"></kbd>

Add permissions, Create inline policy

Click JSON, and paste the code below 
<a>https://github.com/nzdavidv/api-s3-website/blob/main/IAM-s3-read-all</a>. 
I named it 's3-read-all' and then clicked Create. 
TO-DO: refine the S3 permissions just to the bucket it needs.

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
<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/aws-www-5.png" alt="aws-www-5" width="1000px"></kbd>

Add integration, Lambda, select the Lambda function, add a name for the API gateway, click Next

Change the Method to GET, enter /{proxy+} as the Resource path and click Next.

/{proxy+} tells the gateway to pass all requests through to the Lambda.
<img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/aws-www-6.png" alt="aws-www-6" width="800px"></kbd>

Leave auto-deploy on and $default stage name, Next, then Create.

### API gateway to Lambda testing
Click on Stages under Deploy then $default, then the invoke URL link and if it worked you should get a website with "Hello from Lambda!"

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/aws-www-7.png" alt="aws-www-7"  width="800px"></kbd>

**Success!**

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/aws-www-8.png" alt="aws-www-8"  width="800px"></kbd>

# S3 bucket
Create bucket, give it a Bucket name and other options are defaults.
It might seem odd that Block all public access is left off, but this is because access is controlled via Lambda rather than provided to this bucket directly.


<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/aws-www-9.png" alt="aws-www-9"  width="800px"></kbd>

Upload a basic html test file. One provided at  
<a>https://github.com/nzdavidv/api-s3-website/blob/main/test.html</a>
You can upload a test image as well.
Click Upload, upload file with all defaults.

# Lambda code
caveat caveat.. I'm **really** not a developer, never have been a developer. I'm an ex-Unix geek.
Some Python basics.. Calls to print() go to the console not html returned to the browser.
Return body is what sends the html.
This part 'def lambda_handler(event, context):' is invoked with every Lambda call and the event contains lots of data you can use. The code I've written can be found on 
<a>https://github.com/nzdavidv/api-s3-website/blob/main/www-s3-lambda.py</a>
In Lambda, Copy and paste the new code into Code, lambda_function. Then click Deploy.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/aws-www-10.png" alt="aws-www-10"  width="800px"></kbd>

### Set the Lambda environment variable for the bucket name

Click, Configuration, Environment Variables, then Edit
Click Add environment variable
The key is BUCKETNAME and the value is whatever your bucket name is, then click Save

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/aws-www-11.png" alt="aws-www-11"  width="800px"></kbd>

### retesting
API gateway to Lambda re-testing
Appending test.html to the previous test website should now be able to access the uploaded test.html file. 
For me this was: https://glnw9kbzgb.execute-api.us-east-1.amazonaws.com/test.html

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/aws-www-12.png" alt="aws-www-12"  width="800px"></kbd>

You should be able to also see the test image that was uploaded (by changing the URL).

**Success again!** Next session still lots to be done..

- Add an API gateway custom domain, API mappings
- Cloudflare: setup the website, DNS, SSL

Next blog post is <a href="AWS-website2.md">AWS WWW blog Part2</a>
