---
author: Pranav Gajjewar
authorTwitter: ''
cover: ''
date: '2026-01-12T17:54:23+09:00'
description: I finally decided to jump into the rabbit hole
draft: true
hideComments: true
keywords:
- raspberry-pi
- personal
readingTime: false
showFullContent: false
tags:
- blog
- raspberry-pi
title: Buying a Raspberry Pi 5
---

I finally decided to jump into this rabbit hole.

The one that has been tempting me, beckoning me for such a long time.

There is nothing I enjoy more than ~wasting~ spending time setting up dozens of software services that I will (barely) use.

Like they say - the real productivity is the things we learn along the way.
But just learning is not enough unless I also share it.
Share it not to spread it - but share it to captue it more firmly into my own memory.

With that goal in mind - I will start off a series where I explain all the things that I will try on my Raspberry Pi 5.

Starting with the most basic hardware setup first.

# Spend some 💸

These are the three things I bought for the most basic setup.

  1. [Raspberry Pi 5 - 16GB variant](https://www.amazon.co.jp/-/en/dp/B0DTJRNBB6)
  2. [NVMe HAT + 128GB SSD](https://www.amazon.co.jp/dp/B0F389CXSX)
  3. [Active Cooler + Case](https://ja.aliexpress.com/item/1005007002853505.html)

## Explaining the choices

### Raspberry Pi

I decided to buy the 16GB version. Why? Given the current trend in RAM prices - I thought it might be safe to invest in purchasing the higher RAM version now.
What if the price increases later when I would actually need the high RAM?

This was one prominant fear that drove the decision. But on technical terms - where would the RAM actually shine?
Having higher RAM will allow for more memory - which directly translates to running more docker containers - being able to utilize more for cache when running a network filesystem
and various additional benefits that I have not yet been enlightened to.

In practical, 16GB RAM is overkill. Having using it for sometime, I can run a dozen containers within 2GB of RAM usage.
And I have yet to see any caching benefits for NAS usage (more on that later).
In hindsight - an 8GB version (heck, even 4GB) would have been sufficient for my purposes.

### NVMe HAT

If you are planning to run Raspberry Pi for any practical purpose like running docker, you cannot run it on the typical SD Card.
Read/Write performance for SD card is abysmal and everything would feel too slow because IO will become the bottleneck for running anything.

Especially for cases like running docker containers or any kind of database that involves and reading and writing small chunks at random locations over the memory blocks.
SD Cards are not built for this. But SSDs are the storage which shine under this kind of workload.

There are two options for running the Raspberry Pi over SSD.
  - Either do it via USB 3.0 SSD
  - Or use an NVMe interface to connect an NVMe SSD.

The IO performance for USB3.0 is a bottleneck because SSDs can support higher read/write speeds.
So to maximize the performance - use an SSD connected over NVMe.

Practically this translates into --
  - Fast boot times
  - Fast container up for docker services
  - Faster DB operations if you are running

For the purposes of performance-maxxing - you can use the NVMe. But even if you use USB 3.0, for most usecase there will not be much observable difference.
You can very well live with just connecting an SSD over USB3.0.

Here's my benchmark for all the 3 different setups that I outlined above.
  1. [SD Card](https://pibenchmarks.com/benchmark/130023/)
  2. [SSD over USB3.0](https://pibenchmarks.com/benchmark/130043/)
  3. [SSD over NVMe](https://pibenchmarks.com/benchmark/129694/)
