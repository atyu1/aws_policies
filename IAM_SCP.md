# IAM and SCP Policies and examples

### General IAM Policy to access S3
- allow user atyu to access S3 bucket "examplebucket"

```
{
    "Version": "2012-10-17",
    "Id": "S3-Account-Permission",
    "Statement": [
        {
            "Sid": "User-Atyu-Allowed",
            "Effect": "Allow",
            "Principal": {
                "AWS": ["arn:aws:iam::123456789012:root"]
            },
            "Action": [
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": ["arn:aws:s3::examplebucket/*],
            "Condition": {
                "StringEquals": {
                    "aws:username": "atyu"
                }
            }
        }
    ]
}
```

### Not Action 
- Does allow everything but iam
- iam is further processed with other rules or default action
- not action can be used also to further process the service in resource policy
- allow with notaction means that all listed items are further processed in the chain of authorization

```
{
    "Version": "2012-10-17",
    "Id": "Not-Action-Example",
    "Statement": [
        {
            "Sid": "Not-Action",
            "Effect": "Allow",
            "NotAction": [
                "iam:*"
            ],
            "Resource": "*"
        }
    ]
}
```

### Not Action for MFA
- Deny all services except IAM, if MFA is not used

```
{
    "Version": "2012-10-17",
    "Id": "Not-Action-MFA",
    "Statement": [
        {
            "Sid": "Not-Action-MFA",
            "Effect": "Deny",
            "NotAction": [
                "iam:*"
            ],
            "Resource": "*",
            "Condition": {
                "BoolIfExists": {
                    "aws:MultiFactorAuthPresent": "False"
                }
            }
        }
    ]
}
```

### Deny Not Action for specific region
- Deny all other services which are outside of eu-central one and exceptions are cloudfront, iam, route53 and support
- Those 4 services are global so we need to allow them even if we block other regions

```
{
    "Version": "2012-10-17",
    "Id": "DenyRegions",
    "Statement": [
        {
            "Sid": "AllowOnlyEuCentral1",
            "Effect": "Deny",
            "NotAction": [
                "cloudfront:*",
                "iam:*",
                "route53:*",
                "support:*"
            ],
            "Resource": "*",
            "Condition": {
                "StringNotEquals": {
                    "aws:RequestedRegion": ["eu-central-1"]
                }
            }
        }
    ]
}
```


## Principals
### Full organization to specify
```
"Principal": {"AWS": "123456789012"}
"Principal": {"AWS": "arn:aws:iam::123456789012:root"}
```

### Specific role
```
"Principal": {"AWS": "arn:aws:iam::123456789012:role/my-custom-role"}
```

### Role sessions
- like assumed role or via federated
```
"Principal": {"AWS": "arn:aws:sts::123456789012:assumed-role/my-custom-role/my-role-session"}
"Principal": {"Federated": "cognito-identity.amazonaws.com"}
"Principal": {"Federated": "arn:aws:iam::123456789012:saml-provider/provider-name"}
```

### Users
```
"Principal": {"AWS": "arn:aws:iam::123456789012:user/my-user"}
```

### Federated user sessions
```
"Principal": {"AWS": "arn:aws:sts::123456789012:federated-user/my-fed-user"}
```

### Specific service 
- exmaple cloudwatch and ecs

```
"Principal": {"Service": ["ecs.amazonaws.com, cloudwatch.amazonaws.com"]}
```

## Conditions
### String related
- StringEquals/StringNotEquals - case sensitive and exact match, for example tag on resource

```
"StringEquals": {"aws:PrincipalTag/environment": "development"}
```

- StringLike/StringNotLike - case sensitive, partial match by *, for example check s3 prefix is app1 related,
- Symbol * can be used for any string 
- Support variables like ${aws:username}

```
"StringLike": {"s3:prefix": ["App1", "Application1", "Applications/*/App1]}
```

### Date Related
- DateEquals/DateLessThan/DateGreaterThan - compare dates, example, validate if token is newer then specified date

