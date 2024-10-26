# KMS Policies

## Grant management
- Policy to allow manipulate Grants only for resources which have Grant support (condition)

```
{
    "Version": "2012-10-17",
    "Id": "Allow Grant",
    "Statement": [
        {
            "Sid": "GrantForServices",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::123456789012:user/testuser"
            },
            "Action": [
                "kms:CreateGrant",
                "kms:ListGrants",
                "kms:RevokeGrant"
            ],
            "Resource": ["*"],
            "Condition": {
                "Bool": {
                    "kms:GrantIsForAWSResource": true
                }
            }
        }
    ]
}
```

## Conditionas

### ViaService
- ensure that KMS will be used via service only

```
{
    "Version": "2012-10-17",
    "Id": "Allow Grant",
    "Statement": [
        {
            "Sid": "GrantForServices",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::123456789012:user/testuser"
            },
            "Action": [
                "kms:CreateGrant",
                "kms:ListGrants",
                "kms:RevokeGrant"
            ],
            "Resource": ["*"],
            "Condition": {
                "StringEquals": {
                    "kms:ViaService": [
                        "ec2.us-west-2.amazonaws.com",
                        "rds.us-west-2.amazonaws.com"
                    ]
                }
            }
        }
    ]
}
```

### CallerAccount
-  Ensure that specific user/role have access/deny to KMS
-  
```
{
    "Version": "2012-10-17",
    "Id": "Allow Grant",
    "Statement": [
        {
            "Sid": "GrantForServices",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::123456789012:user/testuser"
            },
            "Action": [
                "kms:CreateGrant",
                "kms:ListGrants",
                "kms:RevokeGrant"
            ],
            "Resource": ["*"],
            "Condition": {
                "StringEquals": {
                    "kms:CallerAccount": "123456789012"
                }
            }
        }
    ]
}
```