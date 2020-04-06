---
title: Loading CSV files using AWS Lambda
date: 2020-04-20T12:00:06+09:00
description: Showcasing a nifty integration between S3, Lambdas and Amazon Aurora for loading CSVs to a database easily
draft: false
hideToc: true
enableToc: false
enableTocContent: false
tags:
- lambda
- aws
- aurora
series:
- AWS Patterns
image: images/lambda.png
---

This article is showcasing a nifty integration between S3, Lambdas and Amazon Aurora for loading CSV files to a database in a service-oriented way. Whereas any type of text file can be used, I will stick to `.csv`. 
<!--more-->

## Use case

Some times loading CSVs to a database is a one-off job. A convoluted SQL script is probably created and then deployed by someone who has the right permission.
Other times though, it might be part of a recurring business process in which case we should encapsulate that file loading logic into a service. If you are on the AWS Platform, this can be done in a clean way.

## Setup

The proposed setup is the following, a file gets uploading to an S3 Bucket which fires off an event picked up by the Lambda function. Lambda can consequently doctor the Aurora database to load the file directly from S3 using a `LOAD DATA FROM S3` statement.

#### 
      LOAD DATA FROM S3 's3://ed-match-data-dropper-stack-match-databucket-1egcf9v3ca03e/E0.csv'
      INTO TABLE match_data.football_match_data
      FIELDS TERMINATED BY ','
      LINES TERMINATED BY '\n'
      IGNORE 1 ROWS
      (division, @match_date, @ignore_time, home_team, away_team, ft_hg, ft_ag, ft_r, ht_hg, ht_ag, ht_r)
      SET match_date = STR_TO_DATE(@match_date, '%d/%m/%Y');

This is an Amazon Aurora feature and comes with some caveats explained in full on [AWS docs](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Integrating.LoadFromS3.html).
Having the Lambda only initiate the process and Aurora to actually load the data reduces the amount of code and cost required to a minimum. The Lambda is billed only when it is executed so you can safely leave the service deployed for whenever it is needed again.

## User experience

#### Files get uploaded to the bucket

![Example image](/images/s3-bucket-csv.png)

#### Lambda triggers the loading

![Example image](/images/cloudwatch-lambda-csv.png)


## Implementation

There are a lot of ways to implement this. I preferred to use an infrastructure-as-code template that spins up the Lambda and its S3 bucket while it configures them to access an already existing database. [Code here](https://github.com/earnest-developer/ed-match-data-dropper).