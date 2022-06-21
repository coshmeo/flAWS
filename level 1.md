# level 1
you can view the entire contents of the flaws.cloud bucket here: http://flaws.cloud.s3.amazonaws.com/

from there, you can see that there is a secret file. You can naviate to it by navigating to: http://flaws.cloud/[secret file name]

## Lesson learned
On AWS you can set up S3 buckets with all sorts of permissions and functionality including using them to host static files. A number of people accidentally open them up with permissions that are too loose. Just like how you shouldn't allow directory listings of web servers, you shouldn't allow bucket listings.

## Avoiding the mistake
By default, S3 buckets are private and secure when they are created. To allow it to be accessed as a web page, I had turn on "Static Website Hosting" and changed the bucket policy to allow everyone "s3:GetObject" privileges, which is fine if you plan to publicly host the bucket as a web page. But then to introduce the flaw, I changed the permissions to add "Everyone" to have "List" permissions.