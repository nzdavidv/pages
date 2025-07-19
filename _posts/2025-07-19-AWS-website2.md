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
