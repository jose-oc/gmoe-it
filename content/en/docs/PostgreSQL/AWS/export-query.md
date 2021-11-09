---
title: "Export Query"
date: 2021-11-08T11:34:09+01:00
draft: true
resources:
- src: "**.{png,jpg}"
  title: "Image #:counter"
  params:
    byline: "Photo: Riona MacNamara / CC-BY-CA"
---

Using PostgreSQL in AWS RDS service you can export the result of a query to a S3 bucket. 


## Configuration

First of all, you have to set up some configuration. 
I assume you already have a bucket to copy the results into.

### IAM Role

You have to create a role with permissions to copy the file into a bucket and attached this role to your RDS database.

There are different ways of doing this, you can create it directly in AWS web console, using aws-cli, etc. 
I'll show here the piece of terraform code to do so:

```terraform
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "3.63.0"
    }
  }

  backend "s3" {
    bucket = "gmoe-eu-west-1-terraform-state"
    key    = "rds/terraform.tfstate"
    region = "eu-west-1"
  }
}

provider "aws" {
  allowed_account_ids = [local.aws_account_id]
  default_tags {
    tags = {
      App       = local.app
      BuiltWith = "terraform"
      Colour    = local.colour
      Stage     = local.stage
    }
  }
  region = local.aws_region
}

locals {
  app              = "rds-exportS3-gmoe"
  aws_account_id   = "012345678900"
  aws_account_name = "gmoe"
  aws_region       = "eu-west-1"
  colour           = "red"
  stack            = "rds-demo"
  stage            = "sandbox"
}

# The bucket (already created) to copy data into
data "aws_s3_bucket" "rds_s3export" {
  bucket = "gmoe-rds-mydbname"
}

# Role to be assumed by RDS
resource "aws_iam_role" "rds_s3export" {
  name = "gmoe_rds_s3export"

  assume_role_policy = jsonencode({
    Version = "2012-10-17",
    Statement = [
      {
        Action = "sts:AssumeRole",
        Effect = "Allow",
        Principal = {
          Service = "rds.amazonaws.com"
        }
      }
    ]
  })
}

# Policy to allow copy the data into the S3 bucket
resource "aws_iam_role_policy" "rds_s3_export" {
  name = "gmoe_rds_s3_export"
  role = aws_iam_role.rds_s3export.id

  policy = jsonencode({
    Version = "2012-10-17",
    Statement = [
      {
        Effect = "Allow",
        Action = [
          "s3:AbortMultipartUpload",
          "S3:PutObject"
        ],
        Resource = [
          "${data.aws_s3_bucket.rds_s3export.arn}/*"
        ]
      }
    ]
  })
}

# Link the role to the RDS database
resource "aws_rds_cluster_role_association" "prd_flow_rds_s3_export" {
  db_cluster_identifier = "arn:aws:rds:eu-west-1:012345678900:cluster:gmoe-db-cluster"
  feature_name          = "s3Export"
  role_arn              = aws_iam_role.gmoe_rds_s3export.arn
}
```


### Postgres extension

Get into your database (you can use different clients such as _psql_ or _pgcli_) and enable this extension:

```postgresql
CREATE EXTENSION IF NOT EXISTS aws_s3 CASCADE;
```

## Export your query

Now you're ready to run your query and export the results to your bucket. 
You can do this using different but similar commands:

```postgresql
SELECT * FROM aws_s3.query_export_to_s3(
    'SELECT now()', 
    'gmoe-rds-mydbname',
    'ping/now-query.txt',
    'eu-west-1',
    options :='format csv, header true'
);
```

In that example we are just exporting the result of `SELECT now()` and we specify the bucket name, file path and the AWS region.

{{< alert >}}
The `options` variable is optional, if omitted the file format would be text.
{{< /alert >}}

You can, instead, create a variable with the location you want to use to export your data and then use it in the query:

```postgresql
SELECT aws_commons.create_s3_uri(
   'gmoe-rds-mydbname',
   'path/now-query.txt',
   'eu-east-1'
) AS s3_export_uri \gset

SELECT * FROM aws_s3.query_export_to_s3('SELECT now()', :'s3_export_uri');
```

{{< alert color="warning" >}}
I've found out that this second way works properly with _psql_ but not with _pgcli_.
{{< /alert >}}


As a response of that query you'll have information about the amount of data copied into the bucket:

```postgresql
+-----------------+------------------+------------------+
| rows_uploaded   | files_uploaded   | bytes_uploaded   |
|-----------------+------------------+------------------|
| 1               | 1                | 30               |
+-----------------+------------------+------------------+
SELECT 1
Time: 0.262s
```



{{< alert color="success" >}}
You probably have to connect to the database as the owner (postgres user in my case).
{{< /alert >}}




## Troubleshooting

### Timeout

In case you get a timeout trying to export a query you'll read a message like this:

```postgresql
mydbname> SELECT * FROM aws_s3.query_export_to_s3(
     'SELECT now()',
     'gmoe-rds-mydbname',
     'ping/now-query.txt',
     'eu-west-1'
 );
could not upload to Amazon S3
DETAIL:  Amazon S3 client returned 'Unable to connect to endpoint'.

Time: 117.469s (1 minute 57 seconds), executed in: 117.469s (1 minute 57 seconds)
```

This might happen due to the _security group_ configuration used by RDS. 
Check out the Security Group the RDS is using, verify that the outbound rules allow to connect to S3 via HTTPS. 

In AWS web console go to RDS service and get into your DB Instance, select one of your instances (in case you have more than one) and have a look at the _Connectivity & security_ tab, look for _VPC security groups_. There you see the security group RDS is using, you can click on it to go to its details. 

![](/aws-rds-security-group-id.png)

I hit this problem because the Security Group I was using didn't have any outbound rule defined so no connections were allowed to go out from the database.

![](/aws-security-group-outbound-rules-empty.png)

I solve this issue by adding a new outbound rule with this configuration: 

![](/aws-security-group-outbound-rule-allow-s3-via-https-creation.png)

![](/aws-security-group-outbound-rule-allow-s3-via-https.png)

You can see on the above screenshot that the outbound rule _destination_ was a prefix list (pl-xxxx). I selected this destination to avoid giving a broad output rule to connect to anywhere but to restrict it just to connect to S3 endpoints. 
You can find the _Prefix Lists_ you have available in your AWS account under the VPC service:

![](/aws-vpc-prefix-list-s3.png)

As you can see I have two of them, and the one I selected is configured to cover the S3 CIDRs.


## Caveats

Not all RDS versions allow this feature so you should verify this point to ensure this works for you, nevertheless recent versions won't have any problem with this.

```sh
YOUR_RDS_VERSION=12.4
aws rds describe-db-engine-versions --region us-east-1 --engine postgres --engine-version ${YOUR_RDS_VERSION} | grep s3Export
```