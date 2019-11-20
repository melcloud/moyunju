---
title: "Consul"
date: 2018-06-09T08:23:50+10:00
draft: true
tags: ["study note", "hashicorp", "consul"]
---

## What is Consul?
An application provides:

- **Service discovery**. Clients of Consul can provide a service, and other clients can use Consul to discover providers of a given service. Service discovery can be done through DNS or HTTP
- **Health Checking**. Either associated with a given service (check status code), or with the local node (check host health).
    - Routing traffic base on health status
    - General monitor
- **Key/Value Store**. Hierarchical key/value store.
    - Feature flagging
    - Dynamic configuration
    - Coordination

Consul is also multi datacentre ready.

## Components
The agent can run either in server or client mode. Each datacenter must have at least one server, though a cluster of 3 or 5 servers is recommended. A single server deployment is highly discouraged as data loss is inevitable in a failure scenario.

- **Client**. Responsible for health checking. Automatically forward other requests to *server*.
- **Server**. Where data is stored and replicated. Consul servers elect a leader, and for a set of size n, quorum requires at least (n/2)+1 members. E.g. for a 5 server nodes, at least 3 nodes are required to form quorum. If a quorum of nodes is unavailable, the cluster becomes unavailable and no new log can be committed.

Components that need to discover other services or nodes can query any of the Consul servers or any of the Consul agents. When a cross-datacenter service discovery or configuration request is made, the **local Consul servers** forward the request to the remote datacenter and return the result.

## Development mode
To start a single node Consul environment, use `consul agent -dev`

> Note for OS X Users: Consul uses your hostname as the default node name. If your hostname contains periods, DNS queries to that node will not work with Consul. To avoid this, explicitly set the name of your node with the `-node` flag.

## Consul members
Consul nodes can be listed with `consul members`. The `consul members` command is eventually consistent based on gossip protocol. For a strongly consistent view, call HTTP API endpoint at `localhost:8500/v1/catalog/nodes`. DNS interface can also be used to query nodes. DNS lookups need to point at the Consul agent's DNS server which runs on port 8600 by default.

```bash
dig @127.0.0.1 -p 8600 <address>
```

## Graceful shutdown vs forcibly shutdown
By gracefully leaving, Consul notifies other cluster members that the node *left*. If you had forcibly killed the agent process, other members of the cluster would have detected that the node *failed*.

| Shutdown method | Node status | Implications                                                                                   |
|-----------------|-------------|------------------------------------------------------------------------------------------------|
| Graceful        | left        | services and checks are removed from the catalog. No longer contacted by Consul                |
| Forcibly        | failed      | Marked as critical, not removed from the catalog. Consul will try to reconnect to failed nodes |

Additionally, if an agent is operating as a server, a graceful leave is important to avoid causing a potential availability outage affecting the consensus protocol.
