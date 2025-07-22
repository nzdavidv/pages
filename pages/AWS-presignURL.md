---
title: "AWS PreSigned URL with API Gateway"
date: 2024-03-31
---
This is a repost from my www.nzvink.com website. I didn't re-validate it but it looks simple and should still work.
# Overview
<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/aws-presign-00.png" alt="aws-presign-00" width="700px"></kbd>

# AWS simple PreSignedURL with API Gateway
This is a very stripped down simple upload using a PreSignURL POST method.

Tasks overview

- Setup S3 bucket
- Setup Lambda
- Setup IAM permissions
- Create cloudwatch log group
- Setup API Gateway

# Setup S3 bucket with CORS
S3 buckets, create bucket, I called mine 'nzvink-presignurl-test' and default options (no public access).
Go into the bucket and under permissions, Cross-origin resource sharing (CORS), edit, and paste the content below then save changes:
```
[
{
     "AllowedHeaders": [
        "*"
     ],
     "AllowedMethods": [
        "GET",
        "PUT",
        "POST",
        "HEAD",
        "DELETE"
     ],
     "AllowedOrigins": [
        "*"
     ],
     "ExposeHeaders": [],
     "MaxAgeSeconds": 3000
}
]
```

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/aws-presign-01.png" alt="aws-presign-01" width="500px"></kbd>

# Create PreSignURL Lambda
Lambda, Create function, author from scratch, function name (I called mine 'www-nzvink-presignurl-test'), latest python, x86_64, 'Create function'.
In the new function, under Configuration, change the memory to 256MB.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/aws-presign-02.png" alt="aws-presign-02" width="500px"></kbd>

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/aws-presign-03.png" alt="aws-presign-03" width="500px"></kbd>

## Lambda - add environment variables

BUCKETNAME - will be whatever made the bucketname. For me was nzvink-presignurl-test

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/aws-presign-04.png" alt="aws-presign-04" width="700px"></kbd>

## Lambda - add the code
Copy-paste code from 
<a href="https://github.com/nzdavidv/api-s3-website/blob/main/presignurlpost.py" target="_blank">Github PreSignURLPOST.py</a>

Deploy
Blah blah disclaimer.. it's bare minimum code, use at own peril etc.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/aws-presign-05.png" alt="aws-presign-05" width="700px"></kbd>

# Setup IAM role
In the Lambda function click configuration, permissions and then click on the role name.
Click Add permissions, create inline policy, click JSON, and paste content from below:
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
          "s3:Describe*",
          "s3:putObjectAcl",
          "s3:PutObject",
        ],
        "Resource": "*"
        }
     ]
}
```
Click Next.
Give it a name. I called mine aws-presign-s3
Create policy.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/aws-presign-06.png" alt="aws-presign-06" width="400px"></kbd>

# Setup cloudwatch log group
In Cloudwatch, Create log group with name of lambda function. Mine was /aws/lambda/www-nzvink-presignurl-test
I set the retention to 3 days because I like saving money. Then 'create'.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/aws-presign-07.png" alt="aws-presign-07" width="700px"></kbd>

# Setup API gateway
Create API, HTTP API 'build', Click 'Add integration' and select Lambda in the dropdown. Start typing the name of the lambda (for me www-nzvink-presignurl-test).

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/aws-presign-08.png" alt="aws-presign-08" width="700px"></kbd>

Give it a name, for me 'apigw-presignurl-test'. Next
The resource path must be /{proxy+}
Next

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/aws-presign-09.png" alt="aws-presign-09" width="700px"></kbd>

Default stages (auto-deploy). Next
Create


<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/aws-presign-10.png" alt="aws-presign-10" width="700px"></kbd>

Now if you click Stages, then tick the default radio selector you can see the invoke URL.
Mine is https://5bgcawrxwf.execute-api.us-east-1.amazonaws.com/

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/aws-presign-11.png" alt="aws-presign-11" width="700px"></kbd>

# Testing..
Yay, it works.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/aws-presign-12.png" alt="aws-presign-12" width="700px"></kbd>

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/aws-presign-13.png" alt="aws-presign-13" width="700px"></kbd>

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/aws-presign-14.png" alt="aws-presign-14" width="700px"></kbd>

I kinda have to turn it off now it is done though as it has no security and people could randomly fill up my S3 bucket.
