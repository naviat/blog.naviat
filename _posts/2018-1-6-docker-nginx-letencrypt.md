---
layout: post
tags: Docker, crypto
title: Docker + Nginx + LetsEncrypt
---
### Docker + Nginx + LetsEncrypt

Today’s most web applications probably have these characteristics:

- they are complex
- as such, they depend on many moving pieces (e.g. reverse proxy, cache, db, etc)
- require secure connections via TLS

Here are quick tips I found useful to deploy such applications.

Technologies used:

1. Docker
1. Docker Compose
1. Docker Machine
1. Nginx
1. LetsEncrypt (with tool called certbot)

Docker Compose is extremely useful to help manage the complexity of the application’s moving pieces. In case you did not use it before, here is my 2 second pitch. Docker Compose allows to define all of the components in a single configuration therefore allowing for easier maintenance and deployment.
