---
layout: post
title: FreshTomato - QoS via Bandwidth Limiting
subtitle: Throttling an AppleTV to improve network concurrency
image: /img/2020-04-01-ios-camera/iphone_camera.jpeg
---

**The backstory...** (Skip down to the problem if you really don't care)

My parents live in what many would refer to as the *boonies*. Others would refer to it as rural Niagara. Regardless, living on a farm has its perks which are fairly obvious: lots of privacy, peace and quiet, and just a ton of space to whatever you might want to do. But it also has a few caveats. I will admit that it does get a bit lonely out there, but it also lacks some of the common things that city folks take for granted, most notably, internet.

While our nearby smalltown has cable/fibre infrastructure and in some parts, dsl; you won't find any of that where we are. All we have is a lousy phone line. As a young lad, we used dial-up to get connected, but long gone are the days where a 56K modem is sufficient for being any bit productive.

So what options do we have? 

- **Cellular:** While cellullar technology has rapidly advanced over the years, it still isn't ideal for most people as primary source of internet since it comes at a steep cost (likely due to the monopoly formed by the few cellular companies that can demand any price).
- **Dial-Up:** A 56kbit/s connection would mean that a 1GB movie file would take around 40 hours to download..
- **Satellite**: Comes at a steep cost and has serious latency issues due to the time it takes to reach the satellite.
- **Wireless**: This is what we've turned to and what many others do as well. Let's talk about it.

Wireless internet is basically this:



You find a provider who has infrastructure that is close enough to your home. You also have to ensure a clear line of sight to the provider's infrastructure. You put an antenna as high as you need to in order to acheive that clear line of sight (for us we had to go up 83 feet). And that's it! Now you can have high speed internet at your rural home. While this is the most common option, it should be noted that it still isn't that cheap, especially if you don't have a clear line of sight. For our situation, we had to invest in a large tower, which put us back a few stacks, but in the end, was necessary for reliable service.

This option still sucks from a price standpoint ($100/Month for 25mbps down/5mbps up) but it's by far the best you can get given the options presented above..

A 25mbps connection is sufficient for everything my parents need. But there is one exception to this..

**The Problem**

While watching movies on their 4K AppleTV, they noticed that they were unable to do anything else on the internet while the movie was playing. In other words, the AppleTV appeared to be consuming all of the available bandwidth. I made an effort to solve this problem and I did come up with a valid solution.

**The Solution: Bandwidth Limiting**

My parents have a 4K Television connected to the 4K Apple TV. As with all streaming devices, the goal is to provide the end user with the best quality stream, given their network circumstances. For more on this, see [adaptive bitrate streaming](https://en.wikipedia.org/wiki/Adaptive_bitrate_streaming). The Apple TV has the following bandwidth rerecommendations:

- 15mbps for 4k
- 8mbps for 1080p
- 6mpbs for 720p streaming

We pay for a connection speed of up to 25mpbs, which is another way of ISP's telling you that you probably won't achieve that metric often. Running a quick speed test revealed that, at the time of writing, we have a bandwidth of around 18mpbs. 

While 4k obviously looks the best, I decided to establish if it was possible to throttle their AppleTV to 10mbps. That way, the streams would be limited to 1080p and then there would be an additional 8-15mbps available for other users to use the internet with.

My parents modem is connected to a Netgear R7000 router, which was a fairly popular router that I believe is still on the shelf today. I decided to login to the router's gateway to see what sort of control I could have over the flow of bandwidth. The only useful thing I could find find was QoS rules, which basically allow us to priortize traffic to our connected devices. While this would be useful in situations where bandwidth was more plentiful, I opted to try and find a way to set a hard cap on the bandwidth available to the AppleTV. 

Enter: FreshTomato

Fresh Tomato is a fork of the popular Tomato firmware. Fresh Tomato is a firmware that you can run on your compatible router, giving you more control and functionality than the stock router firmware.













Cheers!

-Nick

