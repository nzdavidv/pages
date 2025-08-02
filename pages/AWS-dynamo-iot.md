---
title: "AWS Lambda DynamoDB IoT website"
date: 2024-01
---
# Overview
> I migrated this content from my old website www.nzvink.com to github. I have not revalidated the AWS screens.

Blog overview: Raspberry Pi pushing IoT data into DynamoDB, and API Gateway with Python Lambda to query and present.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/raspi-dd-pic1.png" alt="raspi-dd-pic1" width="900px"></kbd>

## Objective
The objective was to be able to see my BBQ temperatures when away from home.. in particular for slow-cooking sessions.
The main technology used is Linux running Ubuntu (at home) which queries the IoT devices and then sends data to DynamoDB on a schedule.

There is also an API gateway and Lambda for querying / presenting the IoT data back.

## Tasks overview
- Setup DynamoDB tables
- Setup IAM user
- Setup Linux machine
- Create the Lambda Python 3.8 function
- Create an REST API gateway, integrated with the Lambda
- Create the html form code
