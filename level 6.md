# level 6
using the credentials given, create a aws profile called 'level6'
`vi ~/.aws/credentials`

This account has a SecurityAudit policy attached to it. The SecurityAudit group can get a high level overview of the resources in an AWS account, but it's also useful for looking at IAM policies. First, find out who you are 

    aws --profile level6 iam get-user
    {
        "User": {
            "Path": "/",
            "UserName": "Level6",
            "UserId": "AIDAIRMDOSCWGLCDWOG6A",
            "Arn": "arn:aws:iam::975426262029:user/Level6",
            "CreateDate": "2017-02-26T23:11:16+00:00"
        }
    }

Now that you know your username is Level6 you can find out what policies are attached to it:

aws --profile level6 iam list-attached-user-policies --user-name Level6
    {
        "AttachedPolicies": [
            {
                "PolicyName": "list_apigateways",
                "PolicyArn": "arn:aws:iam::975426262029:policy/list_apigateways"
            },
            {
                "PolicyName": "MySecurityAudit",
                "PolicyArn": "arn:aws:iam::975426262029:policy/MySecurityAudit"
            },
            {
                "PolicyName": "AWSCompromisedKeyQuarantine",
                "PolicyArn": "arn:aws:iam::aws:policy/AWSCompromisedKeyQuarantine"
            }
        ]
    }

Now you know that you have two policies attached:

    "SecurityAudit": This is an AWS policies you can look up either in your console or here
    "list_apigateways": This is a custom one I made. I bet it'll be useful for this challenge, 
    so we should try to figure out what it is. 

Once you know the ARN for the policy you can get it's version id

    aws --profile level6 iam get-policy  --policy-arn arn:aws:iam::975426262029:policy/list_apigateways
    {
        "Policy": {
            "PolicyName": "list_apigateways",
            "PolicyId": "ANPAIRLWTQMGKCSPGTAIO",
            "Arn": "arn:aws:iam::975426262029:policy/list_apigateways",
            "Path": "/",
            "DefaultVersionId": "v4",
            "AttachmentCount": 1,
            "PermissionsBoundaryUsageCount": 0,
            "IsAttachable": true,
            "Description": "List apigateways",
            "CreateDate": "2017-02-20T01:45:17+00:00",
            "UpdateDate": "2017-02-20T01:48:17+00:00",
            "Tags": []
        }
    }

Now that you have the ARN and the version id, you can see what the actual policy is

    aws --profile level6 iam get-policy-version  --policy-arn arn:aws:iam::975426262029:policy/list_apigateways --version-id v4
    {
        "PolicyVersion": {
            "Document": {
                "Version": "2012-10-17",
                "Statement": [
                    {
                        "Action": [
                            "apigateway:GET"
                        ],
                        "Effect": "Allow",
                        "Resource": "arn:aws:apigateway:us-west-2::/restapis/*"
                    }
                ]
            },
            "VersionId": "v4",
            "IsDefaultVersion": true,
            "CreateDate": "2017-02-20T01:48:17+00:00"
        }
    }

This tells us using this policy we can call `apigateway:GET` on `arn:aws:apigateway:us-west-2::/restapis/*`

Using this should lead you in the direction of where the hidden resource is. 

The API gateway in this case is used to call a lambda function, but you need to figure out how to invoke it.

The SecurityAudit policy lets you see some things about lambdas: 

    aws --region us-west-2 --profile level6 lambda list-functions
    {
        "Functions": [
            {
                "FunctionName": "Level6",
                "FunctionArn": "arn:aws:lambda:us-west-2:975426262029:function:Level6",
                "Runtime": "python2.7",
                "Role": "arn:aws:iam::975426262029:role/service-role/Level6",
                "Handler": "lambda_function.lambda_handler",
                "CodeSize": 282,
                "Description": "A starter AWS Lambda function.",
                "Timeout": 3,
                "MemorySize": 128,
                "LastModified": "2017-02-27T00:24:36.054+0000",
                "CodeSha256": "2iEjBytFbH91PXEMO5R/B9DqOgZ7OG/lqoBNZh5JyFw=",
                "Version": "$LATEST",
                "TracingConfig": {
                    "Mode": "PassThrough"
                },
                "RevisionId": "98033dfd-defa-41a8-b820-1f20add9c77b",
                "PackageType": "Zip",
                "Architectures": [
                    "x86_64"
                ]
            }
        ]
    }

That tells you there is a function named `Level6`, and the `SecurityAudit` also lets you run

    aws --region us-west-2 --profile level6 lambda get-policy --function-name Level6

    {
        "Policy": "{\"Version\":\"2012-10-17\",\"Id\":\"default\",\"Statement\":[{\"Sid\":\"904610a93f593b76ad66ed6ed82c0a8b\",\"Effect\":\"Allow\",\"Principal\":{\"Service\":\"apigateway.amazonaws.com\"},\"Action\":\"lambda:InvokeFunction\",\"Resource\":\"arn:aws:lambda:us-west-2:975426262029:function:Level6\",\"Condition\":{\"ArnLike\":{\"AWS:SourceArn\":\"arn:aws:execute-api:us-west-2:975426262029:s33ppypa75/*/GET/level6\"}}}]}",
        "RevisionId": "98033dfd-defa-41a8-b820-1f20add9c77b"
    }

This tells you about the ability to execute `arn:aws:execute-api:us-west-2:975426262029:s33ppypa75/*/GET/level6\` 
That `s33ppypa75` is a `rest-api-id`, which you can then use with that other attached policy: 

    aws --profile level6 --region us-west-2 apigateway get-stages --rest-api-id "s33ppypa75"
    {
        "item": [
            {
                "deploymentId": "8gppiv",
                "stageName": "Prod",
                "cacheClusterEnabled": false,
                "cacheClusterStatus": "NOT_AVAILABLE",
                "methodSettings": {},
                "tracingEnabled": false,
                "createdDate": "2017-02-26T16:26:08-08:00",
                "lastUpdatedDate": "2017-02-26T16:26:08-08:00"
            }
        ]
    }

That tells you the stage name is `Prod`. Lambda functions are called using that rest-api-id, stage name, region, and resource as 
`https://s33ppypa75.execute-api.us-west-2.amazonaws.com/Prod/level6`

and here we have a link to the end of flAWS

It is common to give people and entities read-only permissions such as the SecurityAudit policy. The ability to read your own and other's IAM policies can really help an attacker figure out what exists in your environment and look for weaknesses and mistakes.

## Avoiding this mistake
Don't hand out any permissions liberally, even permissions that only let you read meta-data or know what your permissions are. 