---
layout: post
title: "Running Bayes in AWS"
categories: [Code]
tags: [R, AWS]
---
This is the third post in a series on using AWS for your social stats needs. [Here's](/Code/2017/04/18/cloud_computing/) the original post on getting started and a [follow-up](/2017/07/10/aws_lessons/) post with some additional lessons learned.

Okay, so you've got R up and running on AWS and you've figured out how to get your code to run and walk away. Great. Now for an example where you might actually need to do this: running a Bayesian analysis.

This usage differs a little from what I've previously suggested is the use for AWS. Here, instead of needing a huge increase in power, what you might be looking for is a machine you can start running and ignore, but where 128 cores in a parallel environment isn't going to save you any time.^[1]

Here are the steps to follow to use JAGS on your EC2 instance. These steps come from [this](https://sumtxt.wordpress.com/2011/07/08/how-to-setup-r-and-jags-on-amazon-ec2/) very helpful post, but it's a little out of date and there are a few additional steps you'll need to take.

# Step 0: Get your EC2 instance running and install R
I'm going to assume that you've already read my previous post on this and that you're set here.

# Step 1: Install JAGS
To run JAGS in R, you'll need to have JAGS installed. I'm going to assume here that you have super user powers (i.e. you ran `sudo su` and didn't exit from that) to run these commands

  1. Install the tools you'll need to compile everything:
      * `yum install gcc`
      * `yum install gcc-c++`
      * `yum install gcc-gfortran`
      * `yum install readline-devel`
      * `yum install make`
      Make should already be installed, but if for some reason it's not, you definitely need to have that.
  2. Download JAGS
      * `wget https://sourceforge.net/projects/mcmc-jags/files/JAGS/4.x/Source/JAGS-4.2.0.tar.gz`
      * Check https://sourceforge.net/projects/mcmc-jags/files/JAGS/4.x/Source/ to make sure that you're getting the most recent version and if not, change the `wget` command as appropriate
  3. Install JAGS
      * `tar xf JAGS-4.2.0.tar.gz`
      * `cd JAGS-4.2.0`
      * `./configure --libdir=/usr/local/lib64`
      * `make`
      * `make install`
      * `cd ..`
  4. Link the installation so R can find it
      * `export PKG_CONFIG_PATH=/usr/local/lib64/pkgconfig`
      * You can double check that this is, in fact where the jags.pc file is located by running `ls /usr/local/lib64/pkgconfig`. If it's not in there, you'll have to figure out where it is and adjust accordingly.

# Step 2: Install rjags
Just launch R and run `install.packages("rjags")`.  

# Monitoring Progress
Assuming that everything has gone well, you start your code and walk away. But how will you know when it's finished? Well there are two ways. The first is to take a look at the EC2 console. On the page that displays your instances, you can select "monitoring" toward the bottom of the page and take a look at how much computing power you're using. It's delayed by a few minutes, but it will probably show a spike when you start the code, then a flat line. When that line falls again, you're all done.

The better option is to use `screen` when you start your process instead of `nohup`. Sure `nohup` will work just fine, but you won't be able to see any progress. If you opt for `screen` and then start R and run your code that way (instead of starting `screen` and then using `R CMD BATCH` because obviously that won't solve the monitoring issue), you can always come back later, reattach to the screen and take a look at the nice progress bar that jags provides.

[1]: Since every iteration of a sampler depends on the previous iteration, running in parallel doesn't make a lot of sense. However, if you are running multiple chains, a parallel environment will still save you time.
