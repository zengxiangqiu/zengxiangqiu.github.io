---
title: "Learn Test From k6"
date:  2021-11-30 14:35:16 +0800
categories: [DevOps]
tags: [测试]
---


![many types of tests](https://k6.io/docs/static/e45e3f092ab0445aa3da987a69ddad85/47a22/test-types.webp)

## Smoke Test

Smoke Test's role is to verify that your system can handle minimal load, without any problems.

You want to run a smoke test to:

* Verify that your test script doesn't have errors.
* Verify that your system doesn't throw any errors when under minimal load.

## Load Test

Load Test is primarily concerned with assessing the performance of your system in terms of concurrent users or requests per second.

You should run a Load Test to:

* Assess the current performance of your system under typical and peak load.
* Make sure you continue to meet the performance standards as you make changes to your system (code and infrastructure).

## Stress Test

Stress Test and Spike testing are concerned with assessing the limits of your system and stability under extreme conditions.

You typically want to stress test an API or website to determine:

* How your system will behave under extreme conditions.
* What the maximum capacity of your system is in terms of users or throughput.
* The breaking point of your system and its failure mode.
* If your system will recover without manual intervention after the stress test is over.

### Spike testing

You want to execute a spike test to determine:

* How your system will perform under a sudden surge of traffic.
* If your system will recover once the traffic has subsided.

## Soak Test

Soak Test tells you about reliability and performance of your system over an extended period of time.

You typically run this test to:

* Verify that your system doesn't suffer from bugs or memory leaks, which result in a crash or restart after several hours of operation.
* Verify that expected application restarts don't lose requests.
* Find bugs related to race-conditions that appear sporadically.
* Make sure your database doesn't exhaust the allotted storage space and stops.
* Make sure your logs don't exhaust the allotted disk storage.
* Make sure the external services you depend on don't stop working after a certain amount of requests are executed.
