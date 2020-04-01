---
layout: post
title: Deploy faster with Docker
subtitle: I only like doing things once
image: /img/2020-03-30-deploy-fast-with-docker/docker.png
---

This post will explore how docker can be used to make deployments a lot easier. In a previous post, I demonstrated how I built a jenkins server and a slave. While I'm happy with the outcome, I realized that being able to reproduce that setup could take a lot of time since there was a lot of configuration required. The hope of this post is to build a docker image that has everything it needs to run a Jenkins server. After that, I'll attempt to connect it to a slave and run some form of build. Let's get started.

