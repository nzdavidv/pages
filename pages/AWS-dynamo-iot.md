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

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/raspi-dd-pic5.png" alt="raspi-dd-pic5.png" width="800px"></kbd>

Click on the group, Permissions, Add permissions, Add inline policy

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/raspi-dd-pic6.png" alt="raspi-dd-pic6.png" width="800px"></kbd>

Click JSON, paste the content below and click Review policy
```
{
    "Version": "2012-10-17",
    "Statement": [
      {
        "Sid": "VisualEditor0",
        "Effect": "Allow",
        "Action": [
          "dynamodb:BatchWriteItem",
          "dynamodb:PutItem"
        ],
        "Resource": "*"
      }
    ]
}
```

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/raspi-dd-pic2.png" alt="raspi-dd-pic2.png" width="800px"></kbd> 

Give it a name and click Create policy

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/raspi-dd-pic2b.png" alt="raspi-dd-pic2b.png" width="800px"></kbd> 

## IAM user setup
IAM, Users, add users, give it a Name - I called mine DynamoDB-raspi.
Credential type is Access Key, Next, add user to the group created earlier, next, review, create user

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/dd-iot-6.png" alt="dd-iot-6.png" width="800px"></kbd>

Next click on the user, Security Credentials, scroll down to 'Access keys' and click 'Create access key'.
Use case is 'CLI', tick the acknowledgement down the bottom and then create. Make sure to copy the access key and secret somewhere safe.

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/raspi-dd-pic7.png" alt="raspi-dd-pic7.png" width="800px"></kbd>

# Setup Linux machine
## Useradd and AWS base setup

I added a user, and then followed the instructions at <a>https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html</a>
```
# apt-get install unzip curl
For my Ubuntu laptop:
# curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
For my Raspberry Pi:
# curl "https://awscli.amazonaws.com/awscli-exe-linux-aarch64.zip" -o "awscliv2.zip"
..the rest is the same for both:
# unzip awscliv2.zip
# ./aws/install
# su - www
$ aws configure
AWS Access Key ID [None]: .....
AWS Secret Access Key [None]: ....
Default region name [None]: us-east-1
Default output format [None]: json
```

### Basic AWS DynamoDB test
```
$  aws dynamodb scan --table-name devnames
```

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/dd-iot-8.png" alt="dd-iot-8.png" width="500px"></kbd>

Looking good.