```
"DateGreaterThan": {"aws:TokenIssueTime": "2023-10-11T15:00:00Z"}
```

### Arn manipulation
- ArnLike/ArnNotLike - checks the syntax of arn

```
"ArnLike": {"aws:s3arn": "arn:aws:iam::123456789012:root"}
```

### Booleans
- Bool - checks if true or false
  
```
"Bool": {"aws:SecureTransport": "false"}
```

### IP address
- IPAddress/NotIPAddress - validates if IP is part of CIDR
- `Applies only to Public IPs!!!` not applicable to request via VPC Endpoints
```
"IPAddress" : { "aws:SourceIp": "201.111.0.0/24"}
```

### Request region
- global condition
- validates if request is from any specific region
- used with string Equals

```
"StringEquals": 
  {
    "aws:RequestedRegion": [ 
    "eu-central-1",
    "eu-west-1"
  ] 
}
```

- some services are global and located in us-east-1 so use NotAction + Deny with Regions, see earlier

### Token Issue time validation
- Block users if token is older then a date
- Good approach when token invalidation is needed
- Example is everything before 10th of October match the condition

```
"DateLessThen": 
  {
    "aws:TokenIssueTime": "2023-10-10T00:00:00.000Z"
}
```

## Principals
### PrincipalARN vs ServiceARN
- Principal - is user, role, root
- Service - if service makes a request, like S3


### PrincipalTag vs ResourceTag
- Principal - tag on user,role
- Resource - tag on service, like EC2

## Other

### ABAC example
- limit by SCP to start/stop instances only if tag owner and AccessProject match

```
{
    "Version": "2012-10-17",
    "Id": "Abac",
    "Statement": [
        {
            "Sid": "Abacrule1",
            "Effect": "Allow",
            "Action": [
                "ec2:startInstance",
                "ec2:stopInstance"
            ],
            "Resource": "arn:aws:ec2:*:123456789012:instance/*",
            "Condition": {
                "StringEquals": {
                    "aws:ResourceTag/Owner": "${aws:username}",
                    "aws:ResourceTag/AccessProject": "${aws:PrincipalTag/AccessProject}"
                }
            }
        }
    ]
}
```

### MFA check
- Allow anything on EC2 but stop/start only if user have MFA in his account

```
{
    "Version": "2012-10-17",
    "Id": "MFA",
    "Statement": [
        {
            "Sid": "mfarule1",
            "Effect": "Allow",
            "Action": ["ec2:*"]
        },{
            "Sid": "mfarule2",
            "Effect": "Deny",
            "Action": [
                "ec2:startInstance",
                "ec2:stopInstance"
            ],
            "Resource": "*",
            "Condition": {
                "BoolIfExists": {
                    "aws:MultiFactorAuthPresents": false
                }
            }
        }
    ]
}
```

### Restric EC2 types to t2.micro
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "RequireMicroInstanceType",
      "Effect": "Deny",
      "Action": "ec2:RunInstances",
      "Resource": [
        "arn:aws:ec2:*:*:instance/*"
      ],
      "Condition": {
        "StringNotEquals": {
          "ec2:InstanceType": "t2.micro"
        }
      }
    }
  ]
}
```

### Restrict IMDSv2 only

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "RequireIMDSv2",
      "Effect": "Deny",
      "Action": "ec2:RunInstances",
      "Resource": [
        "arn:aws:ec2:*:*:instance/*"
      ],
      "Condition": {
        "StringNotEquals": {
          "ec2:MetadataHttpTokens": "required"
        }
      }
    }
  ]
}
```

## Exposed Secret and Access Key
- If some keys were exposed we need to deactive and delete them
- That is not enough as there we can create session token from that key and that still will be valid
- To block Session Token, we need to attach the following IAM policy to the User, which keys are exposed

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "StopAttacker",
      "Effect": "Deny",
      "Action": "*",
      "Resource": [
        "*"
      ],
      "Condition": {
        "DateLessThan": {
          "aws:TokenIssueTime": "2024-11-09T19:38:00Z"
        }
      }
    }
  ]
}
```