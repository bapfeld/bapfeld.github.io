---
layout: post
title: "Political Science in the Cloud"
categories: [Code]
tags: [R, AWS]
---
No, I'm not talking about [Aristophanes](http://classics.mit.edu/Aristophanes/clouds.html). I'm talking about using cloud computing to dramatically increase computing power and do things my little laptop can't handle.

As an extension of some research I've been doing with John Gerring, we've found ourselves in need of increased computing power in order to dramatically reduce the time that some of our code takes to run. We explored the possibility of using some of UT's supercomputers, but decided that the simpler solution would be to pay to use Amazon Web Services (AWS).

I know that there are already tutorials out there on how to do this (I learned from [this](https://aws.amazon.com/blogs/big-data/running-r-on-aws/) post), but there are some additional steps you need to take that they really don't discuss in detail, so I'm going to outline this from the start in painfully simple steps.

This post is going to be written with a user in mind who knows just enough about typing things into the terminal to be dangerous and has a firm grasp of R, but who maybe doesn't have any experience with servers or distributed computing.

# Why do this?
The startup costs of this in terms of time are not zero, so this is really only useful to you if you:
    1. Have a computing intense issue that your computer cannot handle
    2. The total amount of time you will have to wait on your computer to run the code is at least several hours (either all at once or accumulated time over multiple runs)
    3. Your code is written or could be modified to run in parallel in a distributed environment

# How much does it cost?
That's really tough to answer because it depends on how much power you want to buy, how long you're going to use it, and where the servers you're using are located. Hourly rates on the AWS servers run from free for the least powerful machines in the first year of your use to a few pennies for the next couple of steps up to somewhere around $1.50 for the most powerful computing machines.

# How do I do it?
Follow these steps!

## One time only steps
### Getting AWS Set Up
First, some steps that you'll only have to do one time. These directions come from [this](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/get-set-up-for-amazon-ec2.html) very helpful set of directions. Feel free to follow those to the letter.

1. Sign up for [AWS](https://aws.amazon.com). You can use your Amazon credentials, but you'll need to provide or confirm credit card information.
2. Create an IAM user
    + Sign in to the [IAM console](https://console.aws.amazon.com/iam/home#/home)
    + Navigate to Users --> Add user
    + Create a username (it can be something like "admin")
    + Give yourself "AWS Management Console access"
    + Select Custom password and set that
    + Uncheck Require password reset
    + Go to the next page
    + Choose Create Group
    + Type a name for your group
    + Choose Filter --> Job function
    + Select the checkbox for AdministratorAccess
    + Click on Create group
    + Click on Next: Review
3. Sign in to the console using your newly created IAM user
    + If you haven't left the previous page, there will be a link with the sign on address
    + If not, it's in the format: `https://your_aws_account_id.signin.aws.amazon.com/console/`
        - If you forgot your account number, you can find that by signing into your full account and looking up your account details
4. Create a key pair
    + From the dashboard, select EC2
        - You may need to expand the All services section
    + Select the region from the region drop down menu on the top right
        - You can use any region available, but your key pair is tied to that region, so that's where you'll be running your applications
    + Choose Key pairs from the Network & Security panel on the left
    + Click create key pair and then give your key a name
        - Amazon recommends using something like `your_IAM_name-key-pair-region`
    + Create the key
        - This will trigger a file download and a new entry on the key pair page. Put this file someplace safe. Note: the file should end with the extension .pem, but it's possible that your browser will tack on a .txt to that. Remove the .txt.
        - This is your private key and this is the ONLY time you can download this file, so don't trash it.
        - As Amazon recommends, I went ahead and edited the permissions: `chmod 400 my-private-key.pem`
5. Create a VPC
        - Maybe. I didn't have to do this. On your EC2 dashboard, look to the right under Account Attributes. If it *only* lists VPC and then below that there it says Default VPC and below that some letters and numbers, then you don't need to create a VPC.
6. Create a security group
    + Note, you need a security group for each region you want to launch an instance in, just as you would need a different key pair. This probably is only important for those reading this if you plan on traveling somewhere and want to work in the cloud from your destination.
    + Back in the EC2 dashboard, select Security Groups
    + Click on Create security group, then give it a name
        - Amazon recommends using something like `your_IAM_name-SG-region`
        - You'll also need to add a description
    + Select your VPC from the vpc list (if you didn't have to create one, then the default will already be selected)
    + Create three rules:
        - Type: HTTP and Source: Anywhere
        - Type: HTTPS and Source: Anywhere
        - Type: SSH and Source: My IP
    + Create your group

### Preparing to Connect
There are a couple of ways to connect to your instance once it's running. The browser method wouldn't work for me, so my explanation will stick to the CLI method. [This](http://docs.aws.amazon.com/cli/latest/userguide/installing.html) article explains how to install these if you're not using mac and/or you don't have `pip` installed. You should also check [this page](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html) out for more detailed information about getting the CLI ready to run. I'm just going to do a bare-bones intro here.

1. Run `pip install --upgrade --user awscli` (or `pip3` as appropriate)
2. Since this command installs the AWS ClI only for the local user, you'll have to make sure that you add it to your path (in my case this means adding export `export PATH=$PATH:~/Library/Python/3.6/bin` to my .bash_profile. See [here](http://docs.aws.amazon.com/cli/latest/userguide/installing.html) for system specific directions on making this run.)
3. Run `aws configure` and enter the values as appropriate.
    + Your Access key and Secret access key can be generated and downloaded (once only) from the IAM console. (But you can always add more, just go to IAM --> Users --> Security Credentials)
    + You need to set a default region. Mine is "us-east-2"
    + Default output options are "text", "json", or "table". I used "text", but you can always change this when you call a specific command (which is useful since `aws ec2 describe-instances` is pretty much unreadable in text format; solve this by adding `--output table`)


## What you do every time
Okay, now the fun stuff. These steps are taken from two pages from Amazon ([here](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html) and [here](https://aws.amazon.com/blogs/big-data/running-r-on-aws/)), so check those out for more details.

1. On the EC2 dashboard, launch an instance and choose Amazon Linux AMI
2. On the next screen, you'll have to choose the type of instance you want to run (i.e. how much computing power you're getting). Since you can change this later, you may want to consider running the t2.micro to set everything up (since that's included in the free tier) and then stopping and restarting with more power later when you really need it.
3. Gather the necessary information.
    + Instance ID and Public DNS. I find the easiest way to grab this is with
        - `aws ec2 describe-instances --output table > instances.txt; open instances.txt -a Atom.app`
    + You'll also need to make sure you know where your key pair secret key .pem file is
4. Connect to your instance
    + `ssh -i /Path/to/your/priave/key.pem ec2-user@public_dns_name`
        - Change the path to your private key and the public dns name
        - leave the "ec2-user" as is, assuming you're using ec2
5. You're in!  The first time you start up, you may be prompted with some approvals and you may need to update some security packages.
6. Install R
    + First, give yourself the proper privileges:
        - `sudo su`
        - If you log out of your instance and then back in again, you will lose your root privileges, but should be able to work without any problems. If not, just run this again.
        - If you don't need this, then you should probably leave root to avoid doing anything too stupid. `exit` or control+d (on Mac) will bring you back to your user level; but keep these powers until you've completed your setup
    + Now install R
        - `yum install -y R` (the `-y` flag assumes yes in the install)
7. Run R
    + `R`

At this point, you're up and running and can do all of the normal R stuff. You'll probably need to install some packages before you do you analysis. I would also assume you'll want to upload your code and data and then run it with a batch command instead of entering it line by line. But I guess that depends on your scenario. But there's one more **super important** step:

8. Shut down your instance! When you've finished working, be sure to shut down your instance. Be warned that this will erase all of your data, so keep reading for how to get that out. If you don't, you'll continue to run infinitely. Even if you're only using the free stuff now, eventually you will incur costs. Plus this isn't nice to the environment or others who use time on these servers.
    + You'll probably want to close your ssh connection. You can do that simply with `exit`.
    + On the EC2 dashboard, choose your instance and then Actions --> Instance state --> Terminate.

## Other considerations
### Moving data in and out
There are a variety of ways to deal with getting data in and out of the system. One option is to use Amazon's S3 cloud storage, but that requires more setup. It may be worth the time, though, if you are a) collaborating on your work and everybody needs access to the data easily or b) you have fairly large files and plan to return to the data on multiple occasions.

But if you're just doing a one-off thing, then one option is to use SCP (secure copy). If you got ssh to work, then scp should be just as easy. The command is nearly identical to the ssh command: `scp -i /path/to/your/private/key.pem path/to/file/to/copy.file ec2-user@public_dns_name:~`. In addition to the file name, notice that there's a colon at the end of the dns name and a tilde. The colon is where the path to the new file starts. In this case we're just putting it in the home directory.

To transfer data out using the same method, just flip the order of the files.

If you stop your instance, AWS will warn you that any data on the "ephemeral" storage will be lost. I'm not entirely sure what this means, but I've safely stopped, started, and changed an instance without losing the files I uploaded.

### Potential Issues
You may run into some problems with your packages in R. I could never get dplyr to install, for example. Also, if you use RCurl to move data in from S3, be aware that you'll need to have the correct system dependencies installed (i.e. run `yum install -y libcurl-dev.x86_64`) when you're setting up your system.

### Starting slow and speeding up
Some of the initial startup takes time. Not a huge amount, but install R and required packages can take a few minutes. If you're planning on really ramping up your computing power but don't want to pay for the setup time, you're in luck! Start a t2.micro instance, get everything installed, then stop the instance. You can then change the instance type to whatever greater power you need, and restart it.[^1] Note that the public DNS name will change when you do this, so if you try to reconnect and you get a timeout message, this is probably why.

### Other lessons
Since I wrote this post originally, I've learned a few lessons. [Here's](/Code/2017/07/10/aws_lessons/) a brief update with what I've learned.

# Computing Power
Okay, so just how much computing power do we have on hand? Well the short answer is that it depends.  It depends on that depends on the sort of instance you're using and it depends on your code. The free tier gives you access to the t2.micro systems. My 2013 MBA has a mere 1.3ghz two-core processor, so it may not be the best benchmark for everybody, but (obviously) for my purposes I can see how much I stand to gain. As a basic test, I ran `system.time(rnorm(100000000))` on my machine and the EC2 instance. My computer spent an elapsed 12.7 seconds to run, with the system using .44 seconds of that.  The t2.micro instance took 9.7 second total, with .24 seconds going to the system.

If this is any indication of overall performance, it looks like even the free tier is offering performance that would increase speed by 25-50% depending on the exact code I'm running. But of course it's the really big stuff that my little computer *really* can't handle that I care about.

Of course not all code is created equal. If you're going to move beyond the basic tier, then you need to make sure your code is written to take advantage of the cores you're paying for. This means having things run correctly in parallel. Writing code to run in parallel is way beyond the scope of this post.

When I have an opportunity to test this on something that my computer can't handle but is suited to this setup, I'll write a follow-up post and update this one with a link to give an idea of just much time it saves.


[^1]: If you have a new AWS account, you will have some limits to the instances you can launch. On your EC2 Dashboard, select Limits to see your current limits. If you need more computing power, you'll have to request a limit increase. Click the "Request limit increase" button on the right and follow the directions. It'll probably take a few days for the request to go through, so plan ahead here.
