---
layout: post
title: First Post - How'd I get here?
image: /img/hello_world.jpeg
---

Typically a first post might be used to help introduce the author, but I think I'll let you figure that out over the course of reading subsequent posts and forming your perception about who I am.

In this post, I'm just going to quickly run through how I brought this blog up.

Jekyll is a static website generator written in Ruby that is remarkably useful for bringing up simple and aesthetically pleasing websites. Since I just needed a *'blog style'* site for posting about my personal projects and interests, it's clearly a no-brainer. With a small amount of research, you'll find the jekyll is heavily used by people with similar use cases, so I won't talk about it any longer.

Instead of starting from scratch, I opted to use an existing theme to make my life a lot easier (I'm not a very competent front-end guy). There's a really popular theme called [beautiful-jekyll](https://github.com/daattali/beautiful-jekyll), and that's exactly what you're looking at now. If you go through the README, it's designed to be used with github pages, which would allow for a very quick and easy setup. I've tried this before and opted to go a different route, merely as a learning experience.

Let's breakdown how I put everything together..

```
​```
$ sudo gem install bundler jekyll
$ mdkir jekyll
$ cd jekyll/
$ #Fork: https://github.com/daattali/beautiful-jekyll.git via github. Personalize the repo.
$ #Then clone your forked repo
$ git clone https://github.com/nickshields/nickshields.com.git
$ cd nickshields.com
$ bundle install
$ bundle exec jekyll serve
​```
```

Running the above commands is all it takes to get a local instance of a simple blog website running. If everything went as planned, you should see the result via: http://localhost:4000



With everything setup locally, I made some changes to completely personalize the theme to my tastes. I removed a bunch of stuff and added all my social links. I tried to come up with some sort of fun tag-line, but all I came up with was **"CLOUD-ENGINEER x iOS Developer"** (My attempt at making software engineering more hype.¯\_(ツ)_/¯ ) 

I commited all my changes to github and the next step was getting some sort of production server running for my blog. As an amature user of AWS, I opted to go that route. AWS S3 storage buckets can serve static websites, which is exactly what I have here. The cool thing is that Amazon has a detailed guide outlining exactly what I did [here](https://docs.aws.amazon.com/AmazonS3/latest/dev/website-hosting-custom-domain-walkthrough.html), so I won't elaborate too much farther. Here's the TLDR setup:

1. Buy a domain via Route53
2. Create a bucket, enable Static web hosting.
3. Create a bucket for website redirect (ie: redirect people from **www**.nickshields.com to nickshields.com)
4. Push built jekyll website to main bucket. More on this later.
5. Ensure bucket is public and that attached policies allow object read access
6. Add alias records for domain (Mapping new domain to bucket DNS address)

If all the above steps go as planned, then your website should be live, as is the case with mine.

Now as far as pushing my blog to AWS, here's how I went about doing that:

```
$ brew install awscli
$ aws configure 
$ cd ~/jekyll/nickshields.com
$ bundle exec jekyll build
$ cd _site/
$ aws s3 sync . s3://nickshields.com/ --delete
```

The [awscli](https://aws.amazon.com/cli/) is really useful here. The above series of commands assume you have brew installed on your machine, which makes for a really quick install of cli. Visit the awscli link for the steps needed to configure it. You just need to generate an Access Key ID and Secret Access Key from IAM in the AWS management console. After installing the cli, I used jekyll to build the website, which roughly translates to compiling markdown documents into html/css/javascript. Then I copied all the contents of the built site to the bucket. The sync --delete command makes it really easy to ensure that the bucket's contents reflect exactly what jekyll built.



Anyways, that concludes how I brought this blog to life. In the coming posts, you'll hear about the rtmp streaming solution that I'm working on and about all the problems that I've come across and had to solve.



Cheers!

-Nick