### Create 'send to AWS' script
The code here <a>https://github.com/nzdavidv/api-s3-website/blob/main/send-temps-to-aws.sh</a> is a very basic shell script to send temp data to AWS.
The idea is that it can be called from a variety of other scripts.. either the igrill python script for my food thermometer (uses https://github.com/bendikwa/igrill and a shell script to query MQTT / mosquitto) or from the cheap-and-cheerful bluetooth thermometers I have and corresponding python code from https://github.com/JsBergbau/MiTemperature2.
I'm happy to share any of the add-on scripts I wrote to join the python code from the websites mentioned to the script.

# Setup AWS API gateway and Lambda
## Lambda function
The Lambda turned into a bit of a monster.. the idea is when it is called with a GET with no parameters it returns a summary page, and the summary page contains some javascript to create and submit a form.
The form sends data to the same Lambda but via POST method. When the Lambda is called via the POST method it returns a graph depending on the parameters it was called with.
I'm not a developer by any stretch. Fair to say I hate python.. but here it is.. <a>https://github.com/nzdavidv/api-s3-website/blob/main/dd-html.py</a> 

Anyhoo.. setting up the Lambda:
- Create function, give it a name, runtime is python
- Copy-paste code from Github into the Code section, Deploy
- In Configuration, General config and change the memory to 256mb and timeout to 4 seconds
- In Configuration, Permissions click the role name to open the IAM role permissions page

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/dd-iot-9.png" alt="dd-iot-9.png" width="500px"></kbd>

- Add permissions, attach policy 'AmazonDynamoDBReadOnlyAccess'
- Optional but you should be able to test the Lambda (with any data) and it return a 200 and some html.

## API Gateway
- Create API, HTTP API, Build
- Click Add integration, Lambda, select the Lambda function just created, give it a name

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/dd-iot-10.png" alt="dd-iot-10.png" width="500px"></kbd>

- The Method can be ANY, because it accepts POST and GET
- Default stage name and auto-deploy
- Optional.. can add a custom domain name

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/dd-iot-11.png" alt="dd-iot-11.png" width="500px"></kbd>

## Tweak and test

The Lambda code references to which would need changing
- Line 62 ref's my API gateway .. form.action = "https://ifcf44wt0j.execute-api.us-east-1.amazonaws.com";
- Line 323 has a reference to a style sheet on my website.
testing testing..

### How I use it in real life

There are two uses.. one is for my Weber BBQ meat temp probe with code from igrill - <a>https://github.com/bendikwa/igrill</a>

..Not detailed that here but happy to share.


The other is for the Xiaomi Mi Home Temperature & Humidity Monitor 2
<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/mi2.jpg" alt="mi2.jpg" width="300px"></kbd>

There was a bunch of setup I did prior to this to get it working, also not detailed here (but happy to share).
The code is from <a>https://github.com/JsBergbau/MiTemperature2</a>

What I will show is how I augment the MiTemperature2 code.

I have shell script that run via crontab
www@raspidev:~/git/raspi-www-bin$ crontab -l|grep temps
0,20,40 0,1,2,3,4,5,22,23 * * * /home/www/temps.sh 1>/home/www/cronouttemps 2>&1
0,10,20,30,40,50 6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21 * * * /home/www/temps.sh 1>/home/www/cronouttemps 2>&1

This runs temps.sh, as below:
```
#!/bin/bash

cd /home/www/mitemp2/
source bin/activate

#check processes not still running..
pidcount=`ps -fu www|grep "LYWSD03MMC.py"|wc -l`
if [ $pidcount -gt 0 ]; then
 for pid in `ps -fu www|grep "LYWSD03MMC.py" |awk '{print $2}'` ; do
  kill $pid
 done
fi
echo "pidcount $pidcount"

for sensor in  A4:C1:38:CF:53:B0 A4:C1:38:DF:F1:7E A4:C1:38:05:D6:64
 do
 ./MiTemperature2/LYWSD03MMC.py -d ${sensor} -r -b -c 1 --callback sendToMySQL.sh
done
```
The callback script sendToMySQL.sh is pretty simple. It writes it to a local database on the raspberry pi and then sends it to AWS.

The IOTSQLUSER and IOTSQLPASSWD variables are picked up from the environment file ~www/.env

sendToMySQL.sh:
```
#!/bin/bash

# $2 sensormac
# $3 temp
# $4 humidity
# $5 voltage

. ~www/.env

datenow=`date "+%Y%m%d %H:%M"`
  datesh=`echo "$datenow"|awk '{print $1}'`
  timesh=`echo "$datenow"|awk '{print $2}'`

INSERT="INSERT INTO homeiot.temps (sensormac, temp, humidity, voltage, ping_date, ping_time)"
echo "${INSERT}" > sqldata.txt
echo "VALUES ('$2', '$3', '$4', '$5', '${datesh}', '${timesh}');" >>  sqldata.txt
mysql --user=${IOTSQLUSER} --password=${IOTSQLPASSWD} --host=localhost homeiot < sqldata.txt

/home/www/send-temps-to-aws.sh $2 $3 $4 $5

rm sqldata.txt

exit 0
```

Finally the send-temps-to-aws.sh script:
```
#!/bin/bash

basedir=/home/www
TMPDIR=$basedir/tmp
if [ ! -d $TMPDIR ]; then mkdir $TMPDIR ; fi
AWSFILE=$TMPDIR/aws-infile2.json

macaddr=$1
temp=$2
humidity=$3
battery=$4

if [ "$macaddr" == "" ] || [ "$temp" == "" ] || [ "$humidity" = "" ] || [ "$battery" == "" ]; then
 echo "usage: $0 mac temp humidity battery"
 exit 1
fi


 datenow=`date "+%Y%m%d %H:%M"`
 datestr=`date "+%d-%m-%Y"`
 timestr=`date "+%H:%M:%S"`
 unixdate=`date "+%s"`
 datesh=`echo "$datenow"|awk '{print $1}'`
 timesh=`echo "$datenow"|awk '{print $2}'`

#example JSON:

#{
#    "datestr": {
#      "S": "03-09-2022"
#    },
#    "macdatetime": {
#      "S": "70:91:8F:9A:D2:63-1662138901"
#    },
#    "datetime": {
#      "N": "1662138901"
#    },
#    "temp": {
#      "N": "72"
#    },
#    "humidity": {
#      "N": "0"
#    },
#    "battery": {
#      "N": "80"
#    },
#    "macaddr": {
#      "S": "70:91:8F:9A:D2:63"
#    }
#}


 echo -e "{\n\t\"datestr\": {\"S\": \"${datestr}\"}," > $AWSFILE
 echo -e "\t\"timestr\": {\"S\": \"${timestr}\"}," >> $AWSFILE
 echo -e "\t\"macdatetime\": {\"S\": \"${macaddr}-${unixdate}\"}," >> $AWSFILE
 echo -e "\t\"datetime\": {\"N\": \"${unixdate}\"}," >> $AWSFILE
 echo -e "\t\"temp\": {\"N\": \"${temp}\"}," >> $AWSFILE
 echo -e "\t\"humidity\": {\"N\": \"${humidity}\"}," >> $AWSFILE
 echo -e "\t\"battery\": {\"N\": \"${battery}\"}," >> $AWSFILE
 echo -e "\t\"macaddr\": {\"S\": \"${macaddr}\"}\n}" >> $AWSFILE

 aws dynamodb put-item --table-name temps --item file://${AWSFILE}
```

### testing

<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/dd-iot-13.png" alt="dd-iot-13.png" width="500px"></kbd>
<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/dd-iot-14.png" alt="dd-iot-14.png" width="500px"></kbd>
<kbd><img src= "https://raw.githubusercontent.com/nzdavidv/pages/refs/heads/main/images/dd-iot-15.png" alt="dd-iot-15.png" width="500px"></kbd>

### 
