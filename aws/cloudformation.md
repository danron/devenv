# Cloud Formation

## Create a stack

```bash
#!/bin/bash

aws cloudformation create-stack --stack-name foo-stack --template-body file://cloudformation/templates/foostack-template.json --parameters  ParameterKey=AccessKeyId,ParameterValue=$AWS_ACCESS_KEY_ID ParameterKey=SecretAccessKey,ParameterValue=$AWS_SECRET_ACCESS_KEY ParameterKey=ImageId,ParameterValue=$LATEST_AMI ParameterKey=NodeCount,ParameterValue=${NODE_COUNT} ParameterKey=CloudInitCfg,ParameterValue="$CloudInit"
```

## Delete a stack

```
aws cloudformation delete-stack --stack-name foo-stack
```

## Query AMIs

```
# Query latest AMI

aws --output text --region eu-west-1 ec2 describe-images --owners 0123456789 --query 'Images[?starts_with(Name,`foo-centos7-barservice`)]|sort_by(@,&Name)[-1].ImageId'

# Query latest AMI based on tag

aws ec2 describe-images --region eu-west-1 --output text --query 'Images[*].{ID:ImageId}' --filters "Name=tag:FOO_VERSION,Values=latest"

```

## Tag a resource

```
aws ec2 create-tags --resources vol-335f8c83 --tags Key=Team,Value=systest
```

## Delete a volume

```
aws ec2 delete-volume --volume-id vol-335f8c83
```

## Query stack for IP

```
aws ec2 describe-instances --query "Reservations[].Instances[].[Tags[?Key=='aws:cloudformation:stack-name'] | [0].Value, PrivateIpAddress]" --filters Name=tag:aws:cloudformation:stack-name,Values=foo-stack\* Name=instance-state-name,Values=running --output text | sort
```
