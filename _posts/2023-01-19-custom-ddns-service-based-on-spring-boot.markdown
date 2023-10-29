---
layout: post
title:  "Custom DDNS service based on Spring Boot"
date:   2023-01-19 12:05:48 +0800
#categories: jekyll update
---

# Custom DDNS service based on Spring Boot

Recently, I found that the DDNS service provided by Asus is not working properly.
When My public IP has changed, the DNS is not updated in time.
It caused my VPN clients to disconnect from the server.
Therefore, I wrote a DDNS service based on Spring Boot for my personal use.
It works fine when run by jar file or docker.
In my scenario, there is a Truenas Scale server locally.
So, I just put a docker on it.
It has been working fine for about several weeks.

For the users who may encounter the same issue with the built-in DDNS server, you can just try my open-source DDNS service.
There is the GitHub link https://github.com/caltong/ddns.
Please refer to the readme file.
You are welcome to report issues or bugs.
BTW, for now, it only supports Cloudflare DNS, because I'm using Cloudflare only right now.
Please let me know if you want to use another DNS provider.
It will be better if you provide me test account for my development.