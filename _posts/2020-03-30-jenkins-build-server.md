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

I went through the typical [Jenkins setup](https://jenkins.io/doc/book/installing/#setup-wizard), which I will not outline here, since it's very well documented elsewhere. At this point I have successfully my own personal jenkins server. The next step is to create a build that automates delivery of this blog.

The first thing I needed was a jenkins user account in AWS IAM. This user account will be used to run builds and push to S3. After creating the account, I attached the existing policy `AmazonS3FullAccess` to the user.

With the user created, I needed to setup the aws cli on the jenkins machine. This was done by creating an Access Key/Secret key under Users > Jenkins > Security Credentials on IAM. Then, using an SSH session to our Jenkins server, run `aws configure` and go through the steps to get setup.

```
$ aws configure
AWS Access Key ID [None]: XXXXXXXXXXX
AWS Secret Access Key [None]: XXXXXXXXXXXXX
Default region name [None]: us-east-2
Default output format [None]: json
```





More to come soon.

Cheers!

-Nick

