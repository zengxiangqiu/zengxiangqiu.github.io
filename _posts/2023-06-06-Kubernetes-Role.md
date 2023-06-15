---
title: "Kubernetes Role"
date:  2023-06-06 09:55:36 +0800
categories: [kubernetes]
tags: [role,rolebinding,serviceaccount]
---

role对**单一**namespace资源授权

ClusterRole 除Role之外扩展node，endpoint,其他namespace


rolebinding

1. subjects(user, serviceaccount, group)
2. roleRef(role,ClusterRole)



