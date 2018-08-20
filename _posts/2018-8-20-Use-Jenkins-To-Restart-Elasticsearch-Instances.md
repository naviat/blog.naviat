---
layout: post
tags: Jenkins
title: Use Jenkins To Restart Elasticsearch Instances!
---
Ever need to restart ES instances for a critical ES cluster?

To play safe, you need to run a long procedure. And Stay Alarmed! The whole procedure would take hours.

How about we do it from Jenkins? Kind of VisualOps.

![First Post](/images/2018-08-20_11-42-47.png)

#### 1.Why Need To Restart?

Instead of how, we definitely need to ask why first.

Some ES instances may run into Full GC. Either because of low free RAM, or huge traffic.

What’s worse. When it’s the master node, the whole cluster will freeze.

If it keeps being this, we have to restart instances. Then ES cluster will recover.
[updating]
