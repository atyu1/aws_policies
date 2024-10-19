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