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

* Time-consuming. It’s not a simple command. See official ES restart procedure: [here](https://www.elastic.co/guide/en/elasticsearch/reference/2.3/restart-upgrade.html)

```
First you need to run a flushed sync. 
Thus we reduce version conflict, after restart.

Temporarily disable shard allocations.
This helps avoid unnecessary shards rebalancing.

Rebalancing huge data will take hours. 
e.g, restarting node with 1TB data might take more than 4 hours.
```
