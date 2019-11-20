---
title: "Service Mesh"
date: 2018-06-17T11:57:10+10:00
draft: true
tags: ["study note", "service mesh", "istio", "consul"]
---

## What is service mesh?
A dedicated infrastructure layer to handle service to service communication by abstracting network away from developers. The main goal of service mesh is to provide reliable delivery of requests through the complex topology of services. The service mesh is a networking model that sits at a layer of abstraction above TCP/IP. [It assumes that the underlying L3/L4 network is present and capable of delivering bytes from point to point. (It also assumes that this network, as with every other aspect of the environment, is unreliable; the service mesh must therefore also be capable of handling network failures)][1].

## Example
 All network traffic (HTTP, REST, gRPC, Redis, etc.) from an individual service instance flows via its local sidecar proxy to the appropriate destination. Thus, the service instance is not aware of the network at large and only knows about its local proxy. In effect, the distributed system network has been abstracted away from the service programmer.

## Control plane

## Data plane


[1]: https://dzone.com/articles/whats-a-service-mesh-and-why-do-i-need-one
