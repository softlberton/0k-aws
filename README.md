# 0k - AWS CLI Guide

<!-- TOC -->

- [EC2](https://github.com/lbrealdev/0k-aws#ec2)
- [Security Groups](https://github.com/lbrealdev/0k-aws#security-groups)
- [EC2 AMI](https://github.com/lbrealdev/0k-aws#ec2-ami)
- [Secrets Manager](https://github.com/lbrealdev/0k-aws#secrets-manager)
- [VPC](https://github.com/lbrealdev/0k-aws#vpc)

### EC2

List EC2 instances by filtering by tag:name value with queries to display some reverse sort data fields for runtime with table output format:
```shell
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=<instance-name>" \
  --query "reverse(sort_by(Reservations[*].Instances[].{ID:InstanceId,Type:InstanceType,Status:State.Name,Init:LaunchTime,EC2Name:Tags[?Key == 'Name'].Value | [0]} &Init))" \
  --output table
```

### Security Groups

List security groups by filtering by security group name with query for IpPermissions (`Inbound Rules) of just UserIdGroupPairs (security group IDs) counting their length and json output format:
````shell
aws ec2 describe-security-groups \
  --filters "Name=group-name,Values=<security-group-name>" \
  --query "SecurityGroups[*].{Name:GroupName,Ingress:IpPermissions[].UserIdGroupPairs.length(@)}" \
  --output json
````

List security groups by filtering by name with query for IpPermissions (Inbound Rules) of just IpRanges (IPs/CIDRs) counting their length and json output format:
````shell
aws ec2 describe-security-groups \
  --filters "Name=group-name,Values=<security-group-name>" \
  --query "SecurityGroups[*].{Name:GroupName,Ingress: IpPermissions[].IpRanges.length(@)}" \
  --output json
````

List a specific security group:
```shell
aws ec2 describe-security-groups \
  --group-ids <security-group-id> \
  --query "SecurityGroups[*].{Name: GroupName, IngressRules: IpPermissions[].IpRanges, EgressRules: IpPermissionsEgress[]}" \
  --output json
```

### EC2 AMI

List all AMIs filtering by name by sorting the query by creation date of all images:  
```shell
aws ec2 describe-images \
  --filters "Name=name,Values=<ami-name-*>" \
  --query 'reverse(sort_by(Images[*], &CreationDate)[].Name)' \
  --output table
```

Get the latest AMI ID:
```shell
aws ec2 describe-images \
  --filters "Name=name,Values=<ami-name-*>" \
  --query 'sort_by(Images[*], &CreationDate)[-1].[ImageId]' \
  --output table
```

### Secrets Manager

Get secret value:
```shell
aws secretsmanager get-secret-value --secret-id "<secret-name>" --query "SecretString" --output text | jq .
```

### VPC

List all subnets showing as in table format:
```shell
aws ec2 describe-subnets \
 --query "sort_by(Subnets[].{Name:Tags[?Key == 'Name'].Value | [0],Vpc:VpcId,Id:SubnetId,AvailableIps:AvailableIpAddressCount} &Name)" \
 --output table
```

Describe the vpcs as in table format:
```shell
aws ec2 describe-vpcs \
  --query "Vpcs[].{Name:Tags[?Key == 'Name'].Value | [0],VpcId:VpcId,CIDR:CidrBlock,Account:OwnerId}" \
  --output table
```
