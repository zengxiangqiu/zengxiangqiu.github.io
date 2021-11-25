---
title:  "Deployment Strategies"
date:   2021-11-25 11:12:27 +0800
categories: [DevOps]
tags: [Deployment]
---

## Rolling Deployment

A rolling deployment is a software release strategy that staggers deployment across multiple phases, which usually include one or more servers performing one or more function within a server cluster. Rather than updating all servers or tiers simultaneously, the organization installs the updated software package on one server or subset of servers at a time. A rolling deployment is used to reduce application downtime and unforeseen consequences or errors in software updates.

![rolling-deployment](https://cdn.ttgtmedia.com/rms/onlineImages/rolling_deployment_desktop.jpg)

refer to [Rolling Deployment](https://searchitoperations.techtarget.com/definition/rolling-deployment)

## Blue/Green Deployment
A blue/green deployment is a change management strategy for releasing software code. Blue/green deployments, which may also be referred to as A/B deployments require two identical hardware environments that are configured exactly the same way. While one environment is active and serving end users, the other environment remains idle.

Blue/green deployments need two identical sets of hardware, and that hardware carries added costs and overhead without actually adding capacity or improving utilization. Organizations that cannot afford to duplicate hardware configurations may use other strategies such as canary testing or rolling deployments. A canary test deploys new code to a small group of users, while a rolling deployment staggers the rollout of new code across servers.

![blue-green-deployment](https://cdn.ttgtmedia.com/rms/onlineImages/bluegreen_deployment_mobile.jpg)

refer to [Blue/Green Deployment](https://searchitoperations.techtarget.com/definition/blue-green-deployment)

## Canary Deployment

In software engineering, canary deployment is the practice of making staged releases. We roll out a software update to a small part of the users first, so they may test it and provide feedback. Once the change is accepted, the update is rolled out to the rest of the users.

Canary deployments show us how users interact with application changes in the real world. As in blue-green deployments, the canary strategy offers no-downtime upgrades and easy rollbacks. Unlike blue-green, canary deployments are smoother, and failures have limited impact.

![canary-deployment](https://wpblog.semaphoreci.com/wp-content/uploads/2020/08/user2.png)

refer to [Canary Deployment](https://semaphoreci.com/blog/what-is-canary-deployment)

