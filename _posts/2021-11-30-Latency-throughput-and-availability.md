---
title: "Latency throughput and availability"
date:  2021-11-30 16:24:13 +0800
categories: [DevOps]
tags: [Latency,throughput,availability]
---

## Latency

Latency is the amount of time in milliseconds (ms) it takes a single message to be delivered.

### What causes latency

* Physical distance
* Complex computation
* Congestion
* Too many nodes

### How to improve latency

* Better paths
* Caching
* Protocol choice: certain protocols, like HTTP/2, intentionally reduce the amount of protocol overhead associated with a request, and can keep latency lower.

## Throughput

Throughput is the amount of data that is successfully transmitted through a system in a certain amount of time, measured in bits per second (bps).

### What causes low throughput

* Congestion
* Protocol overhead
* Latency

### How to improve throughput

* Increasing bandwidth
* Improving latency
* Protocol choice

## Availability

Availability is the amount of time that a system is able to respond, that is the ratio of Uptime / (Uptime + Downtime).

### What causes low availability

* Hardware failure
* Software bugs
* Complex architectures
* Dependent service outages
* Request overload
* Deployment issues

### How to improve availability

* Failover systems
* Clustering
* Backups
* Geographic redundancy
* Automatic testing, deployment, and rollbacks


[Latency, throughput, and availability: system design interview concepts (3 of 9)](https://igotanoffer.com/blogs/tech/latency-throughput-availability-system-design-interview#latency)
