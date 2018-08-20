---
layout: post
tags: Jenkins
title: Use Jenkins To Restart Elasticsearch Instances!
---
Ever need to restart ES instances for a critical ES cluster?

To play safe, you need to run a long procedure. And Stay Alarmed! The whole procedure would take hours.

How about we do it from Jenkins? Kind of VisualOps.

![exampe01](/images/2018-08-20_11-42-47.png)

#### Why Need To Restart?

Instead of how, we definitely need to ask why first.

Some ES instances may run into Full GC. Either because of low free RAM, or huge traffic.

What’s worse. When it’s the master node, the whole cluster will freeze.

If it keeps being this, we have to restart instances. Then ES cluster will recover.

Command to check full gc count for all ES nodes:
```
curl "http://$es_ip:$es_port/_nodes/stats" | \
  jq '[ .nodes | to_entries | sort_by(.value.jvm.gc.collectors.old.collection_count) | .[] | { node: .value.name, full_gc_count: .value.jvm.gc.collectors.old.collection_count } ]'
```

Python way to check full gc count: **[GitHub](https://gist.github.com/naviat/877238284c79b6512e3bbd91d2c54439)**

### What Are The Painpoints?

1. **Time-consuming**. It’s not a simple command. See official ES restart procedure: [here](https://www.elastic.co/guide/en/elasticsearch/reference/2.3/restart-upgrade.html)

```
First you need to run a flushed sync. 
Thus we reduce version conflict, after restart.

Temporarily disable shard allocations.
This helps avoid unnecessary shards rebalancing.

Rebalancing huge data will take hours. 
e.g, restarting node with 1TB data might take more than 4 hours.
```

* **Error-prone**. The official procedure looks quite straightforward. But…

```
You try to run flushed sync before start. 
But the request has just hang for more than 5 minutes. What will you do?

You want to change the shards allocation before or after restart.
But the API just runs into 5XX server errors. Then what?

After the instance restart, ES cluster is supposed to initialize shards.
But it just hang there. What's the next?

Still feel comfortable with the mission ahead of you?
```

* **Dangerous**. ES cluster will turn yellow for a while, even after we have finished the instance restart.

```
ES cluster would be loading the shards of that instance.

During this period of yellow, system is weak.

e.g. You may experience some other surprise(s) during this period.
```

* **Human Intervene**. It’s safer if we can do it manually, apparently this doesn’t scale.

```
As a DevOps professional, you know the pain with this approach. Right?
Try to restart it at midnight. And 3-5 times every week.
```

### How Jenkins Can Help?

If we can have a mature Jenkins job, it would be much better.

Not only you(DevOps) can issue the restart, but also the Developers!

The running history and error messages can be easily tracked

No need to check for each step and wait for ES turn green. Jenkins can send us notifications.

This definitely looks better. Right?

### Key Considerations Of Jenkins Approach

There are no silver bullets here.

Actually Jenkins approach only solve point #4 in the previous section. For points of #1(Time-consuming), #2(Erro-prone), #3(Dangerous), they’re still there.

Here are improvements I have made. I admit, they don’t root-fix the problems.

(Please leave me comments, if you have better suggestions).

* **Safe-protection**: Before Jenkins make any change, check ES status

```
If ES is not green, refuse to run. And send out alerts
Restarting instances with ES is yellow or red, it's super dangerous.

If ES is too slow to respond, refuse to run.
Restarting would bring even more load to the current cluster.
So it's better we ask human intervene.
```

* **Add 2 retries**: this would be helpful when we try to change shard allocations.

* **Ignore errors of synced flushed**. This HTTP request will always run into ```HTTP/1.1 409 Conflict.```

**Here comes the solution in [GitHub!](https://gist.github.com/naviat/f9767d78cf3d84613b5f758ff7be6a0f)** 




