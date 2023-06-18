---
title: "Using Azure Durable Functions to Subscribe to My Favorite Manga "
date: 2023-06-04T21:35:48+08:00
draft: true
tags: ["Azure", "Azure Functions", "Python", "Web Scraping"]
---

# The Problem
Reading manga is one of my favorite pastimes. For those of you who are unfamiliar, manga is the name for Japanese comic, they are known for their distinc artstyle and story. Most manga nowadays are published online, with new episodes coming out every other week or so. I usually have a go-to website for reading the newest releases, but since I am following so many manga at once, it becomes cumbersome to have to manually check the mainpage for each manga I am following...

# The First Attempt
Since the website doesn't have a builtin mechanism to notify me of new releases, and I don't want to register another online account just to receive manga updates, I started looking into an automated solution. The first thought that came to mind is RSS, an increasingly rare piece of technology that was once the golden standard when it came to subscriping to things. Surprisingly, I quickly found what appeared to be the perfect solution to my problem - [RSSHub](https://docs.rsshub.app/en/). It is essentially a hub for many developers to write and upload their custom rss feed for websites that did not have rss baked in. My manga website turned out to be one of them, therefore I can already access an rss feed for my manga. I quickly put together a Logic Apps to consume the feed. However, there were several problem with this approach. Without going into too much details, here are the problems I encountered:
1. **Performance** - sometimes requests sent to RSSHub from Logic Apps take up to half a minute to respond, I have no idea why but the latency is not healthy for Logic Apps.
2. **Format** - the JSON result returned from requests contained a "\<pubDate\>" field for every episode, which I assumed was when the episode was uploaded. However, it turns out whenever a new episode is uploaded, ***EVERY SINGLE EPISODE*** changes its "\<pubDate\>" to the newest time, therefore I can't use this field to select the episodes that were new. (This is the most stupid thing I have ever seen)
3. **Hosting** - ideally, RSSHub should be self-hosted for performance and security reasons. Even though the author was kind enough to let the public use the hosted demo for free, I wasn't satisfied with its performance as stated in 1, but I also didn't want to host it myself and add another docker compose workflow on my vps.

#




