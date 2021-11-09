---
title: "Shell Commands"
date: 2021-11-09T17:45:28+01:00
draft: true
---

Useful AWS shell commands.

## S3 bucket tags

Bash command line to get tags for specific AWS S3 buckets.

```sh
for b in $(aws s3 ls | grep 'A-team\|drafts\|-mynotes-' | awk '{print $3}'); do echo $b; aws s3api get-bucket-tagging --bucket $b; done > tmp/s3_buckets_tags.txt
```

## Assume role to register Batch JobDefinition

This is a target of a Makefile to run within an agent of GoCD CI/CD platform to register a job-definition in AWS Batch assuming a role.

```Makefile
# This target register a job-definition on AWS
# To do so it assumes a role specified by an environment variable (ASSUME_ROLE_ARN)
# The job-definition is in the file batch-job-definition.json and it has a tag (IMAGE_TAG) which is replaced by the current pipeline version (GO_DEPENDENCY_LABEL_UPSTREAM)

release-job-definition:
	echo "Who am I for AWS? "
	aws sts get-caller-identity | jq
	
	role_creds=$$(aws sts assume-role --role-arn ${ASSUME_ROLE_ARN} --role-session-name ${GO_PIPELINE_NAME}) && \
	export AWS_ACCESS_KEY_ID=$$(echo $$role_creds | jq --raw-output .Credentials.AccessKeyId) && \
	export AWS_SECRET_ACCESS_KEY=$$(echo $$role_creds | jq --raw-output .Credentials.SecretAccessKey) && \
	export AWS_SESSION_TOKEN=$$(echo $$role_creds | jq --raw-output .Credentials.SessionToken) && \
	echo "Who am I for AWS after assuming the role ${ASSUME_ROLE_ARN}? " && \
	aws sts get-caller-identity | jq && \
	sed -i 's/IMAGE_TAG/'${GO_DEPENDENCY_LABEL_UPSTREAM}'/g' ./job-definitions/batch-job-definition.json && \
	cat ./job-definitions/batch-job-definition.json | jq && \
	aws batch register-job-definition --region eu-west-1 --cli-input-json file://`pwd`/./job-definitions/batch-job-definition.json
```
