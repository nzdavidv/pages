---
title: "AWS Lambda DynamoDB IoT website"
date: 2024-01
---
# Overview
> I migrated this content from my old website www.nzvink.com to github. I
> 
>  have not revalidated the AWS screens.

### Blog overview
Raspberry Pi pushing IoT data into DynamoDB, and API Gateway with Python Lambda to query and present.

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

# Setup DynamoDB tables
I learnt a bit about DynamoDB design doing this and iterated several times until I got something that worked for me. 

The end solution was 2 tables: one for device names (mac address to names table), and one for IoT data.

### Device names table
In AWS DynamoDB, Tables, Create table.
The first table 'devnames' has the partition key as 'macaddr', and it doesn't have a sort key.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/dd-iot-1.png" alt="dd-iot-1.png" width="600px"></kbd>

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/dd-iot-2.png" alt="dd-iot-2.png" width="600px"></kbd>

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/dd-iot-3.png" alt="dd-iot-3.png" width="600px"></kbd>

Add an entry in the table.. click the table name, then Explore table items, then Create item.
Click JSON view, then copy-paste the entry below, scroll down and click Create. An example entry in JSON:
```
{
    "macaddr": {
      "S": "70:91:8F:9A:D2:63"
    },
    "devname": {
      "S": "iGrill-mini"     }
}
```
The two entries are MAC address (macaddr) and a friendly name for the MAC address (devname).

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/dd-iot-4.png" alt="dd-iot-4.png" width="600px"></kbd>

### Temps table
The second table is called 'temps' with a partition key 'datestr', and sort key of 'macdatetime'.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/dd-iot-5.png" alt="dd-iot-5.png" width="600px"></kbd>

An example entry in JSON:
```
{
    "datestr": {
      "S": "03-09-2022"
    },
    "macdatetime": {
      "S": "70:91:8F:9A:D2:63-1662138901"
    },
    "datetime": {
      "N": "1662138901"
    },
    "temp": {
      "N": "72"
    },
    "humidity": {
      "N": "0"
    },
    "battery": {
      "N": "80"
    },
    "macaddr": {
      "S": "70:91:8F:9A:D2:63"
    }
}
```

The entries are:

- datestr: The date as a string
- macdatetime: MAC-address and Date/time in Unix Epoch format
- datetime: Date/time in Unix Epoch format
- temp: temperature
- humidity: humidity
- battery: battery voltage or percentage life
- macaddr: MAC-address

This doesn't need to be created manually.

# Setup IAM group and user
## IAM group
IAM, user groups, Create group, give it a name (I chose 'dynamodbgroup'), search for and add AmazonDynamoDBReadOnlyAccess permissions, Create Group

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/raspi-dd-pic4.png" alt="raspi-dd-pic4.png" width="400px"></kbd>

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/raspi-dd-pic5.png" alt="raspi-dd-pic5.png" width="600px"></kbd>


