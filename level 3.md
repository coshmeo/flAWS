# level 3
figured out how to set my profile as default:
 
`export AWS_PROFILE=[your profile name]`

This allows you to run all the aws commands without having to specify the profile everytime.

Downloaded the bucket using the command in the first hint

`aws s3 sync s3://level3-9afd3927f195e10225021a578e6f78df.flaws.cloud/ . --no-sign-request`

Hint 2 says I'm supposed to see an AWS key in the git logs, but I'm not seeing anything...

After I synced my github, I think the `.git` file included in the s3 bucket pulled the keys from the publisher's flAWS repository. After messing around for while, I removed all the flAWS files from my own, but I think the idea was you download all the files from the bucket and then request an update from the associated repository.

## Lesson learned
People often leak AWS keys and then try to cover up their mistakes without revoking the keys. You should always revoke any AWS keys (or any secrets) that could have been leaked or were misplaced. Roll your secrets early and often.

## Examples of this problem
Instagram's Million Dollar Bug: In this must read post, a bug bounty researcher uncovered a series of flaws, including finding an S3 bucket that had .tar.gz archives of various revisions of files. One of these archives contained AWS creds that then allowed the researcher to access all S3 buckets of Instagram. For more discussion of how some of the problems discovered could have been avoided, see the post "Instagram's Million Dollar Bug": Case study for defense 

Another interesting issue this level has exhibited, although not that worrisome, is that you can't restrict the ability to list only certain buckets in AWS, so if you want to give an employee the ability to list some buckets in an account, they will be able to list them all. The key you used to discover this bucket can see all the buckets in the account. You can't see what is in the buckets, but you'll know they exist. Similarly, be aware that buckets use a global namespace meaning that bucket names must be unique across all customers, so if you create a bucket named `merger_with_company_Y` or something that is supposed to be secret, it's technically possible for someone to discover that bucket exists.

## Avoiding this mistake
Always roll your secrets if you suspect they were compromised or made public or stored or shared incorrectly. 
Roll early, roll often. 
Rolling secrets means that you revoke the keys (ie. delete them from the AWS account) and generate new ones. 