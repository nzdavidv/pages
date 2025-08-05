---
title: "AWS website using Lambda, API-Gateway, and S3 - part2"
date: 2025-07-19
---
# Overview
This is a repost from my www.nzvink.com website although I refreshed and re-tested the content.
**Web hosting using API gateway, Lambda, S3, all fronted by Cloudflare**

This continues the setup of API gateway, Lambda, S3, all fronted by Cloudflare started earlier. 
I left off with test.html being served via API gateway to Lambda which was accessing files in the S3 bucket.
Still a few things to be done..

- Clone and modify some html code and images, upload to the buckets
- Create a custom domain for the API gateway, and API mappings
- Cloudflare: setup the website, DNS, SSL

## Source and modify some html templates
I sourced some html templates from html codex <a href="https://htmlcodex.com">html codex</a> and downloaded them locally for editing with VSCode.
For the test website I downloaded construction-company-website-template-free.

I uploaded the base html files using s3 'upload file' and then for the folders img etc I used 'upload folders'.
Upload all of the images, .css, and .js files as-is to the s3 bucket

https://glnw9kbzgb.execute-api.us-east-1.amazonaws.com/

It seemed to work.

## Cloudflare initial setup

**note: 2025-07-29 - I haven't re-validated the setup below for cloudflare, AWS certs,  AWS custom domain name..

I followed Cloudflare instructions to setup site with a free plan.
I changed my DNS to use Cloudflare Nameservers with my Registrar.
The next step was to create a Cloudflare certificate.
Under SSL/TLS, Origin Server, 'Create Certificate', I created the cert and downloaded the cert and private key. It was important to get the RSA PEM origin root cert for the cert chain in the import.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/aws-www-13.png" alt="aws-www-13"  width="800px"></kbd>

### AWS Certificate manager
In AWS open Certificate manager then 'Import'. Paste the cert and private key.
It needs to say Imported and issued when done.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/aws-www-14.png" alt="aws-www-14"  width="800px"></kbd>

### API gateway custom domain
Open API Gateway, Custom domain names, then Create. The cert should be able to be selected in the Create Domain Name, ACM Certificate section.
Then click 'Create Domain Name'.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/aws-www-15.png" alt="aws-www-15"  width="800px"></kbd>

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/aws-www-16.png" alt="aws-www-16"  width="800px"></kbd>

Under API Gateway, Custom domain names, click API mappings, and then Configure API mappings.
Add new mapping. Select the API, Stage, and Save. I also like to set Throttling to ensure I don't get hit with an unexpected bill but this would not be appropriate for a high volume website. Now the API gateway is setup to handle the new domain, but nothing has told DNS to direct traffic to it.. Copy the invoke URL domain part (without the https://)

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/aws-www-17.png" alt="aws-www-17"  width="800px"></kbd>

### Cloudflare DNS
In Cloudflare click the website, then DNS.
Click 'Add record'. The type is CNAME and add the name and target.
Under SSL/TLS Overview, the encryption mode needs to be set to Full.

### Final testing
It's alive and works..

## Post-note
Pounding my head on the desk as I made a change to the Python (fixed the .css file handler I had made a mistake with earlier) and it wasn't working.
Nothing in Cloudwatch logs at all. I was hitting the URL and getting something returned (null file with a 200 because of my error - but it should now be fixed) but not showing anything in the Cloudwatch logs.

What am I missing..?

Anyway after a bit I hit the original API gateway (not the custom domain name) and it returned the style.css.

Huh? how can it work via the xx.execute-api.us-east-1.amazonaws.com but fail via the custom domain name?!

Then it hit me.. Cloudflare was caching the failure. Dammit.

Cloudflare cache for 2 hours I think on the free plan so eventually it should come right. Will check again tomorrow.
