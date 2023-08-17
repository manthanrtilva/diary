---
layout: post
title:  "Sniff SSl/TLS traffic"
date:   2023-06-22 03:45:33 +0000
categories: ssl,tls,wirestark,tcpdump,network
---
Now a day all network trafic runs on SSL/TLS. So it's not possible to what's transmitted on network by the application. And it's very much required while writing new protocol or troubleshooting exsting application library to figure out whether payload transmitted on network are as per defined protocol or recieved request/response from server/client are per defined protocol and not malformed.

There are many solutions to do this. But the one one which I mostly use is openssl-keylog. 
