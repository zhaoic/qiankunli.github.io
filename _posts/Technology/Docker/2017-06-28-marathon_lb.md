---

layout: post
title: docker环境下的服务发现
category: 技术
tags: Docker
keywords: Docker,macvlan

---


## 简介

容器的漂移伴随着ip的分配和释放，这对许多依赖ip的服务的运行产生了负面影响，比如nginx等，解决该问题，有以下办法

1. 为服务分配的ip永远不变
2. 第三方工具屏蔽ip变化

	* mesos-dns
   * marathon-lb


## marathon-lb

marathon-lb 可以部署在边际节点，为入口流量做路由负载。也可以部署在private节点，做内部的（marathon app special）负载均衡与服务发现。

下面谈下marathon-lb 部署在边际节点时的方式及问题：

1. 选择一个node，运行mathon-lb.py（容器方式或直接运行脚本）
2. lb监听marathon 事件，获取所有marathon app数据，并根据配置数据（configure + label）获得node port 与 marathon app port的映射关系

对于一个marathon lb配置，访问路线如下：

internet ==> marathon lb node ip : service port ==> container host : host port ==> container port.

问题

1. 默认支持的是docker bridge 模式，对于ip-pert-task类型有一点问题。比如marathon-lb issue[IP-Per_Task + ServicePort Issue](https://github.com/mesosphere/marathon-lb/issues/365)
2. marathon app的marathon.json 需配置 servicePort,每个服务不同。

	* 对于每个应用，marathon.json 需额外配置 service port
	* marathon-lb node 要预留一个port范围给service port


## 引用





	
	