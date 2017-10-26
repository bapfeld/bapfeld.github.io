---
layout: post
title: "More Things I've Learned Using AWS"
Categories: [Code]
tags: [R, AWS]
---
Previously, I wrote a [post](/Code/2017/04/18/cloud_computing/) about getting started using AWS services for social science research. After working a little bit more with AWS, I've learned a few more things that I want to share here.

# Free Limits
One of the great things about getting started with AWS is that there's a free tier (at least for the first year). With it you're given 750 hours/month on one of their low-powered "t2-micro" machines. Great. Well note that this is 750 hours *total* and not 750 hours *per* t2-micro instance that you have running. They don't cost a lot, so if you leave, say, 5 of them running constantly for a month, it's not going to break the bank, but it's still an unpleasant surprise.

# Moving Physical Locations
If you move physical locations (or do anything else that will result in you getting a different IP address) you will need to update your security group. This is really easy to do and you don't need to stop any of your instances. On the IAM EC2 console, choose security groups. Select your group, then Actions, then edit inbound rules. Now just change the entry with the old IP address (the options should read, from left to right, SSH, TCP, 22, Custom, someIPaddress). Hit the custom button and select "My IP". Save and get back to work.

# Moving Between Computers
If you move computers you may need to update the permissions on your `.pem` key file. Everything in my project is kept on GitHub and when I moved to a new machine and tried to connect for the first time, I received a message saying "Permissions 0664 for 'my_pem_file.pem' are too open." Quick fix: `chmod 400 my_pem_file.pem`.

# Walking Away and Letting it Run
You'll pretty quickly find that if you close the terminal you're using to connect to AWS (or your Mac goes to sleep because it's annoying like that or your battery runs out because your other laptop is annoying like *that*), then you'll lose anything you have open there. The instance keeps running and any files you've saved will stay saved, but if you were running a process, that will terminate. Which is annoying. There are at least three solutions to this problem, each with different benefits and drawbacks. There are many pages written about this already (see [here](https://askubuntu.com/questions/8653/how-to-keep-processes-running-after-ending-ssh-session) for a really useful discussion, though I don't personally use the top recommendation), so I'll keep this short: If you remember to do this ahead of time, use either `screen` or `nohup`. The former is good if you're doing something interactive or want to come back and monitor progress (e.g. if there's a progress bar from your jags model). Apparently `screen` is really powerful and you should do a little reading about the basics so you know how to navigate it. The latter option is good if all you want to do is let the process run. If you forgot to do one of these before starting or your process is taking longer than expected, then `disown` is the way to go. Again, do some reading on how to implement it.

# Other types of instances
Even in the free tier of services, you arne't bound do Amazon's Linux AMI. If you choose a different instance type logon details change. If you go with an ubuntu instance, for example, you'll log on as `ubuntu@ec2-public-dns` instead of `ec2-user`. See [here](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/TroubleshootingInstancesConnecting.html#TroubleshootingInstancesConnectingMindTerm) for a complete list of logons by instance type.
