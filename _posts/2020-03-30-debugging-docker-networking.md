---
layout: post
title: Debugging Docker Networking
subtitle: Getting containers to talk
image: /img/hello_world.jpeg
---

Relatively common backstory..

I recently ran into an issue where I used docker-compose to bring up a bunch of services locally. The backstory here is fairly straightforward and common amongst cloud development teams. We have a situation where we have teams of developers working on different services for a large cloud platform. The services that they are working on need to be able to communicate amongst one another. If a team wanted to test how their new features communicated with other services, they would be required to manually setup a local instance of those services before they could test. 

Setting up and maintaining other teams services is usually not that straightforward. Managing the front-end, backend, databases etc.. is a pain. In an attempt to make life easier for everyone, I used docker to create sanboxed environments for each team's service.

In this post, I'll be going over how to use docker compose to bring up various docker containers that are able to 'speak' to each other.