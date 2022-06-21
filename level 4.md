# level 4

For the next level, you need to get access to the web page running on an EC2 at 4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud

It'll be useful to know that a snapshot was made of that EC2 shortly after nginx was setup on it. 

**hint 1:**
*You can snapshot the disk volume of an EC2 as a backup. In this case, the snapshot was made public, but you'll need to find it.*

To do this, first we need the account ID, which we can get using the AWS key from the previous level:

`aws --profile flaws sts get-caller-identity`

Using that command also tells you the name of the account, which in this case is named "backup". The backups this account makes are snapshots of EC2s. Next, discover the snapshot:

`aws --profile flaws  ec2 describe-snapshots --owner-id 975426262029`

We specify the owner-id just to filter the output. For fun, run that command without the owner-id and notice all the snapshots that are publicy readable. By default snapshots are private, and you can transfer them between accounts securely by specifiying the account ID of the other account, but a number of people just make them public and forget about them it seems. 

Now that you know the snapshot ID, you're going to want to mount it. You'll need to do this in your own AWS account, which you can get for free.

First, create a volume using the snapshot:

`aws --profile YOUR_ACCOUNT ec2 create-volume --availability-zone us-west-2a --region us-west-2  --snapshot-id  snap-0b49342abd1bdcb89`

Now in the AWS console in your browser you can create an EC2 in the `us-west-2` region, and in the storage options choose the volume you just created.

SSH in with something like:

`ssh -i YOUR_KEY.pem  username@ec2address`

We'll need to mount this extra volume by running:

    lsblk

    # Returns:
    #  NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
    #  xvda    202:0    0   8G  0 disk
    #  └─xvda1 202:1    0   8G  0 part /
    #  xvdb    202:16   0   8G  0 disk
    #  └─xvdb1 202:17   0   8G  0 part

    sudo file -s /dev/xvdb1

    # Returns:
    #  /dev/xvdb1: Linux rev 1.0 ext4 filesystem data, UUID=5a2075d0-d095-4511-bef9-802fd8a7610e, volume name "cloudimg-rootfs" (extents) (large files) (huge files)

    # Next we mount it

    sudo mount /dev/xvdb1 /mnt

Now you can dig around in that snapshot navigating to `/mnt/home/ubuntu`

    ls
    meta-data  setupNginx.sh

`cat` the setup file and you will find the username and password.

## Lesson learned
AWS allows you to make snapshots of EC2's and databases (RDS). The main purpose for that is to make backups, but people sometimes use snapshots to get access back to their own EC2's when they forget the passwords. This also allows attackers to get access to things. Snapshots are normally restricted to your own account, so a possible attack would be an attacker getting access to an AWS key that allows them to start/stop and do other things with EC2's and then uses that to snapshot an EC2 and spin up an EC2 with that volume in your environment to get access to it. Like all backups, you need to be cautious about protecting them. 