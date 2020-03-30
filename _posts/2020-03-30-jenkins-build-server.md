---
layout: post
title: Bringing up a Jenkins Server
image: /img/hello_world.jpeg
---

Since I've opted leverage AWS to host this blog instead of github pages, I needed a more intuitive and simple way to push changes. Prior to this post, here's how how I push updates:

```
$ cd ~/jekyll/nickshields.com
$ bundle exec jekyll build
$ cd _site/
$ aws s3 sync . s3://nickshields.com/ --delete
```

The typical workflow is as follows:

1. Checkout Repo.
2. Make Changes.
3. Build the updated site.
4. Sync it with the S3 bucket.
5. Push changes to Github.

While this workflow isn't too complex by any means, I felt the need to simply things. The target workflow that I have in mind and what we'll accomplish here is as follows:

1. Checkout Repo.
2. Make Changes.
3. Push Changes to Github.

I'll use my own Jenkins server to build and publish the updated site each time that changes are committed to the master branch of my repo.

So let's setup a Jenkins server..

AWS is really handy for spinning up virtual machines really fast (using EC2), so that's the route I've opted to take. Without going into too much detail, I simply spun up an EC2 instance using the current AWS Linux AMI 2018.03.0 (x64) image.

After launched, I SSH'd in and ensured that the AWS Security Group and Network ACL associated with this instance allowed for all traffic to enter/leave this machine (I'll lock it down later, but for now it's easier to debug problems when security is at a minimum).

Here's the setup I followed to get jenkins running:

```
$ sudo yum remove java-1.7.0-openjdk
$ sudo yum install java-1.8.0
$ sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
$ sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
$ sudo yum install jenkins -y
$ sudo service jenkins start
$ chkconfig jenkins on
```

The above steps remove ensure that we have a suitable version of Java 8 installed, which is necessary for Jenkins to run. We then download, install and set jenkins to automatically start running on startup.

On AWS, I went into Route53 and created a record set for my domain and mapped it to the IP of the EC2 instance. This allowed me to map jenkins.nickshields.com to the virtual machine where the jenkins instance is located.

At this point, the problem is that navigating to jenkins.nickshields.com brings upnothing, since jenkins is set to run on port 8080. While you can tell jenkins to run on port 80, the internet suggests that doing so is bad practice, since we need to change the permissions of the jenkins server and it's generally recommended to run jenkins as a user (not root).

The workaround to this problem is actually straightforward. I opted to use nginx to  forward all traffic from port 80 to 8080. Here's how I set that up:

```
$ sudo yum install nginx -y
$ sudo vim /etc/nginx/nginx.conf
```

Modify `nginx.conf`. You need to add the location object that is shown here: 

```
http {
    ...
		...
    server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  jenkins.nickshields.com;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
                proxy_set_header X-Forwarded-To $remote_addr;
                proxy_set_header Host $http_host;
                proxy_pass       "http://127.0.0.1:8080";
        }
   ...
```

It's also a good idea to set the server_name parameter to the domain that you created in Route53.

After that, restart nginx to see that the changes took effect:

```
$ sudo service nginx restart
```

Navigating to jenkins.nickshields.com now shows the jenkins setup splash screen, which is exactly what I wanted:

![Jenkins](https://jenkins.io/doc/book/resources/tutorials/setup-jenkins-01-unlock-jenkins-page.jpg){: .center-block :}

I went through the typical [Jenkins setup](https://jenkins.io/doc/book/installing/#setup-wizard), which I will not outline here, since it's very well documented elsewhere. I just used all the recommended settings, and installed all the recommended packages. At this point I have successfully my own personal jenkins server. The next step is to create a build that automates delivery of this blog.

While fairly straightforward in principle, the remainder of this post took me hours to complete, as I came across many issues trying to build. Most of the problems I came across were associated with user permissions (jenkins user trying to run things) as well as errors with getting ruby working on Amazon Linux.

Either way, this is how I eventually brought everything together..

I launched another EC2 instance to serve as a jenkins slave. After it came online, I had to do the following:

1. On the master node (jenkins machine):

```
$ ssh-keygen -t rsa
```

I needed to get a public key from the master node, so that it could communicate with the slave.

2. On the slave:

```
$ echo "ssh-rsa ..." >> .ssh/authorized_keys
```

This step was added the masters public key to the slave so that the master can freely ssh into it. 

3. Install all the required tools on the slave:

```
$ sudo yum remove java-1.7.0-openjdk
$ sudo yum install java-1.8.0
$ sudo yum install git
$ gpg2 --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB
$ \curl -sSL https://get.rvm.io | bash -s stable
$ source /home/ec2-user/.rvm/scripts/rvm
$ rvm install 2.4
$ gem install bundler jekyll
```

The above steps install java 1.8. I don't think jenkins requires 1.8, but installing it played very nicely when I went to enable the slave via the jenkins ui. I also installed rvm since it seems to be the most trivial for Ruby dependency management. After installing rvm, I installed Ruby 2.4, as well as the bundler and jekyll gems, which are needed for building my blog.

The last thing I needed on the slave was a jenkins user account in AWS IAM. This user account is used to run builds and push to S3. After creating the account, I attached the existing policy `AmazonS3FullAccess` to the user. With the user account setup, I used `aws configure` to configure the slave with access to AWS S3 buckets.

```
$ aws configure
AWS Access Key ID [None]: XXXXXXXXXXX
AWS Secret Access Key [None]: XXXXXXXXXXXXX
Default region name [None]: us-east-2
Default output format [None]: json
```

Now that the slave was configured, I needed to add it to jenkins so I could use it. From Jenkins, I went to Jenkins > Nodes > New Node and set the following:



For the Credentials, I added new Jenkins credentials with the following info:



At the completion of these steps, I was able to successfully launch the slave (There was actually a ton of debugging required at this point, but through trial and error, this post only demonstrates the final product... lucky you)

Note: I disabled the master node in jenkins so that builds wouldn't execute on the jenkins host. The actual reason for this is because I was having troubles getting builds to run on this node due to permission errors. Running builds on the slave worked as intended..

Creating the build is fairly straightforward, now that I have all the kinks worked out. In Jenkins, create a freestyle job. The below pics outline the build:



The source code management section is where you specify the location of the git repo. I used */master since I want to pull from this branch when changes get pushed.



Under build triggers, I enabled GitHub hook trigger for GITScm polling, since I want to setup a webhook from GitHub to notify jenkins of new changes.



Under the build steps, I needed to use `#!/bin/bash -l` because rvm needs a login shell, which is not how jenkins jobs execute (they use) Under post actions, I added a step to delete the workspace after the build completes, since we don not need any of it after it gets synced onto the S3 bucket.

The last thing I had to do was add a webhook under my repo's settings:



Once I saved this, I commited some changes to my repo, and it triggered the jenkins build. The build succesfully completed and my changes were reflected on my production website.



While this was a long-winded post, it serves as a good starting point for someone who might want to tinker around with build systems. The overall project here is fairly straightforward and the end result is relatively satisfying. I can happily report that making changes to my blog is a lot easier. The next steps will be to leverage a tool like docker to automate these entire deployments. More on that soon.



Cheers!

-Nick

