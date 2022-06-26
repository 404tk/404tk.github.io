---
title: etcd接管K8s api-server
author: 404tk
date: 2022-06-22
category: Kubernetes
layout: post
---

Exploit: etcd-get-k8s-token
-------------
List key and value pairs under the /registry/secrets/kube-system/ in etcd service, regular extract plaintext service-account-token, requests the default port 6443 'K8s API-server' service to verify the validity of the token and take over the cluster.  

遍历`etcd`中`/registry/secrets/kube-system/`前缀下的key、value对，正则提取明文service-account-token，对默认6443端口`K8s api-server`服务进行请求，验证token有效性，可进一步接管集群。  

Usage
-------------
```shell
./cdk run etcd-get-k8s-token (anonymous|default) <endpoint> <cert> <cert_key> <ca>
```  

Example
-------------
```shell
./cdk run etcd-get-k8s-token anonymous http://172.16.61.10:2379
```  
![](/assets/images/etcd_unauth.png)