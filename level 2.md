# level 2
After installing the AWS CLI installed, I was finally able to get this one to work. You have to add User Name header to the csv file with your access keys, and add the name you want to use on the second line.

then, you have to run the command:
`aws configure import --csv file://[path to your csv file]`
*important note: you must include the 'file://' part or it won't work.*

Then it's simply a matter of copy and pasting the command on the first hint:
`aws s3 --profile [your profile name] ls s3://level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud`
This command lists the contents of this bucket, showing the secret file. You can then navigate to that file in the same fashion as the 1st level.

## Lesson learned
Similar to opening permissions to "Everyone", people accidentally open permissions to "Any Authenticated AWS User". They might mistakenly think this will only be users of their account, when in fact it means anyone that has an AWS account.

## Examples of this problem
Open permissions for authenticated AWS user on Shopify (link) 

## Avoiding the mistake
Only open permissions to specific AWS users. 